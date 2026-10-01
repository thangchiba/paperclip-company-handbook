# Backend Integration Manual

Hướng dẫn chi tiết tích hợp Keycloak auth vào Backend API.
Document này đủ chi tiết để AI hoặc developer implement từ đầu đến cuối.

> **Prerequisite:** Đọc [INTEGRATION.md](./INTEGRATION.md) để hiểu auth flow tổng quan.

---

## Mục lục

1. [Nguyên tắc chung](#1-nguyên-tắc-chung)
2. [Java / Spring Boot](#2-java--spring-boot)
3. [Go](#3-go)
4. [Node.js / Express](#4-nodejs--express)
5. [Python / FastAPI](#5-python--fastapi)
6. [Các pattern nâng cao](#6-các-pattern-nâng-cao)
7. [Testing & Debug](#7-testing--debug)

---

## 1. Nguyên tắc chung

### Backend KHÔNG redirect login

Backend API **chỉ validate JWT token**, không redirect user đến login page.
Frontend chịu trách nhiệm lấy token từ Keycloak rồi gửi kèm request.

### Request flow

```
Client gửi request:
  GET /api/resource
  Authorization: Bearer <access_token>

Backend xử lý:
  1. Extract token từ header Authorization
  2. Validate JWT signature bằng JWKS public keys
  3. Check token chưa expired (exp)
  4. Check issuer đúng (iss)
  5. Extract user info & roles từ token claims
  6. Authorize dựa trên roles
  7. Trả response hoặc 401/403
```

### Keycloak config cho Backend

```
Keycloak URL:    https://auth.hoang.jp
Realm:           hoang
Client ID:       backend-api
Client Secret:   (chỉ client confidential; lấy từ secret store của dự án, không ghi vào docs)
Issuer URL:      https://auth.hoang.jp/realms/hoang
JWKS URL:        https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs
```

### Environment variables chuẩn

Mọi backend service nên đọc config từ env:

```bash
KEYCLOAK_ISSUER_URL=https://auth.hoang.jp/realms/hoang
KEYCLOAK_JWKS_URL=https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs
KEYCLOAK_CLIENT_ID=backend-api
# Chỉ client confidential: giá trị lấy từ secret store của dự án, không ghi vào file
KEYCLOAK_CLIENT_SECRET=
```

---

## 2. Java / Spring Boot

### 2.1 Dependencies

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

Hoặc Gradle:
```groovy
implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
```

### 2.2 Application config

```yaml
# application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${KEYCLOAK_ISSUER_URL:https://auth.hoang.jp/realms/hoang}
          jwk-set-uri: ${KEYCLOAK_JWKS_URL:https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs}
```

### 2.3 Security Configuration

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.web.SecurityFilterChain;

import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // enables @PreAuthorize
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())  // API không cần CSRF
            .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**", "/health", "/actuator/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("admin")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter()))
            );
        return http.build();
    }

    /**
     * Converter để map Keycloak realm_access.roles thành Spring Security authorities.
     * Keycloak trả roles trong claim "realm_access.roles" thay vì "scope" mặc định.
     */
    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(new KeycloakRealmRoleConverter());
        return converter;
    }

    static class KeycloakRealmRoleConverter implements Converter<Jwt, Collection<GrantedAuthority>> {
        @Override
        @SuppressWarnings("unchecked")
        public Collection<GrantedAuthority> convert(Jwt jwt) {
            Map<String, Object> realmAccess = jwt.getClaimAsMap("realm_access");
            if (realmAccess == null || !realmAccess.containsKey("roles")) {
                return Collections.emptyList();
            }
            List<String> roles = (List<String>) realmAccess.get("roles");
            return roles.stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .collect(Collectors.toList());
        }
    }
}
```

### 2.4 Lấy user info trong Controller

```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class ApiController {

    // Bất kỳ authenticated user
    @GetMapping("/profile")
    public Map<String, Object> getProfile(@AuthenticationPrincipal Jwt jwt) {
        return Map.of(
            "userId", jwt.getSubject(),                          // Keycloak user UUID
            "username", jwt.getClaimAsString("preferred_username"),
            "email", jwt.getClaimAsString("email"),
            "name", jwt.getClaimAsString("given_name") + " " + jwt.getClaimAsString("family_name")
        );
    }

    // Chỉ admin
    @PreAuthorize("hasRole('admin')")
    @GetMapping("/admin/users")
    public String adminOnly() {
        return "admin content";
    }

    // User hoặc admin
    @PreAuthorize("hasAnyRole('user', 'admin')")
    @GetMapping("/data")
    public String userData() {
        return "user data";
    }
}
```

### 2.5 CORS Configuration (nếu FE gọi trực tiếp)

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOriginPatterns(List.of("https://*.hoang.jp", "http://localhost:*"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
    config.setAllowCredentials(true);

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/**", config);
    return source;
}
```

