# Tích hợp SSO cho app mới

Hướng dẫn step-by-step thêm Keycloak SSO vào bất kỳ app nào trong hệ sinh thái hoang.jp.
Tất cả app dùng chung realm `hoang` — user login 1 lần, tự động xác thực ở tất cả app khác.

---

## Keycloak Info

```
Keycloak URL:      https://auth.hoang.jp
Realm:             hoang
OIDC Discovery:    https://auth.hoang.jp/realms/hoang/.well-known/openid-configuration
JWKS:              https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs
Issuer:            https://auth.hoang.jp/realms/hoang
Token Endpoint:    https://auth.hoang.jp/realms/hoang/protocol/openid-connect/token
Logout Endpoint:   https://auth.hoang.jp/realms/hoang/protocol/openid-connect/logout
Admin Console:     chỉ dùng trong LAN (Board quyết 02/10/2026, HOA-316)
Admin .env:        ~/Workspace/OSS/keycloak/.env
```

---

## Bước 1: Tạo Keycloak Client cho app

Mỗi app cần 1 client riêng. Có 2 cách:

### Cách A: Qua Admin REST API (recommended cho AI)

```bash
# 1. Lấy admin token (chạy trên MacbookServer). Mật khẩu đi qua stdin, không nằm trong argv (D-0015).
cd ~/Workspace/OSS/keycloak
kcenv() { grep "^$1=" .env | cut -d= -f2-; }   # đọc 1 biến trong .env, không in ra màn hình
KC_TOKEN=$(kcenv KC_ADMIN_PASSWORD | tr -d '\n' \
  | curl -s -X POST "http://localhost:26201/realms/master/protocol/openid-connect/token" \
      -d "grant_type=password" -d "client_id=admin-cli" \
      --data-urlencode "username=$(kcenv KC_ADMIN_USER)" \
      --data-urlencode "password@-" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

# 2a. Tạo PUBLIC client (cho SPA frontend)
curl -s -X POST "http://localhost:26201/admin/realms/hoang/clients" \
  -H "Authorization: Bearer $KC_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": "<APP_CLIENT_ID>",
    "name": "<App Display Name>",
    "enabled": true,
    "publicClient": true,
    "standardFlowEnabled": true,
    "directAccessGrantsEnabled": false,
    "redirectUris": ["https://<app-domain>/*", "http://localhost:<dev-port>/*"],
    "webOrigins": ["https://<app-domain>", "http://localhost:<dev-port>"],
    "protocol": "openid-connect",
    "attributes": {
      "pkce.code.challenge.method": "S256",
      "post.logout.redirect.uris": "https://<app-domain>/*##http://localhost:<dev-port>/*"
    }
  }'

# 2b. Tạo CONFIDENTIAL client (cho backend có server-side)
curl -s -X POST "http://localhost:26201/admin/realms/hoang/clients" \
  -H "Authorization: Bearer $KC_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": "<APP_CLIENT_ID>",
    "name": "<App Display Name>",
    "enabled": true,
    "publicClient": false,
    "clientAuthenticatorType": "client-secret",
    "standardFlowEnabled": true,
    "directAccessGrantsEnabled": false,
    "serviceAccountsEnabled": true,
    "redirectUris": ["https://<app-domain>/*", "http://localhost:<dev-port>/*"],
    "webOrigins": ["https://<app-domain>", "http://localhost:<dev-port>"],
    "protocol": "openid-connect",
    "attributes": {
      "post.logout.redirect.uris": "https://<app-domain>/*##http://localhost:<dev-port>/*"
    }
  }'

# 3. Lấy client secret (chỉ cho confidential client)
CLIENT_UUID=$(curl -s "http://localhost:26201/admin/realms/hoang/clients?clientId=<APP_CLIENT_ID>" \
  -H "Authorization: Bearer $KC_TOKEN" | python3 -c "import sys,json; print(json.load(sys.stdin)[0]['id'])")
curl -s "http://localhost:26201/admin/realms/hoang/clients/$CLIENT_UUID/client-secret" \
  -H "Authorization: Bearer $KC_TOKEN" | python3 -c "import sys,json; print(json.load(sys.stdin)['value'])"
```

### Cách B: Qua Admin Console UI

1. Mở https://auth.hoang.jp/admin/
2. Chọn realm **hoang**
3. Clients → Create client
4. Điền Client ID, chọn type (public / confidential)
5. Set Redirect URIs, Web Origins

### Naming convention

```
Client ID = tên ngắn hoặc subdomain của app
Ví dụ: diagram-editor, note-app, chat-app, excalidraw
```

### Sau khi tạo xong, cập nhật CLIENTS.md

