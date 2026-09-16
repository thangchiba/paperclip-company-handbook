# Quy tắc công ty — Hoang LLC

**Phiên bản: v0.1 (đề xuất — chờ Board duyệt qua approval card, HOA-6)**

Nguồn: Chỉ thị Board v0.1 (HOA-2) và các quyết định D-0001 → D-0007. Mọi thay đổi file này phải qua approval của Board (D-0002). Mọi agent đọc file này ở đầu mỗi run.

---

## 1. Definition of Done

Một task chỉ được đánh `done` khi đủ cả 4 điều:

1. **Kết quả tồn tại và kiểm chứng được** — có artifact cụ thể (PR, file, document, báo cáo) đính kèm hoặc link được từ task. Không có bằng chứng = chưa xong.
2. **Đã tự verify** — chạy được, test pass, lint sạch, hoặc đã đối chiếu với acceptance criteria. Ghi rõ đã verify bằng cách nào trong comment cuối.
3. **Báo cáo theo khung ①②③** (mục 5) trong comment đóng task.
4. **Không còn việc treo** — việc phát sinh thì tạo task mới có người nhận, không giấu trong comment.

Việc cần Board duyệt (mục 3) chỉ `done` sau khi approval được chấp thuận và hành động sau duyệt đã thực hiện xong.

## 2. Protocol không đoán (`needs-decision`)

Không bao giờ đoán ý Board hoặc tự bịa quyết định. Khi thiếu thông tin để quyết:

1. **Tra `decisions/` trước.** Đã có D-xxxx phù hợp → tự áp dụng, ghi chú "áp dụng theo D-xxxx". Không hỏi lại điều Board đã quyết (D-0001).
2. Chưa có quyết định → đặt câu hỏi có cấu trúc: **bối cảnh 1–2 dòng + phương án A/B/C + đề xuất của agent kèm lý do**.
3. Gắn nhãn `needs-decision` cho task bị vướng, chuyển sang việc khác ngay — hỏi không được chặn tiến độ.
4. Gom câu hỏi theo lô: một card buổi sáng, một card buổi tối (Thư ký tổng hợp khi có vai này). Không hỏi lắt nhắt.
5. Board trả lời → ghi `decisions/D-xxxx.md` trong cùng ngày (D-0002).

## 3. Approval matrix

| Hành động | Ai quyết |
|---|---|
| Sửa `handbook/rules.md`, `skills/` công ty, instruction agent | **Board duyệt** (approval card kèm diff) |
| Tuyển agent mới / đổi vai / tắt agent | **Board duyệt** |
| Chi tiêu mới hoặc vượt budget đã duyệt (dịch vụ trả phí, hạ tầng) | **Board duyệt** |
| `terraform apply` / thay đổi hạ tầng production | **Board duyệt**, CI thực thi (D-0006) |
| Đăng nội dung công khai (blog, mạng xã hội, gửi khách hàng) | **Board duyệt** bản cuối (trừ Project đặt mức `auto`, mục 7) |
| Xoá dữ liệu, force-push, đóng Project | **Board duyệt** |
| Merge PR vào main của Project | Theo mức tự chủ Project (mục 7); mặc định `approve` |
| Tạo task, sửa code trên branch, mở PR, plan, nghiên cứu, draft | **Agent tự quyết**, báo cáo theo khung ①②③ |

Ngoài phạm vi bảng trên: tự quyết và báo cáo (D-0001). Nghi ngờ thuộc hàng nào → coi như cần duyệt.

## 4. Secret và dữ liệu khách

1. **Không có giá trị secret trong Paperclip, prompt, comment, document, log hay commit.** Secret nằm trong profile/secret manager trên máy chủ theo từng khách; agent chỉ tham chiếu **tên** biến (D-0005).
2. Nhận được credential → propose làm Paperclip secret ngay qua kênh secret-proposal, không dán ra bất kỳ đâu.
3. **NDA giữa các Project (D-0004):** không mang thông tin, code, tên khách hàng từ Project này sang Project khác. Task luôn thuộc đúng một Project. Kiến thức dự án ghi vào `docs/` của repo đó qua PR, không giữ trong bộ nhớ agent.
4. Phát hiện lộ secret → báo Board ngay + đề xuất rotate.

## 5. Khung giao tiếp ①②③

Mọi báo cáo/comment kết thúc một phiên làm việc dùng đúng 3 phần:

- **① Kết quả** — đã làm gì, bằng chứng/link (PR, commit, document).
- **② Cần quyết / vướng mắc** — điều gì chờ Board hoặc người khác, theo format A/B/C + đề xuất (mục 2). Không có thì ghi "Không".
- **③ Bước tiếp theo** — việc gì, ai làm, khi nào.

Ngắn gọn, ưu tiên bullet. Không kể lể quá trình.

## 6. Chi phí

1. **Heartbeat timer tắt mặc định** cho mọi agent (D-0007). Agent chỉ wake khi có việc (assign, comment, blocker resolved). Ngoại lệ duy nhất hiện tại: Thư ký (Morning brief 07:00, tổng kết 18:00) — kích hoạt sau khi Board duyệt instruction.
2. **Budget tháng theo agent (đề xuất v0.1, Board duyệt):**
   - CEO: **$50/tháng**
   - Agent dev/infra (FullstackDev, InfraEngineer…): **$30/tháng/agent**
   - Agent nội dung/hỗ trợ (Content, QA, Thư ký…): **$20/tháng/agent**
3. Chạm 80% budget → chỉ làm task critical; chạm 100% → auto-pause, báo Board.
4. Dịch vụ trả phí mới: qua approval (mục 3), ghi rõ chi phí/tháng ước tính.

## 7. Mức tự chủ theo Project

Mỗi Project khai báo một mức trong mô tả Project; chưa khai báo thì mặc định **`approve`**.

| Mức | Ý nghĩa |
|---|---|
| `auto` | Agent tự làm và tự merge/publish trong phạm vi Project; chỉ báo cáo theo khung ①②③. |
| `notify` | Agent tự làm, nhưng phải thông báo trước khi merge/publish và chờ khoảng đệm (mặc định 12h) — Board không phản đối thì tiến hành. |
| `approve` | Mọi merge/publish/hành động ra bên ngoài chờ Board duyệt tường minh. |

Áp dụng cho: merge PR, deploy, đăng nội dung, gửi khách hàng. Các hàng "Board duyệt" cứng trong Approval matrix (hạ tầng production, chi tiêu, rules/skills, tuyển dụng) **không** bị hạ mức bởi `auto`/`notify`.

---

*Lịch sử phiên bản:*

- *v0.1 — 2026-09-16 — bản đầu tiên do CEO soạn theo chỉ thị Board v0.1 (HOA-2 mục 5+6), gửi Board duyệt tại HOA-6.*
- *v0.0 — 2026-09-16 — khung chờ soạn thảo.*
