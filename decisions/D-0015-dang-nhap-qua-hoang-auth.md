# D-0015 — Dự án của Hoang đăng nhập qua Hoang Auth, mỗi dự án một client

- **Ngày:** 2026-10-01
- **Phạm vi:** Toàn công ty, cho auth của các dự án. Áp dụng trước cho Project Pro5 (repo `thangchiba/pro5`) và Project Figure (repo `thangchiba/odeku`, sản phẩm Odeku).
- **Nguồn:** Board tạo HOA-316 và giao CTO. CTO ghi lại quyết định này và làm phần kỹ thuật tại HOA-316 (ADR trong document `adr` của issue).
- **Sửa đổi:** Không.

## Bối cảnh

- Board thường dùng **Hoang Auth** cho các dự án của mình. Hoang Auth là Keycloak tại `https://auth.hoang.jp`, realm `hoang`, hiện chỉ đang thử nghiệm nội bộ.
- Lúc Board quyết, mỗi dự án tự lo đăng nhập:
  - Pro5 dùng magic link gửi qua SES. Admin là danh sách email trong SSM.
  - Odeku: staff đăng nhập qua Cloudflare Access bằng OTP. Khách không có tài khoản.

## Phương án đã cân nhắc

- A: Giữ cách đăng nhập riêng của từng dự án.
- B: Dùng Hoang Auth, nhưng mọi dự án chung một client.
- C: Dùng Hoang Auth, mỗi dự án một client riêng. **Board chọn.**

## Quyết định

1. **Auth của các dự án chuyển sang dùng Hoang Auth.**
2. **Pro5 và Odeku đăng nhập bằng Hoang Auth.**
3. **Mỗi dự án có client riêng** trong realm `hoang`.
4. Thiết kế chi tiết do CTO quyết, ghi trong ADR của HOA-316. Gồm: loại client, redirect URI, cách xác định admin, thứ tự rollout.

## Hệ quả

- Dự án mới cần đăng nhập thì dùng Hoang Auth với client riêng, không tự làm auth.
- Agent cần quyền trên Keycloak để tạo client. CTO hỏi Board bằng card trên HOA-316.
- Board để skill triển khai Hoang Auth trên máy Mac. Agent cần đọc được skill này để làm đúng quy ước của Board.
