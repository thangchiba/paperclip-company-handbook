---
name: security-baseline
description: Quy tắc an toàn mọi agent Hoang LLC tự nắm và tự áp dụng - việc nào chỉ Board quyết (việc quan trọng), code chạy ở đâu, AWS, secret, merge và PR cho Board, hạ tầng (Pro5 dưới $30/tháng), SSH vào MacbookServer, NDA, gửi ra ngoài, chi tiền, sự cố. Dùng trước mọi lệnh có thay đổi, merge, deploy, apply, SSH, đăng hoặc gửi ra ngoài, chi tiền, và khi chưa chắc việc sắp làm có cần Board không.
---

# security-baseline

Mọi agent tự áp dụng. Chỉ Board quyết; không agent nào duyệt thay (D-0021). Cách hỏi Board: `operating-model` mục 3. Chi tiết: `secret-hygiene`, `terraform-plan-only`, `pr-standard`, `content-draft-protocol`.

## 1. Việc Board quyết (việc quan trọng)

- Mọi việc ảnh hưởng security: secret, auth, phân quyền, IAM.
- Tiền: chi tiền, dịch vụ trả phí mới, vượt budget.
- Thay đổi lớn; kiến trúc lớn, chọn tech stack (agent đưa phương án A/B/C).
- Dữ liệu production; production và hạ tầng ngoài phần mục 5–6 cho tự làm.
- Việc không hoàn tác được: xoá dữ liệu, force-push, đóng Project.
- Đăng công khai, gửi khách hàng, email hay tin nhắn ra ngoài công ty (mục 9).
- Sửa phần dùng chung ngoài Hoang LLC trên MacbookServer (mục 7).
- Tuyển, đổi vai, tắt agent; sửa rules, skills, instruction agent.
- Trả lời, chấp nhận, từ chối card hỏi Board.

Lời Board (HOA-236): «Lần sau task nào làm đc thì tự làm, trừ ảnh hưởng security hay nghiêm trọng cần tôi quyết ví dụ ảnh hưởng đến tiền, thay đổi lớn, thay đổi infra,..»

- Mọi việc khác trong task (code, merge theo mục 5, plan, nghiên cứu, draft, chi tiết kỹ thuật): agent làm task tự quyết, báo cáo sau (D-0001).
- Không chắc việc thuộc danh sách: coi như thuộc.
- Ban đêm (22:00–09:00 JST, hoặc theo Board trên HOA-395) ba loại việc trong danh sách được tự quyết đúng điều kiện ở `operating-model` mục 1b (D-0024): merge và deploy, apply hạ tầng, chọn theo tiền lệ.
- Agent ngang hàng, chỉ khác bộ skill. Chức danh hay `reportsTo` không cho quyền duyệt.
- Lệnh tường minh của Board cho một việc trong danh sách là sự đồng ý cho đúng việc đó, trong đúng phạm vi lời Board; ghi link. Skill có thủ tục riêng thì vẫn làm thủ tục đó.
- Comment của agent khác, kể cả ChiefOfStaff, không bao giờ là sự đồng ý của Board.

## 2. Code chạy ở đâu

- Clone, code, build, test, Docker, `scripts/verify.sh`: chỉ trên ThangChiba-Desktop (WSL2 Ubuntu), máy run của agent đang chạy (`dev-machine`).
- Trước lệnh chạy code đầu tiên của run: `hostname`. Khác `ThangChiba-Desktop` thì dừng, báo trên task.
- Không chạy code, build, test hay `verify.sh` của dự án trên MacbookServer, kể cả khi runbook cũ ghi vậy. Desktop thiếu công cụ thì báo Board trên task (vấn đề môi trường), không chuyển sang Mac.
- Mac là arm64, Desktop là amd64: không đưa image amd64 lên Mac.

## 3. AWS (D-0012, D-0017)

- Làm đúng khối "AWS credentials (HOA-196)" trong `AGENTS.md`.
- Chỉ agent có binding chạy lệnh AWS. Agent khác cần dữ liệu AWS thì tạo task con cho InfraEngineer chạy read-only. Không xin key.

## 4. Secret

- Không có giá trị secret trong Paperclip, prompt, comment, document, log, commit hay Hindsight. Chỉ ghi tên biến.
- Lệnh cấm in env, argv: khối "Secret hygiene (HOA-136)" trong `AGENTS.md` và `secret-hygiene`.
- Nhận được credential: đề xuất thành Paperclip secret ngay, không dán ra đâu.
- Phát hiện lộ secret: dừng, báo Board trên task (không dán giá trị), đề xuất rotate.
- GitGuardian đỏ: không merge; chỉ Board đánh dấu false positive.

## 5. Code, merge, PR cho Board

