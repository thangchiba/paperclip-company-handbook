---
name: secret-hygiene
description: Quy tắc không để secret và cả "hình dạng secret" (user:pass@host, PASSWORD=..., ${VAR:?msg}) lọt vào git, log hay Paperclip; tự quét trước mỗi commit và cách xử lý khi GitGuardian báo đỏ. Dùng cho mọi agent khi viết code, compose/CI/Terraform, docs, .env.example, test, hoặc khi in biến môi trường.
---

# secret-hygiene

Bổ sung cho `handbook/rules.md` mục 4 (không có giá trị secret ở bất kỳ đâu). Ra đời sau
HOA-177: GitGuardian chặn PR #7 của odeku vì `deploy/compose.yaml` chứa
`postgresql://figure:${POSTGRES_PASSWORD:?…}@db` — **không có secret thật**, nhưng scanner
đọc chuỗi đó như một mật khẩu. Kết luận: cấm cả *hình dạng* secret, không chỉ giá trị.

## 1. Cấm trong mọi file được commit (kể cả comment, docs, test, .env.example)

| Cấm | Vì sao | Viết thay bằng |
|---|---|---|
| `scheme://user:ANYTHING@host` — dù ANYTHING là `${VAR}`, `<password>`, `change-me`, `xxx` | mọi scanner bắt URI có credential | URL **không có password** (`postgresql://figure@db/figure`) + password đưa riêng qua `*_FILE` hoặc biến env đọc từ file ngoài git |
| `PASSWORD/SECRET/TOKEN/KEY: ${VAR:?thông báo}` hoặc `${VAR:-giá trị}` | scanner coi phần sau `:?` / `:-` là giá trị thật | `${VAR}` trần, ghi yêu cầu vào comment; hoặc `*_FILE` |
| `PASSWORD=change-me`, `TOKEN=my-secret-token`, `key = "sk_test_…"` trong example/test | placeholder tự nghĩ ra không nằm trong danh sách placeholder của scanner | để **trống** (`PASSWORD=`) và ghi cách sinh giá trị trong comment; test thì build chuỗi bằng thư viện (`make_url(...).set(password=...)`), không viết literal |
| Giá trị thật ở bất kỳ dạng nào: key, token, connection string, cookie, private key, `.env` thật | lộ = phải rotate | secret manager / file ngoài git (mode 600), agent chỉ tham chiếu **tên** biến (D-0005) |

Ưu tiên cơ chế **file-based secret**: Docker/Compose `secrets:` + `POSTGRES_PASSWORD_FILE`,
`FIGURE_DATABASE_PASSWORD_FILE`; Terraform `random_password` + SSM SecureString
(`terraform-plan-only`); CI dùng secret store của CI, không hardcode.

## 2. Tự quét trước mỗi commit (bắt buộc, không cần cài gì)

```bash
# Quét các dòng THÊM MỚI đã stage (bỏ lockfile). Bỏ qua `*_FILE=` và `${VAR}` / `${VAR:-}` trần.
d=$(git diff --cached -U0 -- . ':!*.lock' ':!package-lock.json' | grep -E '^\+[^+]' \
    | grep -vE '_FILE[[:space:]]*[=:]|\$\{[A-Z_]+(:-)?\}[[:space:]]*$')
printf '%s\n' "$d" | grep -nE '://[^/@[:space:]]+:[^@[:space:]]+@|(PASSWORD|PASSWD|SECRET|TOKEN|API_KEY|PRIVATE_KEY)[A-Z_]*[[:space:]]*[=:][[:space:]]*[^[:space:]$]|\$\{[A-Z_]+:[?-][^}]+\}|AKIA[0-9A-Z]{16}|sk_(live|test)_|whsec_|-----BEGIN [A-Z ]*PRIVATE KEY|eyJ[A-Za-z0-9_-]{20,}\.eyJ|[A-Fa-f0-9]{48,}'
printf '%s\n' "$d" | grep -inE '(password|passwd|secret|token|api_key|private_key)[a-z_]*[^=]{0,20}=[[:space:]]*["'"'"'][^"'"'"']{4,}["'"'"']'
```

- Có dòng khớp → sửa cho tới khi hai lệnh grep không in gì. Dòng chỉ có `KEY=` trống, `${VAR}` trần
  hoặc `*_FILE=<path>` là hợp lệ. Fixture test kiểu `"test-admin-token"` được phép **nếu** không phải
  URI có credential và không giống giá trị thật (hex dài, prefix `sk_`, JWT…) — ghi rõ trong PR.
- Đã kiểm chứng (HOA-177): bắt đúng cả hai dòng GitGuardian báo trên `deploy/compose.yaml`, không
  bắt các dòng đã sửa (`…://figure@db`, `POSTGRES_PASSWORD_FILE`, `${CLOUDFLARE_TUNNEL_TOKEN}`).
- Repo có `ggshield`/`gitleaks` thì chạy thêm (`ggshield secret scan pre-commit`, `gitleaks protect --staged`).
- Đây là một phần của "self-review" trong `pr-standard` mục 4.

## 3. Khi GitGuardian / scanner báo đỏ trên PR

1. **Không** merge khi check đỏ, **không** tự đánh dấu "false positive" trên dashboard (chỉ Board có quyền).
2. Đọc comment của bot: nó ghi rõ file + dòng (`R27`) + loại (`Generic Password`, `PostgreSQL URI`…).
3. Phân loại:
   - **Giá trị thật lọt** → báo Board ngay trong task (không dán lại giá trị), đề xuất rotate trước rồi mới sửa code (rules.md 4.4).
   - **Chỉ là hình dạng** (interpolation, placeholder) → sửa theo bảng mục 1 và commit mới.
4. Check của GitGuardian quét **mọi commit trong PR**, nên commit sửa không làm check xanh nếu commit cũ vẫn chứa chuỗi. Báo Board chọn: (A) Board đánh dấu incident là false positive trên dashboard, hoặc (B) Board duyệt rewrite/squash branch (force-push cần Board duyệt — rules.md mục 3). Đề xuất mặc định: A khi không có giá trị thật.
5. Ghi lại vào task theo khung ①②③: incident id, file/dòng, thật hay hình dạng, đã sửa bằng gì.

## 4. Không in secret ra log / chat / Paperclip

- Không chạy `env`, `printenv`, `set`, `export -p`, `cat /proc/*/environ`, không `echo $SSH_ORIGINAL_COMMAND`
  (HOA-130/136: biến này chứa credential AWS/GitHub/Paperclip dạng plaintext). Cần biết biến nào tồn tại:
  `env | sed 's/=.*/=<set>/'`.
- Khi in file cấu hình, connection string, header HTTP hay output `docker compose config`: mask trước
  (`sed -E 's#(://[^:@/]+:)[^@]+@#\1<masked>@#g'`).
- Comment/document trên Paperclip chỉ ghi **tên** biến, path của secret file, incident id — không bao giờ ghi giá trị.

## Checklist 10 giây trước khi `git commit`

- [ ] Không có `user:…@host` ở bất kỳ file nào trong diff.
- [ ] Không có `PASSWORD/TOKEN/SECRET/KEY = <giá trị>` — chỉ `=` trống, `${VAR}` trần, hoặc `*_FILE` path.
- [ ] Lệnh quét mục 2 không in ra gì.
- [ ] `.env`, `*.pem`, dump DB, file secret không nằm trong `git status`.
