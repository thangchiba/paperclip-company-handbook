# D-0020 — Hoang Auth: trang quản trị Keycloak chỉ mở cho IP global nhà Board; bật chống dò mật khẩu; tắt tài khoản test và 2 client cũ

- **Ngày:** 2026-10-02
- **Phạm vi:** Keycloak Hoang Auth (`auth.hoang.jp`): realm `hoang` (mọi app dùng chung) và realm `master` (quản trị).
- **Nguồn:** Board trả lời card "Hoang Auth: giờ chặn trang quản trị, tài khoản test, 2 secret bị lộ" trên HOA-316 (12:20 JST). CTO ghi lại.
- **Sửa đổi:** Thay lựa chọn "chỉ vào từ LAN" của Board ngày 01/10 (card trước trên HOA-316). Trang quản trị giữ URL cũ, không chuyển sang cổng localhost.

*(Sửa bởi D-0021 mục 8e: việc "nhờ CTO" chuyển cho InfraEngineer.)*

## Bối cảnh

- Trang quản trị `https://auth.hoang.jp/admin/` và realm `master` mở ra Internet.
- Chống dò mật khẩu (brute force detection) đang tắt ở cả 2 realm.
- Một tài khoản test có realm role `admin`, mật khẩu ghi trong tài liệu trên Mac.
- Secret của 2 client cũ `backend-api` và `admin-panel` lọt vào log run nội bộ của CTO. Không ra GitHub. Không app nào dùng 2 client này.
- Theo D-0018, sửa phần dùng chung của Hoang Auth phải hỏi Board trước.

## Phương án đã cân nhắc

- **Trang quản trị:**
  - A: chỉ mở trên Mac ở `http://localhost:26201`, tunnel trả 404 cho `/admin` (CTO đề xuất).
  - B: như A, nhưng làm trong khung 02:00–04:00.
  - C: giữ nguyên.
  - **Board chọn cách khác:** giữ URL cũ, chỉ cho IP global của Board.
- **Tài khoản test và chống dò:** A, tắt tài khoản test và bật chống dò (**Board chọn**). B, chỉ bật chống dò. C, giữ nguyên.
- **2 client cũ:** A, đổi secret. B, tắt hẳn (**Board chọn**). C, giữ nguyên.
- **Đưa skill `hoang-auth` vào thư viện skill của Paperclip:** Board chọn để sau.

## Quyết định

Lời Board về trang quản trị: *"Tôi nghĩ vẫn cho vào admin được nhưng bóp IP global đi, tôi có ip global. Chứ nhớ port đau đầu lắm."*

1. Trang quản trị giữ URL `https://auth.hoang.jp/admin/` và chỉ mở cho IP global ở nhà Board. Nơi khác bị chặn.
2. Bật chống dò mật khẩu ở realm `hoang` và `master`: sai 5 lần thì khoá tạm 15 phút, theo ghi chú setup gốc.
3. Tắt (không xoá) tài khoản test có role `admin`.
4. Tắt hẳn client `backend-api` và `admin-panel`.

## Cách CTO áp dụng (02/10)

- Chặn bằng Cloudflare Access: app "Keycloak admin" của zone `hoang.jp`. Không restart Keycloak hay tunnel.
  - App phủ `auth.hoang.jp/admin*` và `auth.hoang.jp/realms*/master*`. Như vậy cả các biến thể path mà Keycloak vẫn hiểu (ví dụ `//admin`, `/realms;x/master`) cũng bị chặn.
  - Policy Bypass gồm IPv4 nhà Board (/32) và dải IPv6 /64 của LAN nhà, vì trình duyệt trong LAN thường đi IPv6. Mọi IP khác nhận 403.
  - Repo này public nên không ghi IP vào đây.
- Board ở ngoài nhà (IP khác) thì không vào được trang quản trị. Khi cần: nhờ CTO thêm IP vào policy, hoặc dùng `kcadm` / REST qua `localhost:26201` trên Mac.
- IP nhà đổi: sửa policy ở Cloudflare (Zero Trust → Access → Applications → "Keycloak admin"), hoặc nhờ CTO.
- Muốn bật lại 2 client cũ: đổi secret trước, vì secret cũ coi như đã lộ.

## Hệ quả

- Mục "Keycloak hardening" trong checklist trước khi Pro5 bật `both` (HOA-348) đã đạt.
- Cách đăng nhập các app (Google, GitHub, mật khẩu) không đổi.