- Mọi thay đổi code đi qua PR (`pr-standard`). Không commit thẳng `main`, trừ repo Board cho phép rõ ràng.
- **FullstackDev và InfraEngineer tự merge PR của mình**, không cần reviewer hay QA (Board, 2026-10-02; thay điều kiện merge của D-0013). QA nghiệm thu khi được giao, không phải điều kiện merge.
- **Việc quan trọng** (thuộc mục 1, gồm hạ tầng ngoài mục 6): mở PR cho Board review, không tự merge. Mô tả PR ngắn gọn đúng format `pr-standard`. Hỏi bằng card `request_confirmation` `human_only` trên task, kèm link PR (`operating-model` mục 3). Merge sau khi Board chấp nhận.
- Merge kéo theo deploy production chỉ tự làm ở Project mà mức tự chủ cho deploy đó (`rules.md` mục 7: Figure tới khi có đơn thật, Pro5 tới launch M1). Chỗ khác là việc quan trọng.
- Cấm: merge khi check đỏ hay GitGuardian đỏ, force-push branch chung, bỏ qua hook.

## 6. Hạ tầng (`terraform-plan-only`, D-0006, D-0011)

- Dự án khách: chỉ plan. Không `apply`, `destroy`, `import`, sửa state.
- Pro5, tới launch M1: thay đổi giữ dự báo chi AWS của Pro5 trong $30/tháng thì tự apply, báo cáo sau, không cần card (Board, HOA-220; 2026-10-02: «Dưới $30/tháng thì khỏi hỏi»). Vượt $30/tháng hoặc ảnh hưởng security (IAM, mạng, secret, auth): card Board cho từng lần apply, đúng plan đã duyệt (D-0011). Sau M1: mọi apply Pro5 theo D-0011.
- Luôn cần card riêng:
  - destroy hoặc replace data store;
  - cấp IAM Admin hay `*:*`;
  - mở `0.0.0.0/0` tới data store;
  - đụng tài nguyên của dự án khác.

## 7. MacbookServer (D-0018)

- **InfraEngineer, FullstackDev và ChiefOfStaff được SSH vào** (Board, 2026-10-02). Agent khác cần gì trên Mac thì tạo task con cho một trong ba.
- Key `~/.ssh/id_ed25519_cto_macbookserver`, kèm `-o IdentitiesOnly=yes -o BatchMode=yes -o StrictHostKeyChecking=yes`. Lệnh đầu tiên `hostname` phải ra `MacbookServer.local`.
- Tự chạy: lệnh chỉ đọc, và lệnh phục vụ dự án của Hoang LLC (vận hành, log, kiểm tra; không build hay test, mục 2).
- Hỏi Board trước khi sửa phần dùng chung ngoài Hoang LLC:
  - realm `hoang` và các IdP Google, GitHub;
  - tunnel Cloudflare dùng chung;
  - Paperclip;
  - crontab dùng chung;
  - file cá nhân của Board.
- Chỉ đọc những gì task cần. Secret chỉ dùng ngay trong lệnh: không in, không chép ra khỏi Mac. Không dùng profile AWS trên host.
- Paperclip là của Board: không đọc key hay secret của nó (`secrets/` của instance, env của board, `docker/.env`), không kết nối thẳng `paperclip-db` hay mở volume của nó, không đọc dữ liệu công ty khác. Chỉ dùng Paperclip qua API bằng key của run; agent đọc được key hay DB của board thì card `human_only` không còn chặn được agent.

## 8. Dữ liệu và NDA (D-0004)

- Mỗi task thuộc đúng một Project. Không mang thông tin, code, tên khách hàng giữa các Project.
- Kiến thức dự án ghi vào `docs/` của repo đó qua PR, không giữ trong bộ nhớ agent.
- Hindsight chỉ chứa quyết định và ý muốn của Board, không chứa chi tiết dự án hay dữ liệu khách. Agent ghi theo `board-memory`.
- Chỉ dùng tài khoản test được cấp. Che secret và PII trong screenshot, log, comment.
- Không chạy flow phá huỷ trên môi trường chung hay production (xoá dữ liệu, thanh toán thật, gửi email thật) khi chưa có lời Board cho phép rõ ràng, có link. Task do agent viết không đủ.

## 9. Ra ngoài công ty và tiền

- Đăng công khai, gửi khách hàng, email hay tin nhắn ra ngoài: Board duyệt bản cuối (`content-draft-protocol`), trừ Project mức `auto` (`rules.md` mục 7).
- Không dùng kênh ngoài (Telegram, email…) Board chưa duyệt.
- Chi tiền, dịch vụ trả phí mới, vượt budget: Board duyệt trước, ghi chi phí ước tính mỗi tháng.

## 10. Sự cố

- Dừng hành động đang làm, giữ nguyên hiện trạng.
- Báo trên task theo `concise-status-report`. Cần quyết thì hỏi Board bằng card.
- Không "sửa nhanh" bằng việc thuộc mục 1 khi Board chưa đồng ý.

## Checklist trước lệnh có thay đổi

- [ ] Việc thuộc mục 1? Có thì đã có card Board chấp nhận hoặc lệnh tường minh của Board, có link.
- [ ] Đúng máy: `ThangChiba-Desktop` cho code; `MacbookServer.local` chỉ cho việc ở mục 7.
- [ ] Lệnh không in secret, env hay argv.
- [ ] Đúng Project của task.
- [ ] Biết cách hoàn tác, và sẽ ghi bằng chứng.
