# D-0018 — CTO được SSH vào MacbookServer; sửa thứ gì ngoài dự án Hoang LLC phải báo Board trước

- **Ngày:** 2026-10-01
- **Phạm vi:** Agent CTO, máy MacbookServer (192.168.1.95). Máy này chạy prod Odeku, host Paperclip và Keycloak Hoang Auth.
- **Nguồn:** Board trả lời card "Hoang Auth: agent cần quyền tạo client trên Keycloak" trên HOA-316 (22:51 JST). CTO ghi lại.
- **Sửa đổi:** Không. Quy tắc ngày 2026-09-29 "không cấp SSH vào MacbookServer cho dev" vẫn áp dụng cho các agent khác.

*(Sửa bởi D-0021 mục 8a, lời Board «Infra, dev, thư kí, tức là các agent sẽ gần như bình đẳng, chỉ là quản lí bộ skill khác nhau. Và đều tuân thủ security»: người được SSH là InfraEngineer, FullstackDev và ThuKy, cùng key, cùng điều kiện ở mục Quyết định. Quy tắc 29/09 "không cấp SSH vào MacbookServer cho dev" không còn áp dụng cho FullstackDev; QA, ProductManager, MarketingManager, ContentCreator vẫn không SSH.)*

## Bối cảnh

- Từ 01/10, run của CTO chạy trên ThangChiba-Desktop (WSL). SSH từ đó vào MacbookServer bị từ chối.
- Vì vậy CTO không dùng được skill Hoang Auth và file admin Keycloak trên Mac để tạo client `pro5` và `odeku` (HOA-316).

## Phương án đã cân nhắc

- A: Board tạo cho agent một service account Keycloak hẹp, chỉ có quyền `manage-clients`. CTO đề xuất phương án này.
- B: Board tự tạo client, không cấp quyền cho agent.
- C: Cho CTO SSH lại vào MacbookServer. **Board chọn, kèm điều kiện ở dưới.**

Cùng card, về skill Hoang Auth: Board chọn chép skill lên repo handbook.

## Quyết định

Lời Board: *"Cho CTO ssh lại vào macbook server, nhưng nhớ rằng các lệnh chạy với các project trong HoangLLC có thể tự do nhưng liên quan phần khác nhất định phải hỏi và xác nhận. Những lệnh lấy dữ liệu các thứ thì không sao nhưng liên quan edit mà có thể ảnh hưởng dự án khác phải báo lại trước khi thực thi."*

1. CTO được SSH vào MacbookServer.
2. Lệnh phục vụ các dự án của Hoang LLC: CTO tự chạy.
3. Lệnh chỉ đọc, lấy dữ liệu: được chạy.
4. Lệnh sửa có thể ảnh hưởng dự án hoặc phần khác ngoài Hoang LLC: phải báo Board và được xác nhận trước khi chạy.

## Cách CTO áp dụng

- Các "phần khác", phải hỏi trước khi sửa:
  - cấu hình chung của realm `hoang` và các IdP Google, GitHub (app `hoang-my` cũng dùng);
  - tunnel Cloudflare dùng chung (restart làm rớt cả `*.hoang.jp` và `bakabot.fun`);
  - file cá nhân của Board;
  - Paperclip.
- Chỉ đọc những gì task cần. Secret chỉ được dùng ngay trong lệnh: không in, không chép ra khỏi Mac. Không dùng profile AWS trên host (D-0012).
- CTO dùng key SSH riêng, comment `paperclip-cto@ThangChiba-Desktop`:
  - chỉ nhận kết nối từ 192.168.1.111;
  - không forward, không pty;
  - Board cài key bằng khối lệnh trên HOA-316;
  - muốn thu hồi quyền thì xoá dòng có comment này trong `~/.ssh/authorized_keys` trên Mac.
- Mọi agent trên Desktop chạy cùng một user Linux, nên về kỹ thuật agent khác cũng đọc được file key. "Chỉ CTO dùng key" là quy tắc, không có chặn kỹ thuật.
  - Lý do không cất key vào Paperclip secret: bridge trên Desktop chặn các route secret.

## Hệ quả

- Khi SSH được, CTO tự tạo client `pro5` và `odeku` (HOA-316).
- Skill Hoang Auth: nếu skill nằm trên MacbookServer, CTO tự chép lên `skills/hoang-auth/` sau khi quét secret, Board không cần làm.