Thêm 1 dòng vào bảng trong `~/Workspace/OSS/keycloak/docs/CLIENTS.md` trên MacbookServer (file có secret: chỉ thêm dòng, không `cat` cả file, không chép ra ngoài).

---

## Bước 2: Tích hợp vào Frontend

### Cách SSO hoạt động

```
User mở App A lần đầu
  → App A redirect đến Keycloak login page
  → User đăng nhập (hoặc dùng Google/GitHub)
  → Keycloak tạo SSO session cookie
  → Redirect về App A với tokens

User mở App B (đã login App A)
  → App B redirect đến Keycloak
  → Keycloak thấy SSO session cookie còn valid
  → Keycloak KHÔNG hiện login page
  → Redirect về App B với tokens ngay lập tức (< 1 giây)
```

**SSO hoạt động vì:** tất cả app dùng chung realm `hoang`, và Keycloak session cookie nằm trên domain `auth.hoang.jp`.

### Tích hợp React / Next.js / Vue (keycloak-js)

**Install:**
```bash
npm install keycloak-js
```

**Tạo file auth:**
```typescript
// src/lib/keycloak.ts
import Keycloak from 'keycloak-js';

const keycloak = new Keycloak({
  url: 'https://auth.hoang.jp',
  realm: 'hoang',
  clientId: '<APP_CLIENT_ID>',  // ← client ID vừa tạo ở bước 1
});

export default keycloak;
```

**Init + Token Refresh (quan trọng!):**
```typescript
// src/lib/auth-init.ts
import keycloak from './keycloak';

let initialized = false;

export async function initAuth(): Promise<boolean> {
  if (initialized) return keycloak.authenticated ?? false;

  const authenticated = await keycloak.init({
    onLoad: 'check-sso',              // check SSO session, không force login
    pkceMethod: 'S256',
    checkLoginIframe: false,           // tắt iframe check (tránh lỗi với Cloudflare)
    silentCheckSsoRedirectUri:
      window.location.origin + '/silent-check-sso.html',
  });

  initialized = true;

  // ======= TOKEN AUTO-REFRESH =======
  // keycloak-js tự gọi refresh_token endpoint khi token sắp hết hạn.
  // Callback này fire khi token expired — ta cố gắng refresh.
  keycloak.onTokenExpired = () => {
    keycloak.updateToken(30)  // refresh nếu token còn < 30 giây
      .catch(() => {
        // Refresh token cũng hết hạn → user phải login lại
        console.warn('Session expired, redirecting to login');
        keycloak.login();
      });
  };

  return authenticated;
}

export function getToken(): string | undefined {
  return keycloak.token;
}

export function login() {
  keycloak.login();
}

export function logout() {
  keycloak.logout({
    redirectUri: window.location.origin,
  });
}

export function getUserInfo() {
  if (!keycloak.tokenParsed) return null;
  return {
    id: keycloak.tokenParsed.sub,
    username: keycloak.tokenParsed.preferred_username,
    email: keycloak.tokenParsed.email,
    roles: keycloak.tokenParsed.realm_access?.roles ?? [],
  };
}
```

**Tạo file `public/silent-check-sso.html`:**
```html
<!DOCTYPE html><html><body><script>parent.postMessage(location.href, location.origin);</script></body></html>
```

**API client với auto-refresh token:**
```typescript
// src/lib/api.ts
import keycloak from './keycloak';

const API_BASE = import.meta.env.VITE_API_URL || '';

export async function apiFetch(path: string, options: RequestInit = {}) {
  // Refresh token nếu sắp hết hạn (còn < 10 giây)
  try {
    await keycloak.updateToken(10);
  } catch {
    keycloak.login();
    throw new Error('Session expired');
  }

  const headers = new Headers(options.headers);
  headers.set('Authorization', `Bearer ${keycloak.token}`);
  if (!headers.has('Content-Type')) {
    headers.set('Content-Type', 'application/json');
  }

  const res = await fetch(`${API_BASE}${path}`, { ...options, headers });

  if (res.status === 401) {
    keycloak.login();
    throw new Error('Unauthorized');
  }

  return res;
}

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

---

## Bước 3: Tích hợp vào Backend

Backend **chỉ validate JWT token** — không redirect login, không refresh token.

### Environment variables

```bash
KEYCLOAK_ISSUER_URL=https://auth.hoang.jp/realms/hoang
KEYCLOAK_JWKS_URL=https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs
```

### Node.js / Express (dùng `jose`)

```bash
npm install jose
```

```typescript
// src/middleware/auth.ts
import { createRemoteJWKSet, jwtVerify } from 'jose';