---

## 3. Go

### 3.1 Dependencies

```bash
go get github.com/coreos/go-oidc/v3/oidc
go get github.com/golang-jwt/jwt/v5
```

### 3.2 Auth middleware (full implementation)

```go
package auth

import (
	"context"
	"log"
	"net/http"
	"os"
	"strings"

	"github.com/coreos/go-oidc/v3/oidc"
)

// Claims chứa thông tin user từ Keycloak JWT token
type Claims struct {
	Sub               string      `json:"sub"`
	PreferredUsername  string      `json:"preferred_username"`
	Email             string      `json:"email"`
	GivenName         string      `json:"given_name"`
	FamilyName        string      `json:"family_name"`
	RealmAccess       RealmAccess `json:"realm_access"`
}

type RealmAccess struct {
	Roles []string `json:"roles"`
}

// HasRole kiểm tra user có role cụ thể không
func (c *Claims) HasRole(role string) bool {
	for _, r := range c.RealmAccess.Roles {
		if r == role {
			return true
		}
	}
	return false
}

type contextKey string

const ClaimsKey contextKey = "claims"

// Verifier wraps OIDC token verification
type Verifier struct {
	verifier *oidc.IDTokenVerifier
}

// NewVerifier tạo JWT verifier từ Keycloak OIDC discovery
func NewVerifier() (*Verifier, error) {
	issuerURL := os.Getenv("KEYCLOAK_ISSUER_URL")
	if issuerURL == "" {
		issuerURL = "https://auth.hoang.jp/realms/hoang"
	}
	clientID := os.Getenv("KEYCLOAK_CLIENT_ID")
	if clientID == "" {
		clientID = "backend-api"
	}

	provider, err := oidc.NewProvider(context.Background(), issuerURL)
	if err != nil {
		return nil, err
	}

	verifier := provider.Verifier(&oidc.Config{
		// Keycloak đặt audience = "account" mặc định, không phải client ID.
		// Skip audience check ở đây, validate iss + signature là đủ.
		// Hoặc configure audience mapping trong Keycloak nếu cần strict check.
		SkipClientIDCheck: true,
	})

	return &Verifier{verifier: verifier}, nil
}

// Middleware xác thực JWT token
func (v *Verifier) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		tokenStr := extractBearerToken(r)
		if tokenStr == "" {
			http.Error(w, `{"error":"missing authorization header"}`, http.StatusUnauthorized)
			return
		}

		idToken, err := v.verifier.Verify(r.Context(), tokenStr)
		if err != nil {
			log.Printf("Token verification failed: %v", err)
			http.Error(w, `{"error":"invalid token"}`, http.StatusUnauthorized)
			return
		}

		var claims Claims
		if err := idToken.Claims(&claims); err != nil {
			http.Error(w, `{"error":"invalid claims"}`, http.StatusUnauthorized)
			return
		}

		ctx := context.WithValue(r.Context(), ClaimsKey, &claims)
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}

// RequireRole middleware kiểm tra role
func RequireRole(role string, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		claims := GetClaims(r.Context())
		if claims == nil || !claims.HasRole(role) {
			http.Error(w, `{"error":"forbidden"}`, http.StatusForbidden)
			return
		}
		next.ServeHTTP(w, r)
	})
}

// GetClaims lấy claims từ context (sau khi đã qua auth middleware)
func GetClaims(ctx context.Context) *Claims {
	claims, _ := ctx.Value(ClaimsKey).(*Claims)
	return claims
}

func extractBearerToken(r *http.Request) string {
	auth := r.Header.Get("Authorization")
	if !strings.HasPrefix(auth, "Bearer ") {
		return ""
	}
	return strings.TrimPrefix(auth, "Bearer ")
}
```

### 3.3 Sử dụng trong main.go

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"

	"yourproject/auth"
)

