---
name: hoang-auth
description: Tích hợp đăng nhập Hoang Auth (Keycloak SSO tại auth.hoang.jp, realm hoang) cho một dự án - tạo client riêng cho dự án, thêm đăng nhập vào frontend và kiểm JWT ở backend. Dùng khi dự án cần đăng nhập hoặc phân quyền, khi tạo hay sửa client Keycloak, hoặc khi review code auth dùng Hoang Auth.
---

# hoang-auth — Tích hợp Hoang Auth (Keycloak SSO)

Nguồn: skill `integrate-auth` của Board trên MacbookServer (`~/.claude/commands/integrate-auth.md`) và bộ docs
`~/Workspace/OSS/keycloak/docs/`, chép lên đây đã bỏ secret (D-0018, HOA-316). Secret, mật khẩu test và danh sách
client chỉ nằm trên Mac.

Theo D-0016: dự án cần đăng nhập thì dùng Hoang Auth, **mỗi dự án một client riêng**, không tự làm auth.

## Keycloak instance

| Mục | Giá trị |
|---|---|
| URL | `https://auth.hoang.jp` |
| Realm | `hoang` (mọi app dùng chung, nên SSO tự chạy giữa các app) |
| Issuer | `https://auth.hoang.jp/realms/hoang` |
| Discovery | `https://auth.hoang.jp/realms/hoang/.well-known/openid-configuration` |
| JWKS | `https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs` |
| Chạy ở | MacbookServer, Docker Compose (`~/Workspace/OSS/keycloak`), cổng local `26201` |
| Credential admin | `~/Workspace/OSS/keycloak/.env` trên Mac. Chỉ dùng trong lệnh, không in, không chép ra khỏi Mac |
| Danh sách client | `~/Workspace/OSS/keycloak/docs/CLIENTS.md` trên Mac. **File có secret: không `cat` cả file** |
| Docs tích hợp | `references/` cạnh file này: `INTEGRATION.md` (tổng quan), `INTEGRATION-BACKEND.md`, `INTEGRATION-FRONTEND.md` |

## Ai làm bước nào

- **Tạo hoặc sửa client (bước 1, 4, 5.2):** InfraEngineer, qua SSH vào MacbookServer (`security-baseline` mục 7).
  Agent khác cần client thì tạo task cho InfraEngineer, ghi rõ: tên client, kiểu app (SPA hay backend tự làm OIDC),
  domain prod, redirect URI, role cần có.
- **Tích hợp code (bước 2–3):** agent dev của dự án.
- **Board quyết** (D-0016 mục 4, D-0018, D-0021): thiết kế khác mặc định ở bước 1 (agent đề xuất A/B/C); mọi sửa
  cấu hình chung (cài đặt realm `hoang`, IdP Google/GitHub, theme, tunnel Cloudflare, thêm người dùng hay đổi chính
  sách đăng ký), vì app ngoài Hoang LLC cũng dùng.

## Bước 0: Phân tích dự án

1. Xác định repo và thư mục đang làm.
2. Xác định tech stack (React/Next.js/Vue, Node, Python, Go, Java…).
3. Tìm domain và port của app (docker-compose, `.env`, `package.json`). Thiếu tên app, domain hay tech stack thì hỏi
   Board trên task trước khi tạo client.
4. Đọc `references/INTEGRATION.md` để hiểu luồng SSO.

## Bước 1: Tạo client

- **Client ID:** tên ngắn của dự án hoặc subdomain (ví dụ `pro5`, `odeku`, `fire`).
- **Mặc định của Hoang LLC** (chặt hơn skill gốc, áp dụng từ HOA-316):
  - `publicClient: true` + PKCE `S256`, cho cả SPA lẫn backend tự làm OIDC. Chỉ dùng confidential khi thật cần;
    khi đó secret đi thẳng vào secret store của dự án, không ghi vào docs hay `CLIENTS.md`.
  - `fullScopeAllowed: false`, để token không mang role của app khác.
  - Quyền admin của app là **client role của chính client đó** (ví dụ `odeku/admin`, nằm ở claim
    `resource_access.<client>.roles`). Không dùng realm role `admin` chung.
  - Tắt `directAccessGrantsEnabled` (password grant), implicit flow và service account nếu không cần.
  - Redirect URI chỉ ghi domain prod, càng cụ thể càng tốt. Không thêm `localhost` vào client prod: test local
    bằng IdP giả, hoặc dùng client dev riêng.
  - `webOrigins` chỉ cần khi SPA gọi token endpoint từ trình duyệt (keycloak-js cần).
  - Backend kiểm `aud`: thêm mapper `oidc-audience-mapper` trỏ về chính client.
  - Trường `description` của client tối đa 255 ký tự (dài hơn thì import trả 500).
- **Cách tạo:** viết file JSON rồi Partial import với `ifResourceExists: FAIL`. Client đã có thì nhận 409, không ghi đè gì.