const ISSUER = process.env.KEYCLOAK_ISSUER_URL || 'https://auth.hoang.jp/realms/hoang';
const JWKS = createRemoteJWKSet(
  new URL(process.env.KEYCLOAK_JWKS_URL || 'https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs')
);

export async function authenticate(req, res, next) {
  const auth = req.headers.authorization;
  if (!auth?.startsWith('Bearer ')) return res.status(401).json({ error: 'No token' });

  try {
    const { payload } = await jwtVerify(auth.slice(7), JWKS, { issuer: ISSUER });
    req.user = {
      id: payload.sub,
      username: payload.preferred_username,
      email: payload.email,
      roles: payload.realm_access?.roles ?? [],
    };
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}

export function requireRole(...roles) {
  return (req, res, next) => {
    if (!roles.some(r => req.user?.roles?.includes(r))) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}
```

> Chi tiết đầy đủ cho Java/Go/Python: xem [INTEGRATION-BACKEND.md](./INTEGRATION-BACKEND.md)

---

## Token Refresh — Cách hoạt động

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TOKEN LIFECYCLE                                 │
│                                                                         │
│  access_token   ██████████░░░░░░░░░░░░░░  (5 phút)                    │
│                           ↑                                             │
│                     sắp hết hạn                                         │
│                     → gọi updateToken()                                 │
│                     → Keycloak trả access_token mới                    │
│                                                                         │
│  refresh_token  ████████████████████████████████░  (30 phút)           │
│                                                     ↑                   │
│                                               hết hạn                   │
│                                               → user phải login lại    │
│                                                                         │
│  SSO session    ████████████████████████████████████████████ (10 giờ)   │
│                 → nếu user login lại, SSO cookie vẫn valid             │
│                 → Keycloak skip login page, trả token ngay             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Frontend tự xử lý:
  1. keycloak.onTokenExpired → keycloak.updateToken(30) → access_token mới
  2. Mỗi API call → keycloak.updateToken(10) trước khi gửi request
  3. Nếu refresh_token cũng hết → redirect login (SSO auto-login nếu < 10 giờ)

Backend KHÔNG làm gì về refresh:
  - Nhận access_token → validate → trả response
  - Token expired → trả 401 → frontend tự refresh rồi retry
```

### Tuỳ chỉnh thời gian token (nếu cần)

> Đây là cấu hình chung của realm `hoang`: phải hỏi Board trước khi đổi (D-0018).

```bash
# Biến KC_TOKEN lấy như Bước 1, Cách A

# Đổi access_token lifespan (ví dụ: 15 phút = 900 giây)
curl -s -X PUT "http://localhost:26201/admin/realms/hoang" \
  -H "Authorization: Bearer $KC_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"accessTokenLifespan": 900}'

# Đổi SSO session idle timeout (ví dụ: 2 giờ)
curl -s -X PUT "http://localhost:26201/admin/realms/hoang" \
  -H "Authorization: Bearer $KC_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ssoSessionIdleTimeout": 7200}'
```

---

## JWT Token Structure

Mọi backend validate token này. Đây là các field quan trọng:

```json
{
  "iss": "https://auth.hoang.jp/realms/hoang",
  "sub": "uuid-of-user",
  "azp": "<client-id>",
  "exp": 1740000000,
  "preferred_username": "thang",
  "email": "thang@hoang.jp",
  "realm_access": {
    "roles": ["user", "admin"]
  }
}
```

| Field | Dùng để |
|-------|---------|
| `sub` | User ID (UUID) — dùng làm foreign key trong DB |
| `iss` | Verify đúng Keycloak instance |
| `exp` | Check token hết hạn chưa |
| `realm_access.roles` | Phân quyền (RBAC) |
| `preferred_username` | Hiển thị tên user |
| `email` | Email user |

---

## Checklist tích hợp app mới

```
[ ] 1. Tạo Keycloak client cho app (public hoặc confidential)
[ ] 2. Set redirect URIs (production + localhost dev)
[ ] 3. Set web origins (CORS)
[ ] 4. Frontend: install keycloak-js, init với check-sso + PKCE
[ ] 5. Frontend: setup token auto-refresh (onTokenExpired + updateToken)
[ ] 6. Frontend: API client tự attach Bearer token
[ ] 7. Frontend: tạo silent-check-sso.html
[ ] 8. Backend: validate JWT bằng JWKS (jose / go-oidc / spring)
[ ] 9. Backend: extract user info từ token claims
[ ] 10. Test: login → API call → refresh → logout → SSO cross-app
[ ] 11. Cập nhật CLIENTS.md
```