func main() {
	verifier, err := auth.NewVerifier()
	if err != nil {
		log.Fatal("Failed to create OIDC verifier:", err)
	}

	mux := http.NewServeMux()

	// Public endpoint - không cần auth
	mux.HandleFunc("GET /health", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte(`{"status":"ok"}`))
	})

	// Protected endpoints - wrap với auth middleware
	protected := http.NewServeMux()
	protected.HandleFunc("GET /api/profile", profileHandler)
	protected.Handle("GET /api/admin/users", auth.RequireRole("admin", http.HandlerFunc(adminHandler)))

	mux.Handle("/api/", verifier.Middleware(protected))

	log.Println("Server started on :8080")
	log.Fatal(http.ListenAndServe(":8080", mux))
}

func profileHandler(w http.ResponseWriter, r *http.Request) {
	claims := auth.GetClaims(r.Context())
	json.NewEncoder(w).Encode(map[string]any{
		"userId":   claims.Sub,
		"username": claims.PreferredUsername,
		"email":    claims.Email,
		"roles":    claims.RealmAccess.Roles,
	})
}

func adminHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte(`{"message":"admin only content"}`))
}
```

---

## 4. Node.js / Express

### 4.1 Dependencies

```bash
npm install jose        # JWT validation (lightweight, no Keycloak SDK needed)
```

> Dùng `jose` thay vì `keycloak-connect` vì nhẹ hơn, ít dependency, không lock vào Keycloak SDK.

### 4.2 Auth middleware (full implementation)

```typescript
// src/middleware/auth.ts
import { createRemoteJWKSet, jwtVerify, JWTPayload } from 'jose';
import { Request, Response, NextFunction } from 'express';

const ISSUER_URL = process.env.KEYCLOAK_ISSUER_URL || 'https://auth.hoang.jp/realms/hoang';
const JWKS_URL = process.env.KEYCLOAK_JWKS_URL || 'https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs';

// JWKS tự cache & rotate keys
const JWKS = createRemoteJWKSet(new URL(JWKS_URL));

// Extend JWT payload with Keycloak-specific fields
interface KeycloakTokenPayload extends JWTPayload {
  preferred_username?: string;
  email?: string;
  given_name?: string;
  family_name?: string;
  realm_access?: {
    roles: string[];
  };
}

// Extend Express Request
declare global {
  namespace Express {
    interface Request {
      user?: {
        id: string;           // sub (Keycloak user UUID)
        username: string;
        email: string;
        firstName: string;
        lastName: string;
        roles: string[];
        token: KeycloakTokenPayload;
      };
    }
  }
}

/**
 * Middleware xác thực JWT token từ Keycloak.
 * Verify signature bằng JWKS, check issuer, extract user info.
 */
export async function authenticate(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing authorization header' });
  }

  const token = authHeader.slice(7);

  try {
    const { payload } = await jwtVerify(token, JWKS, {
      issuer: ISSUER_URL,
    }) as { payload: KeycloakTokenPayload };

    req.user = {
      id: payload.sub!,
      username: payload.preferred_username || '',
      email: payload.email || '',
      firstName: payload.given_name || '',
      lastName: payload.family_name || '',
      roles: payload.realm_access?.roles || [],
      token: payload,
    };

    next();
  } catch (err) {
    console.error('JWT verification failed:', err);
    return res.status(401).json({ error: 'Invalid token' });
  }
}

/**
 * Middleware kiểm tra role. Dùng sau authenticate().
 * Usage: app.get('/admin', authenticate, requireRole('admin'), handler)
 */
export function requireRole(...roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    const userRoles = req.user?.roles || [];
    const hasRole = roles.some(role => userRoles.includes(role));

    if (!hasRole) {
      return res.status(403).json({ error: 'Forbidden', requiredRoles: roles });
    }
    next();
  };
}
```

### 4.3 Sử dụng trong app

```typescript
// src/app.ts
import express from 'express';
import cors from 'cors';
import { authenticate, requireRole } from './middleware/auth';

const app = express();

// CORS
app.use(cors({
  origin: [/\.hoang\.jp$/, /^http:\/\/localhost:\d+$/],
  credentials: true,
}));

app.use(express.json());

// Public
app.get('/health', (req, res) => res.json({ status: 'ok' }));

// Protected - any authenticated user
app.get('/api/profile', authenticate, (req, res) => {
  res.json({
    userId: req.user!.id,
    username: req.user!.username,
    email: req.user!.email,
    roles: req.user!.roles,
  });
});

// Protected - admin only
app.get('/api/admin/users', authenticate, requireRole('admin'), (req, res) => {
  res.json({ message: 'admin content' });
});