```bash
# Chạy trên MacbookServer. Mật khẩu admin đi qua stdin, không nằm trong argv (D-0015).
cd ~/Workspace/OSS/keycloak
kcenv() { grep "^$1=" .env | cut -d= -f2-; }   # đọc 1 biến trong .env, không in ra màn hình
KC_TOKEN=$(kcenv KC_ADMIN_PASSWORD | tr -d '\n' \
  | curl -s -X POST "http://localhost:26201/realms/master/protocol/openid-connect/token" \
      -d "grant_type=password" -d "client_id=admin-cli" \
      --data-urlencode "username=$(kcenv KC_ADMIN_USER)" \
      --data-urlencode "password@-" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

# clients.json: {"ifResourceExists": "FAIL", "clients": [...], "roles": {"client": {"<client>": [...]}}}
curl -s -X POST "http://localhost:26201/admin/realms/hoang/partialImport" \
  -H "Authorization: Bearer $KC_TOKEN" -H "Content-Type: application/json" \
  --data @clients.json
```

- Ví dụ JSON đã chạy trên prod: client `pro5` (backend tự làm OIDC) và `odeku` (SPA, có role `admin`, audience
  mapper), đính kèm ở HOA-316 (`hoang-auth-clients-hoa316-v3.json`).
- Rollback: xoá client vừa tạo (`DELETE /admin/realms/hoang/clients/<uuid>`), khi chưa có app nào dùng nó.

## Bước 2: Tích hợp frontend (nếu có)

Chi tiết: `references/INTEGRATION.md` (Bước 2) và `references/INTEGRATION-FRONTEND.md`.

1. `npm install keycloak-js`.
2. Tạo `keycloak.ts` với `url`, `realm`, `clientId`.
3. Tạo `auth-init.ts`: init với `pkceMethod: 'S256'` và tự refresh token.
4. Tạo `api.ts`: wrapper `fetch` tự gắn `Authorization: Bearer <token>`.
5. Gắn vào entry point của app. Trang admin dùng `onLoad: 'login-required'`.

**Quan trọng — refresh token:**
- Đăng ký callback `keycloak.onTokenExpired`.
- Gọi `keycloak.updateToken(30)` trước mỗi lần gọi API.
- Refresh lỗi thì chuyển sang login. Nếu phiên SSO còn, Keycloak tự đăng nhập lại mà không hỏi mật khẩu.
- Không dựa vào silent check-sso bằng iframe khi app khác site với `auth.hoang.jp`: trình duyệt chặn cookie bên thứ ba.

## Bước 3: Tích hợp backend (nếu có)

Chi tiết: `references/INTEGRATION.md` (Bước 3) và `references/INTEGRATION-BACKEND.md` (Java, Go, Node, Python).

1. Kiểm JWT bằng JWKS (có cache): chữ ký, `iss`, `exp`, và `aud` hoặc `azp` đúng client của app.
2. Lấy thông tin user từ claim (`sub`, `email`, `email_verified`).
3. Phân quyền theo `resource_access.<client>.roles` (client role của app). Không coi "đã đăng nhập" hay
   realm role `user` là đủ quyền: ai có tài khoản Google cũng tự tạo được tài khoản trong realm và nhận
   role mặc định, gồm `user` (mục Ghi chú).
4. Backend không refresh token: token hết hạn thì trả 401.
5. Backend tự làm OIDC rồi cấp session riêng (kiểu Pro5): dùng thư viện có chứng nhận (`openid-client` cho Node,
   Authlib cho Python). Kiểm state, nonce, PKCE và `iss` trong callback. Không tự viết phần kiểm JWT.

Env cho backend:

```
KEYCLOAK_ISSUER_URL=https://auth.hoang.jp/realms/hoang
KEYCLOAK_JWKS_URL=https://auth.hoang.jp/realms/hoang/protocol/openid-connect/certs
```

## Bước 4: Cập nhật danh sách client

Thêm 1 dòng vào bảng `Realm: hoang` trong `~/Workspace/OSS/keycloak/docs/CLIENTS.md` trên Mac. Chỉ thêm dòng
bằng script, không in cả file. Không ghi secret vào file này.

## Bước 5: Kiểm thử

1. Mở URL auth với PKCE: phải hiện trang đăng nhập Hoang Auth, không phải `Client not found`.
   Redirect URI lạ phải bị từ chối (400 `Invalid parameter: redirect_uri`).
2. Xem token mẫu mà không cần đăng nhập (trên Mac):
   `GET /admin/realms/hoang/clients/<uuid>/evaluate-scopes/generate-example-access-token?scope=openid&userId=<user-id>`.
   Kiểm `aud`, `azp`, role, và không có role của app khác.
3. Đăng nhập thật trên trình duyệt với một tài khoản đã có trong realm.

## Ghi chú

- Mọi app dùng chung realm `hoang`: đăng nhập 1 app thì các app khác cũng nhận phiên SSO.
- Google và GitHub đăng nhập được mà frontend không phải viết code riêng.
- Người dùng mới (từ 03/10/2026; Board chọn phương án B trên card `744a1e09` ở HOA-396, làm ở HOA-409):
  - Google: ai có tài khoản Google với email đã xác minh đều tự tạo được tài khoản ở lần đăng nhập đầu.
    IdP `google` dùng flow `first broker login` và essential claim `email_verified` = `true`.
  - GitHub: chỉ liên kết với tài khoản đã có (`invite-only-broker-login`), vì GitHub có thể đưa email
    chưa xác minh (HOA-360, K7).
  - Form đăng ký bằng mật khẩu vẫn tắt.
  - User mới nhận role mặc định của realm (gồm realm role `user`) và đăng nhập được mọi client của realm.
    App nào cần giới hạn người dùng phải tự kiểm client role của mình.
