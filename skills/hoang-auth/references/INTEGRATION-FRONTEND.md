# Frontend Integration Manual

Hướng dẫn chi tiết tích hợp Keycloak auth vào Frontend SPA và Mobile App.
Document này đủ chi tiết để AI hoặc developer implement từ đầu đến cuối.

> **Prerequisite:** Đọc [INTEGRATION.md](./INTEGRATION.md) để hiểu auth flow tổng quan.

---

## Mục lục

1. [Nguyên tắc chung](#1-nguyên-tắc-chung)
2. [React (keycloak-js)](#2-react-keycloak-js)
3. [React (oidc-client-ts) — Recommended](#3-react-oidc-client-ts--recommended)
4. [Vue 3](#4-vue-3)
5. [Mobile (React Native)](#5-mobile-react-native)
6. [Các pattern nâng cao](#6-các-pattern-nâng-cao)
7. [Testing & Debug](#7-testing--debug)

---

## 1. Nguyên tắc chung

### Frontend chịu trách nhiệm

1. **Redirect user đến Keycloak login page** (Authorization Code + PKCE)
2. **Nhận authorization code** sau khi user login thành công
3. **Exchange code → access_token + refresh_token**
4. **Gửi access_token** kèm mọi API call (`Authorization: Bearer <token>`)
5. **Auto-refresh token** trước khi hết hạn
6. **Xử lý logout** (clear local state + redirect Keycloak logout)

### KHÔNG BAO GIỜ

- Lưu client secret ở frontend (public client không có secret)
- Tự validate JWT signature ở frontend (backend làm việc này)
- Gửi username/password trực tiếp qua frontend (luôn redirect qua Keycloak)

### Keycloak config cho Frontend

```
Keycloak URL:    https://auth.hoang.jp
Realm:           hoang
Client ID:       frontend-spa
Auth Flow:       Authorization Code + PKCE (S256)
```

### Token lifecycle

```
┌─ Login ──────────────────────────────────────────────────────────────┐
│                                                                       │
│  User clicks login → redirect to Keycloak → user enters credentials  │
│  → Keycloak redirects back with ?code=xxx                            │
│  → Frontend exchanges code for tokens                                 │
│                                                                       │
│  access_token:  5 phút  (dùng cho API calls)                        │
│  refresh_token: 30 phút (dùng để lấy access_token mới)              │
│  SSO session:   10 giờ  (silent check-sso hoạt động trong khoảng này)│
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

---

## 2. React (keycloak-js)

Official Keycloak JS adapter. Simple, ít code, nhưng ít flexible hơn oidc-client-ts.

### 2.1 Install

```bash
npm install keycloak-js
```

### 2.2 Keycloak instance

```typescript
// src/lib/keycloak.ts
import Keycloak from 'keycloak-js';

const keycloak = new Keycloak({
  url: 'https://auth.hoang.jp',
  realm: 'hoang',
  clientId: 'frontend-spa',
});

export default keycloak;
```

### 2.3 Auth Provider

```tsx
// src/providers/AuthProvider.tsx
import { createContext, useContext, useEffect, useState, ReactNode } from 'react';
import keycloak from '../lib/keycloak';

interface AuthContextType {
  isAuthenticated: boolean;
  isLoading: boolean;
  token: string | undefined;
  user: {
    id: string;
    username: string;
    email: string;
    firstName: string;
    lastName: string;
    roles: string[];
  } | null;
  login: () => void;
  logout: () => void;
  hasRole: (role: string) => boolean;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [isLoading, setIsLoading] = useState(true);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  useEffect(() => {
    keycloak
      .init({
        onLoad: 'check-sso',           // Không force login, check SSO session
        pkceMethod: 'S256',
        checkLoginIframe: false,        // Tắt để tránh issue với Cloudflare
        silentCheckSsoRedirectUri:
          window.location.origin + '/silent-check-sso.html',
      })
      .then((authenticated) => {
        setIsAuthenticated(authenticated);
        setIsLoading(false);
      })
      .catch((err) => {
        console.error('Keycloak init failed:', err);
        setIsLoading(false);
      });

    // Auto-refresh token trước khi hết hạn
    keycloak.onTokenExpired = () => {
      keycloak.updateToken(30).catch(() => {
        console.warn('Token refresh failed, logging out');
        keycloak.logout();
      });
    };
  }, []);

  const parsedToken = keycloak.tokenParsed;

  const user = isAuthenticated && parsedToken
    ? {
        id: parsedToken.sub!,
        username: parsedToken.preferred_username || '',
        email: parsedToken.email || '',
        firstName: parsedToken.given_name || '',
        lastName: parsedToken.family_name || '',
        roles: parsedToken.realm_access?.roles || [],
      }
    : null;

  const hasRole = (role: string) => user?.roles.includes(role) ?? false;

  const value: AuthContextType = {
    isAuthenticated,
    isLoading,
    token: keycloak.token,
    user,
    login: () => keycloak.login(),
    logout: () => keycloak.logout({ redirectUri: window.location.origin }),
    hasRole,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
}
```

### 2.4 Silent SSO check page

Tạo file `public/silent-check-sso.html`:

```html
<!DOCTYPE html>
<html>
<body>
  <script>
    parent.postMessage(location.href, location.origin);
  </script>
</body>
</html>
```

### 2.5 API client với auto-attach token

```typescript
// src/lib/api.ts
import keycloak from './keycloak';

const API_BASE = import.meta.env.VITE_API_URL || 'http://localhost:8080';

/**
 * Fetch wrapper tự đính kèm Bearer token.
 * Tự refresh token nếu sắp hết hạn trước khi gọi API.
 */
export async function apiFetch(path: string, options: RequestInit = {}): Promise<Response> {
  // Ensure token is fresh (refresh if expires within 10 seconds)
  try {
    await keycloak.updateToken(10);
  } catch {
    keycloak.login();
    throw new Error('Session expired');
  }

  const headers = new Headers(options.headers);
  headers.set('Authorization', `Bearer ${keycloak.token}`);
  headers.set('Content-Type', 'application/json');

  const response = await fetch(`${API_BASE}${path}`, {
    ...options,
    headers,
  });

  if (response.status === 401) {
    // Token invalid on server side, force re-login
    keycloak.login();
    throw new Error('Unauthorized');
  }

  return response;
}

// Helper methods
export const api = {
  get: (path: string) => apiFetch(path).then(r => r.json()),

  post: (path: string, body: unknown) =>
    apiFetch(path, { method: 'POST', body: JSON.stringify(body) }).then(r => r.json()),

  put: (path: string, body: unknown) =>
    apiFetch(path, { method: 'PUT', body: JSON.stringify(body) }).then(r => r.json()),

  delete: (path: string) =>
    apiFetch(path, { method: 'DELETE' }).then(r => r.json()),
};
```

### 2.6 Sử dụng trong components

```tsx
// src/App.tsx
import { AuthProvider } from './providers/AuthProvider';
import { Dashboard } from './pages/Dashboard';

export default function App() {
  return (
    <AuthProvider>
      <Dashboard />
    </AuthProvider>
  );
}
```

```tsx
// src/pages/Dashboard.tsx
import { useAuth } from '../providers/AuthProvider';
import { api } from '../lib/api';

export function Dashboard() {
  const { isLoading, isAuthenticated, user, login, logout, hasRole } = useAuth();

  if (isLoading) return <div>Loading...</div>;
  if (!isAuthenticated) return <button onClick={login}>Login</button>;

  return (
    <div>
      <p>Welcome, {user!.username} ({user!.email})</p>
      <p>Roles: {user!.roles.join(', ')}</p>

      <button onClick={() => api.get('/api/profile').then(console.log)}>
        Get Profile
      </button>

      {hasRole('admin') && (
        <button onClick={() => api.get('/api/admin/users').then(console.log)}>
          Admin Panel
        </button>
      )}

      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

### 2.7 Protected Route component

```tsx
// src/components/ProtectedRoute.tsx
import { ReactNode } from 'react';
import { useAuth } from '../providers/AuthProvider';

interface Props {
  children: ReactNode;
  roles?: string[];         // Required roles (any of)
  fallback?: ReactNode;     // Show when unauthorized
}

export function ProtectedRoute({ children, roles, fallback }: Props) {
  const { isLoading, isAuthenticated, login, hasRole } = useAuth();

  if (isLoading) return <div>Loading...</div>;

  if (!isAuthenticated) {
    login();
    return <div>Redirecting to login...</div>;
  }

  if (roles && !roles.some(hasRole)) {
    return fallback ? <>{fallback}</> : <div>Access denied</div>;
  }

  return <>{children}</>;
}

// Usage:
// <ProtectedRoute roles={['admin']}>
//   <AdminPage />
// </ProtectedRoute>
```

---

## 3. React (oidc-client-ts) — Recommended

Lightweight, framework-agnostic OIDC library. Recommended cho projects mới.

### 3.1 Install

```bash
npm install oidc-client-ts react-oidc-context
```

### 3.2 OIDC Configuration

```typescript
// src/lib/oidc-config.ts
import { WebStorageStateStore } from 'oidc-client-ts';

export const oidcConfig = {
  authority: 'https://auth.hoang.jp/realms/hoang',
  client_id: 'frontend-spa',
  redirect_uri: window.location.origin + '/callback',
  post_logout_redirect_uri: window.location.origin,
  response_type: 'code',
  scope: 'openid profile email',
  automaticSilentRenew: true,     // Auto-refresh token
  userStore: new WebStorageStateStore({ store: window.sessionStorage }),
};
```

### 3.3 Setup Provider

```tsx
// src/main.tsx
import { AuthProvider } from 'react-oidc-context';
import { oidcConfig } from './lib/oidc-config';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <AuthProvider {...oidcConfig}>
    <App />
  </AuthProvider>
);
```

### 3.4 Callback page

```tsx
// src/pages/Callback.tsx (route: /callback)
import { useAuth } from 'react-oidc-context';
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

export function Callback() {
  const auth = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if (!auth.isLoading && auth.isAuthenticated) {
      // Redirect về trang trước đó hoặc home
      const returnTo = sessionStorage.getItem('returnTo') || '/';
      sessionStorage.removeItem('returnTo');
      navigate(returnTo, { replace: true });
    }
  }, [auth.isLoading, auth.isAuthenticated]);

  return <div>Processing login...</div>;
}
```

### 3.5 Sử dụng trong components

```tsx
// src/App.tsx
import { useAuth } from 'react-oidc-context';

function App() {
  const auth = useAuth();

  if (auth.isLoading) return <div>Loading...</div>;

  if (auth.error) return <div>Auth error: {auth.error.message}</div>;

  if (!auth.isAuthenticated) {
    return <button onClick={() => auth.signinRedirect()}>Login</button>;
  }

  // Extract Keycloak-specific claims
  const roles = (auth.user?.profile as any)?.realm_access?.roles || [];
  const hasRole = (role: string) => roles.includes(role);

  return (
    <div>
      <p>Welcome, {auth.user?.profile.preferred_username}</p>
      <p>Email: {auth.user?.profile.email}</p>

      {hasRole('admin') && <AdminPanel />}

      <button onClick={() => auth.removeUser().then(() => auth.signoutRedirect())}>
        Logout
      </button>
    </div>
  );
}
```

### 3.6 API client

```typescript
// src/lib/api.ts
import { User } from 'oidc-client-ts';

const API_BASE = import.meta.env.VITE_API_URL || 'http://localhost:8080';

/**
 * Tạo API client với token từ OIDC user.
 * Dùng: const api = createApiClient(auth.user);
 *       const data = await api.get('/api/profile');
 */
export function createApiClient(user: User | null | undefined) {
  async function apiFetch(path: string, options: RequestInit = {}) {
    if (!user?.access_token) {
      throw new Error('Not authenticated');
    }

    const headers = new Headers(options.headers);
    headers.set('Authorization', `Bearer ${user.access_token}`);
    headers.set('Content-Type', 'application/json');

    const response = await fetch(`${API_BASE}${path}`, { ...options, headers });

    if (!response.ok) {
      throw new Error(`API error: ${response.status} ${response.statusText}`);
    }
    return response.json();
  }

  return {
    get: (path: string) => apiFetch(path),
    post: (path: string, body: unknown) =>
      apiFetch(path, { method: 'POST', body: JSON.stringify(body) }),
    put: (path: string, body: unknown) =>
      apiFetch(path, { method: 'PUT', body: JSON.stringify(body) }),
    delete: (path: string) => apiFetch(path, { method: 'DELETE' }),
  };
}
```

---

## 4. Vue 3

### 4.1 Install

```bash
npm install oidc-client-ts
```

### 4.2 Auth composable

```typescript
// src/composables/useAuth.ts
import { ref, computed, readonly } from 'vue';
import { UserManager, WebStorageStateStore, User } from 'oidc-client-ts';

const userManager = new UserManager({
  authority: 'https://auth.hoang.jp/realms/hoang',
  client_id: 'frontend-spa',
  redirect_uri: window.location.origin + '/callback',
  post_logout_redirect_uri: window.location.origin,
  response_type: 'code',
  scope: 'openid profile email',
  automaticSilentRenew: true,
  userStore: new WebStorageStateStore({ store: window.sessionStorage }),
});

const user = ref<User | null>(null);
const isLoading = ref(true);

// Init: check if user already has session
userManager.getUser().then((u) => {
  user.value = u;
  isLoading.value = false;
});

// Listen for token events
userManager.events.addUserLoaded((u) => { user.value = u; });
userManager.events.addUserUnloaded(() => { user.value = null; });
userManager.events.addSilentRenewError(() => { login(); });

export function useAuth() {
  const isAuthenticated = computed(() => !!user.value && !user.value.expired);

  const profile = computed(() => {
    if (!user.value) return null;
    const p = user.value.profile as any;
    return {
      id: p.sub,
      username: p.preferred_username || '',
      email: p.email || '',
      firstName: p.given_name || '',
      lastName: p.family_name || '',
      roles: p.realm_access?.roles || [],
    };
  });

  const token = computed(() => user.value?.access_token);

  const hasRole = (role: string) => profile.value?.roles.includes(role) ?? false;

  function login() {
    sessionStorage.setItem('returnTo', window.location.pathname);
    userManager.signinRedirect();
  }

  function logout() {
    userManager.signoutRedirect();
  }

  // Call this in /callback route
  async function handleCallback() {
    const u = await userManager.signinRedirectCallback();
    user.value = u;
    return sessionStorage.getItem('returnTo') || '/';
  }

  return {
    isLoading: readonly(isLoading),
    isAuthenticated,
    user: profile,
    token,
    hasRole,
    login,
    logout,
    handleCallback,
  };
}
```

### 4.3 Sử dụng

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { useAuth } from './composables/useAuth';

const { isLoading, isAuthenticated, user, login, logout, hasRole } = useAuth();
</script>

<template>
  <div v-if="isLoading">Loading...</div>

  <div v-else-if="!isAuthenticated">
    <button @click="login">Login</button>
  </div>

  <div v-else>
    <p>Welcome, {{ user?.username }}</p>
    <p>Roles: {{ user?.roles.join(', ') }}</p>

    <div v-if="hasRole('admin')">
      <h2>Admin Panel</h2>
    </div>

    <button @click="logout">Logout</button>
  </div>
</template>
```

### 4.4 Vue Router guard

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router';
import { useAuth } from '../composables/useAuth';

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: () => import('../pages/Home.vue') },
    { path: '/callback', component: () => import('../pages/Callback.vue') },
    {
      path: '/dashboard',
      component: () => import('../pages/Dashboard.vue'),
      meta: { requiresAuth: true },
    },
    {
      path: '/admin',
      component: () => import('../pages/Admin.vue'),
      meta: { requiresAuth: true, roles: ['admin'] },
    },
  ],
});

router.beforeEach((to) => {
  const { isAuthenticated, hasRole, login } = useAuth();

  if (to.meta.requiresAuth && !isAuthenticated.value) {
    login();
    return false;
  }

  const requiredRoles = to.meta.roles as string[] | undefined;
  if (requiredRoles && !requiredRoles.some(hasRole)) {
    return '/';  // redirect home if missing role
  }
});

export default router;
```

---

## 5. Mobile (React Native)

### 5.1 Install

```bash
npm install react-native-app-auth
# iOS
cd ios && pod install
```

### 5.2 Config & Auth functions

```typescript
// src/auth/keycloak.ts
import { authorize, refresh, revoke, AuthConfiguration } from 'react-native-app-auth';
import AsyncStorage from '@react-native-async-storage/async-storage';

const config: AuthConfiguration = {
  issuer: 'https://auth.hoang.jp/realms/hoang',
  clientId: 'mobile-app',
  redirectUrl: 'hoang://callback',       // Custom URL scheme (match Keycloak config)
  scopes: ['openid', 'profile', 'email'],
  usePKCE: true,
};

const STORAGE_KEY = 'auth_tokens';

interface Tokens {
  accessToken: string;
  refreshToken: string;
  accessTokenExpirationDate: string;
}

export async function login(): Promise<Tokens> {
  const result = await authorize(config);
  const tokens: Tokens = {
    accessToken: result.accessToken,
    refreshToken: result.refreshToken,
    accessTokenExpirationDate: result.accessTokenExpirationDate,
  };
  await AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(tokens));
  return tokens;
}

export async function getValidToken(): Promise<string> {
  const stored = await AsyncStorage.getItem(STORAGE_KEY);
  if (!stored) throw new Error('Not authenticated');

  const tokens: Tokens = JSON.parse(stored);

  // Check if expired
  if (new Date(tokens.accessTokenExpirationDate) <= new Date()) {
    // Refresh
    const result = await refresh(config, { refreshToken: tokens.refreshToken });
    const newTokens: Tokens = {
      accessToken: result.accessToken,
      refreshToken: result.refreshToken || tokens.refreshToken,
      accessTokenExpirationDate: result.accessTokenExpirationDate,
    };
    await AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(newTokens));
    return newTokens.accessToken;
  }

  return tokens.accessToken;
}

export async function logout(): Promise<void> {
  await AsyncStorage.removeItem(STORAGE_KEY);
  // Optionally revoke token on Keycloak
}

export async function isAuthenticated(): Promise<boolean> {
  const stored = await AsyncStorage.getItem(STORAGE_KEY);
  return !!stored;
}
```

### 5.3 API calls

```typescript
// src/api/client.ts
import { getValidToken } from '../auth/keycloak';

const API_BASE = 'https://api.hoang.jp';  // Your backend URL

export async function apiFetch(path: string, options: RequestInit = {}) {
  const token = await getValidToken();

  const response = await fetch(`${API_BASE}${path}`, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
  });

  if (response.status === 401) {
    // Token invalid, force re-login
    throw new Error('Session expired');
  }

  return response.json();
}
```

### 5.4 iOS Info.plist (URL Scheme)

```xml
<!-- ios/YourApp/Info.plist -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>hoang</string>
    </array>
  </dict>
</array>
```

### 5.5 Android (URL Scheme)

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<activity android:name="net.openid.appauth.RedirectUriReceiverActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="hoang" android:host="callback" />
    </intent-filter>
</activity>
```

---

## 6. Các pattern nâng cao

### 6.1 Token Refresh Strategy

```
┌─ Access Token Lifecycle ────────────────────────────────┐
│                                                          │
│  t=0m    Token issued (valid 5 min)                     │
│  t=4m    Token về còn < 1 phút → trigger silent refresh │
│  t=5m    Token expired                                   │
│                                                          │
│  Refresh flow:                                           │
│  1. Library tự gọi token endpoint với refresh_token     │
│  2. Keycloak trả access_token mới                       │
│  3. Cập nhật token trong memory                         │
│  4. Nếu refresh_token cũng expired → redirect login     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Quan trọng:** Cả `keycloak-js` và `oidc-client-ts` đều hỗ trợ auto-refresh.
Bạn KHÔNG cần tự implement refresh logic.

### 6.2 Logout đúng cách

```typescript
// ĐÚNG: Logout cả local + Keycloak (clear SSO session)
function logout() {
  // 1. Clear local state
  // 2. Redirect to Keycloak logout endpoint
  window.location.href = 'https://auth.hoang.jp/realms/hoang/protocol/openid-connect/logout'
    + '?post_logout_redirect_uri=' + encodeURIComponent(window.location.origin)
    + '&client_id=frontend-spa';
}

// SAI: Chỉ clear local state → user vẫn có SSO session trên Keycloak
// → Lần sau mở app sẽ tự login lại mà không cần nhập password
```

### 6.3 Handling Multiple Tabs

```typescript
// Dùng BroadcastChannel để sync auth state across tabs
const authChannel = new BroadcastChannel('auth');

authChannel.onmessage = (event) => {
  if (event.data === 'logout') {
    // Clear local auth state and redirect
    window.location.href = '/';
  }
};

// Khi logout, broadcast to other tabs
function logout() {
  authChannel.postMessage('logout');
  // ... actual logout logic
}
```

### 6.4 Role-based UI Rendering

```tsx
// Pattern: Component chỉ hiện khi user có role
function RoleGuard({ roles, children }: { roles: string[]; children: ReactNode }) {
  const { hasRole } = useAuth();
  if (!roles.some(hasRole)) return null;
  return <>{children}</>;
}

// Usage
<RoleGuard roles={['admin']}>
  <DeleteButton />
</RoleGuard>

<RoleGuard roles={['admin', 'moderator']}>
  <ModerateButton />
</RoleGuard>
```

### 6.5 Social Login trên Frontend

Social login (Google, GitHub) hoạt động tự động — user sẽ thấy nút trên Keycloak login page.
Frontend KHÔNG cần code đặc biệt. Flow:

```
Frontend → redirect to Keycloak login page
  → User clicks "Google" / "GitHub"
  → Keycloak redirects to Google/GitHub
  → User authorizes
  → Google/GitHub redirects back to Keycloak
  → Keycloak creates/links user account
  → Keycloak redirects back to Frontend with auth code
  → Frontend exchanges code for tokens (same as normal flow)
```

---

## 7. Testing & Debug

### 7.1 Test login flow thủ công

Mở browser, vào URL này để trigger login:
```
https://auth.hoang.jp/realms/hoang/protocol/openid-connect/auth
  ?client_id=frontend-spa
  &response_type=code
  &scope=openid%20profile%20email
  &redirect_uri=http://localhost:3000/callback
  &code_challenge_method=S256
  &code_challenge=<generated>
```

Hoặc đơn giản hơn, vào Keycloak account page:
```
https://auth.hoang.jp/realms/hoang/account/
```

### 7.2 Check token trong browser DevTools

```javascript
// Console: xem token đang dùng
// Nếu dùng keycloak-js:
console.log(keycloak.tokenParsed);
console.log(keycloak.token);

// Nếu dùng oidc-client-ts:
// Token trong auth.user.access_token
// Decoded: auth.user.profile
```

### 7.3 Common Frontend Errors

| Error | Nguyên nhân | Fix |
|-------|-------------|-----|
| `Invalid redirect_uri` | redirect_uri không match Keycloak client config | Thêm URL vào Keycloak Admin → Client → Valid Redirect URIs |
| `CORS error` | Backend chưa config CORS cho frontend origin | Config CORS trên backend (xem INTEGRATION-BACKEND.md) |
| `Token expired` liên tục | Auto-refresh không hoạt động | Check `automaticSilentRenew: true` hoặc `keycloak.onTokenExpired` |
| `Login loop` | Redirect vòng lặp giữa app và Keycloak | Check redirect_uri, check `onLoad` setting |
| Blank page sau login | Callback route không handle đúng | Đảm bảo `/callback` route gọi `signinRedirectCallback()` |
| `PKCE error` | Missing code_challenge | Đảm bảo `pkceMethod: 'S256'` trong config |

### 7.4 Debug Checklist

```
[ ] Keycloak đang chạy? → curl https://auth.hoang.jp/realms/hoang
[ ] Client ID đúng? → frontend-spa
[ ] Redirect URI đã thêm trong Keycloak? → http://localhost:3000/*
[ ] PKCE enabled? → pkceMethod: 'S256'
[ ] Web Origins đã config? → http://localhost:3000, https://*.hoang.jp
[ ] Backend CORS cho phép frontend origin?
[ ] Token có roles? → Check tokenParsed.realm_access.roles
```