// Protected - admin or moderator
app.delete('/api/posts/:id', authenticate, requireRole('admin', 'moderator'), (req, res) => {
  res.json({ message: `Deleted post ${req.params.id}` });
});

app.listen(8080, () => console.log('Server on :8080'));
```

---

## 5. Python / FastAPI

### 5.1 Dependencies

```bash
pip install python-jose[cryptography] httpx
```

### 5.2 Auth dependency (full implementation)

```python
# auth.py
import os
from typing import Optional
from dataclasses import dataclass, field

import httpx
from jose import jwt, JWTError
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

ISSUER_URL = os.getenv("KEYCLOAK_ISSUER_URL", "https://auth.hoang.jp/realms/hoang")
JWKS_URL = os.getenv("KEYCLOAK_JWKS_URL", "https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs")

security = HTTPBearer()

# Cache JWKS keys
_jwks_cache: Optional[dict] = None


async def get_jwks() -> dict:
    """Fetch và cache JWKS public keys từ Keycloak."""
    global _jwks_cache
    if _jwks_cache is None:
        async with httpx.AsyncClient() as client:
            resp = await client.get(JWKS_URL)
            _jwks_cache = resp.json()
    return _jwks_cache


@dataclass
class User:
    """User info extracted từ Keycloak JWT token."""
    id: str                          # sub (UUID)
    username: str
    email: str
    first_name: str
    last_name: str
    roles: list[str] = field(default_factory=list)

    def has_role(self, role: str) -> bool:
        return role in self.roles


async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
) -> User:
    """
    FastAPI dependency: validate JWT token, return User object.
    Usage: @app.get("/api/data")
           async def endpoint(user: User = Depends(get_current_user)):
    """
    token = credentials.credentials

    try:
        jwks = await get_jwks()
        # Decode header to get kid
        unverified_header = jwt.get_unverified_header(token)
        # Find matching key
        key = None
        for k in jwks["keys"]:
            if k["kid"] == unverified_header.get("kid"):
                key = k
                break
        if key is None:
            raise HTTPException(status_code=401, detail="Invalid token key")

        payload = jwt.decode(
            token,
            key,
            algorithms=["RS256"],
            issuer=ISSUER_URL,
            options={"verify_aud": False},  # Keycloak audience = "account"
        )
    except JWTError as e:
        raise HTTPException(status_code=401, detail=f"Invalid token: {e}")

    realm_access = payload.get("realm_access", {})

    return User(
        id=payload.get("sub", ""),
        username=payload.get("preferred_username", ""),
        email=payload.get("email", ""),
        first_name=payload.get("given_name", ""),
        last_name=payload.get("family_name", ""),
        roles=realm_access.get("roles", []),
    )


def require_role(*roles: str):
    """
    FastAPI dependency: kiểm tra role sau authenticate.
    Usage: @app.get("/admin", dependencies=[Depends(require_role("admin"))])
    """
    async def _check(user: User = Depends(get_current_user)):
        if not any(user.has_role(r) for r in roles):
            raise HTTPException(
                status_code=403,
                detail=f"Required roles: {roles}",
            )
        return user
    return _check
```

### 5.3 Sử dụng trong app

```python
# main.py
from fastapi import FastAPI, Depends
from fastapi.middleware.cors import CORSMiddleware

from auth import User, get_current_user, require_role

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origin_regex=r"(https://.*\.hoang\.jp|http://localhost:\d+)",
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/health")
async def health():
    return {"status": "ok"}


@app.get("/api/profile")
async def profile(user: User = Depends(get_current_user)):
    return {
        "userId": user.id,
        "username": user.username,
        "email": user.email,
        "roles": user.roles,
    }


@app.get("/api/admin/users")
async def admin_users(user: User = Depends(require_role("admin"))):
    return {"message": "admin content", "requestedBy": user.username}


@app.delete("/api/posts/{post_id}")
async def delete_post(
    post_id: str,
    user: User = Depends(require_role("admin", "moderator")),
):
    return {"message": f"Deleted post {post_id}"}
```

---

## 6. Các pattern nâng cao

### 6.1 Service-to-Service Authentication (Machine-to-Machine)

Khi backend A cần gọi backend B mà không có user context.
Dùng **Client Credentials Grant** với client `backend-api`.

```bash
# Lấy token cho service account
curl -X POST "https://auth.hoang.jp/realms/hoang/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=backend-api" \
  -d "client_secret=<secret>"
```

```typescript
// Node.js example
async function getServiceToken(): Promise<string> {
  const resp = await fetch('https://auth.hoang.jp/realms/hoang/protocol/openid-connect/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'client_credentials',
      client_id: 'backend-api',
      client_secret: process.env.KEYCLOAK_CLIENT_SECRET!,
    }),
  });
  const data = await resp.json();
  return data.access_token;
}

// Gọi service khác
const token = await getServiceToken();
const resp = await fetch('http://other-service/api/internal', {
  headers: { Authorization: `Bearer ${token}` },
});
```

### 6.2 User ID làm Foreign Key trong Database

Dùng `sub` (UUID) từ JWT token làm foreign key thay vì tạo user table riêng.

```sql
-- Ví dụ: posts table
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    author_id UUID NOT NULL,         -- = JWT sub (Keycloak user ID)
    title TEXT NOT NULL,
    content TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Query posts của user hiện tại
-- Backend lấy author_id từ req.user.id (= JWT sub)
SELECT * FROM posts WHERE author_id = $1;
```

### 6.3 Caching JWKS Keys

JWKS keys thay đổi rất ít (chỉ khi Keycloak rotate keys). Nên cache:

- **jose (Node.js):** Tự cache, tự rotate khi gặp unknown kid
- **go-oidc (Go):** Tự cache
- **python-jose:** Cần tự implement cache (xem code ở trên)
- **Spring Boot:** Tự cache qua NimbusJwtDecoder

### 6.4 Handling Token Expiration

- Access token expire sau **5 phút** (cấu hình trong realm)
- Backend trả 401 khi token expired → Frontend tự refresh token
- Backend **KHÔNG** refresh token — đó là việc của Frontend

### 6.5 Multi-tenant (nhiều realm)

Nếu sau này cần nhiều realm, sửa middleware để đọc issuer từ token dynamically:

```typescript
// Thay vì hardcode 1 issuer, verify từ token
const { payload } = await jwtVerify(token, JWKS, {
  issuer: ['https://auth.hoang.jp/realms/hoang', 'https://auth.hoang.jp/realms/other'],
});
```

---

## 7. Testing & Debug

### 7.1 Lấy test token nhanh

```bash
# Password grant (chỉ cho test/dev, KHÔNG dùng cho production flow). Client của Hoang LLC tắt password grant;
# muốn xem token mẫu thì dùng evaluate-scopes (SKILL.md, bước 5). Mật khẩu test đi qua stdin.
curl -s -X POST "https://auth.hoang.jp/realms/hoang/protocol/openid-connect/token" \
  -d "grant_type=password" \
  -d "client_id=backend-api" \
  -d "client_secret=<secret>" \
  -d "username=$TEST_USER" \
  --data-urlencode "password@-" | python3 -m json.tool
```

### 7.2 Decode token (không verify)

```bash
# Paste access_token vào đây
echo "<token>" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool
```

Hoặc dùng https://jwt.io (paste token để xem claims).

### 7.3 Test API với curl

```bash
# 1. Lấy token
TOKEN=$(curl -s -X POST "https://auth.hoang.jp/realms/hoang/protocol/openid-connect/token" \
  -d "grant_type=password" \
  -d "client_id=backend-api" \
  -d "client_secret=<secret>" \
  -d "username=$TEST_USER" \
  --data-urlencode "password@-" | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

# 2. Gọi API
curl -H "Authorization: Bearer $TOKEN" http://localhost:8080/api/profile

# 3. Test 401 (no token)
curl http://localhost:8080/api/profile
# Expected: 401 Unauthorized

# 4. Test 403 (wrong role)
curl -H "Authorization: Bearer $TOKEN" http://localhost:8080/api/admin/users
# Expected: 200 if user has admin role, 403 if not
```

### 7.4 Common Errors

| Error | Nguyên nhân | Fix |
|-------|-------------|-----|
| `401 Invalid token` | Token expired hoặc signature sai | Lấy token mới, check issuer URL |
| `401 Missing authorization header` | Không gửi Bearer token | Thêm `Authorization: Bearer <token>` |
| `403 Forbidden` | User không có role cần thiết | Gán role trong Keycloak Admin Console |
| `JWKS fetch failed` | Backend không connect được Keycloak | Check network, DNS, Keycloak running |
| `Issuer mismatch` | issuer trong token ≠ config | Đảm bảo dùng `https://auth.hoang.jp/realms/hoang` |
