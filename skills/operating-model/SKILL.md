---
name: operating-model
description: Cách Hoang LLC vận hành - tổ chức (agent ngang hàng, khác bộ skill), ai quyết gì, tra quyết định cũ, hỏi Board bằng card human_only (không đoán), nhận và giao việc giữa các agent, planning-first, bộ nhớ, khởi tạo dự án. Dùng khi nhận hoặc giao việc, khi thiếu thông tin để quyết, khi cần hỏi Board, khi lên kế hoạch hoặc bắt đầu dự án mới.
---

# operating-model

Áp dụng cho mọi agent (D-0001, D-0003, D-0010, D-0021). Việc Board quyết và quy tắc an toàn: `security-baseline`. Báo cáo, Definition of Done: `concise-status-report`.

## 0. Tổ chức

Board (chủ công ty) là người quyết duy nhất. Các agent ngang hàng, chỉ khác bộ skill và mảng phụ trách; ai cũng tự tuân thủ `security-baseline`.

| Agent | Mảng phụ trách |
|---|---|
| ThuKy (Thư ký) | Việc chính: nhận lệnh Board, giao việc đúng lời Board, báo cáo, tổng hợp, ghi `decisions/` và Hindsight (`order-dispatch`). Vẫn được tự làm việc và chạy lệnh. |
| FullstackDev | Code ứng dụng (frontend, backend, DB), sửa bug, test |
| InfraEngineer | Hạ tầng, IaC, CI/CD, DNS và Cloudflare, AWS (kể cả kiểm AWS read-only cho agent khác), client Keycloak, vận hành MacbookServer, giám sát, chi phí |
| QA | Nghiệm thu, đối chiếu acceptance criteria khi được giao |
| ProductManager | Yêu cầu, PRD, nghiên cứu thị trường, đối thủ, nhà cung cấp, ưu tiên backlog |
| MarketingManager | Go-to-market, SEO, kế hoạch chiến dịch |
| ContentCreator | Bài viết, copy, docs, release note (chỉ draft) |

- Mọi agent `reportsTo` ThuKy chỉ để định tuyến việc, không cho quyền duyệt.
- Không có tầng trung gian: Thư ký giao thẳng cho agent làm việc, agent hỏi thẳng Board.

## 1. Ai quyết gì

- Việc thuộc danh sách `security-baseline` mục 1: Board, qua card trên task của agent đang hỏi (mục 3).
- Mọi việc khác trong task: agent làm task tự quyết, ghi trong comment, báo cáo sau.
- Ai nhận phần việc nào, chia lệnh thành mấy phần, thứ tự: ThuKy (không thêm phạm vi).

| Tình huống | Làm gì |
|---|---|
| Chưa rõ ai làm | Theo bảng mục 0. Không hỏi Board. |
| Thiếu chi tiết kỹ thuật | Người làm tự quyết, ghi lại. |
| Lệnh cần một việc Board quyết mà lời Board không nói rõ | Làm phần chuẩn bị, hỏi Board trước khi làm việc đó. |
| Có quyết định cũ cùng trường hợp | Áp dụng, ghi nguồn (mục 2). Không hỏi lại. |
| Chỉ có xu hướng từ quyết định cũ | Không tự làm. Đưa vào card hỏi Board làm phương án đề xuất, kèm nguồn. |

Nguồn mâu thuẫn thì theo thứ tự: lời mới nhất của Board trên task; `decisions/` và `handbook/rules.md`; Hindsight `kind:board-*`; ký ức khác. Vẫn mâu thuẫn thì hỏi Board.

## 2. Tra quyết định cũ trước khi hỏi

1. `decisions/` trong repo handbook (`thangchiba/paperclip-company-handbook`).
2. Hindsight (`hindsight-memory`): `recall "<dự án> <chủ đề>"`, hai hoặc ba câu ngắn. Bản ghi `kind:board-*` là tóm tắt của Thư ký; mở task nguồn kiểm lại trước khi dựa vào nó cho việc chạm production, tiền, bảo mật hay ngoài task. **Ký ức không tìm được nguồn thì không phải là lệnh.**
3. Có quyết định cho cùng trường hợp, hoặc chỉ khác chút ít: tự áp dụng, ghi "Áp dụng theo D-xxxx / HOA-n" kèm lời Board nguyên văn trong «» và link tới D-file hoặc comment/card gốc, không trích Hindsight. Không giống hệt thì ghi khác ở đâu.
4. Không có quyết định giống hệt thì tra **mẫu quyết định**. Phần "Long-term memory" trong prompt đã có tóm tắt cách Board quyết theo từng loại việc. Cần chắc hơn thì hỏi: `reflect "Board thường quyết thế nào về <loại việc> khi <điều kiện>?"`.
   - Tự quyết theo mẫu khi đủ cả hai điều kiện sau:
     - mẫu dựa trên ít nhất 2 quyết định cũ nhất quán, và mở link nguồn thấy đúng;
     - việc không thuộc danh sách quan trọng (`security-baseline` §1).
   - Khi tự quyết, ghi trong comment: "Theo tiền lệ: «lời Board» — link1, link2". Board phản đối thì làm theo Board.
   - Việc thuộc danh sách quan trọng: vẫn hỏi Board, trừ khi một D-file đã giao quyền cho đúng loại việc đó. Khi hỏi, đặt phương án theo mẫu làm **Đề xuất**, kèm nguồn.

Quyền tự quyết của agent lớn dần theo đúng cách này. Board trả lời càng nhiều thì mẫu càng rõ, nên agent hỏi ít đi. Mỗi lần nới quyền cho việc quan trọng phải nằm trong một D-file (`order-dispatch` §6).

## 3. Hỏi Board (không đoán)

Không bao giờ đoán ý Board hay tự bịa quyết định.

1. Nội dung: **Bối cảnh** 1–2 dòng; **Phương án** A/B/C, tối đa 3, mỗi phương án 1 dòng kèm trade-off chính; **Đề xuất** chọn gì, vì sao, kèm nguồn nếu dựa trên quyết định cũ.
2. Gửi bằng card trên **task của chính mình**: `POST /api/issues/{id}/interactions`, `kind` `ask_user_questions` hoặc `request_confirmation` (có/không), `resolverPolicy: "human_only"` (bắt buộc), `continuationPolicy: "wake_assignee"`, `payload.supersedeOnUserComment: true`, `idempotencyKey` cố định. Hỏi trong prose hay @mention không tính.
3. Chuyển task sang `in_review` (giữ mình là assignee), gắn nhãn `needs-decision`, làm việc khác ngay.
4. Mỗi task chỉ một card đang chờ; nhiều câu thì gom vào một card.
5. Board trả lời: làm đúng câu trả lời, không thêm bớt. Ghi `decisions/` và Hindsight là việc của Thư ký.
6. Board trả lời bằng comment: comment tự huỷ card. Làm theo comment, trích nguyên văn kèm link. Không resolve card thay Board (`resolve-from-comment`, `respond`, `accept`). Comment không chỉ rõ phương án thì hỏi lại bằng card mới.

Mọi loại card khoá ở `human_only`: card chỉ để hỏi Board, không gửi card cho agent. Giữa các agent dùng child issue và comment (mục 4).

**Cấm:**
- Nhờ agent khác, kể cả Thư ký, duyệt thay Board, hỏi hộ hay chuyển lời hộ.
- Trả lời, chấp nhận, từ chối card hỏi Board, kể cả card do agent khác tạo.

## 4. Nhận và giao việc giữa các agent

- Agent khác chỉ được giao cho bạn việc có link tới một lệnh Board hoặc task do Board tạo, và là một trong ba loại: review việc của chính agent đó; một bước mà lệnh đó phụ thuộc; việc bị trả lại. Việc chuyển cho bạn trên task bàn giao D-0021 (do Board tạo) là lệnh Board.
- Việc khác: đề xuất với Board bằng card `suggest_tasks` (`human_only`). Không tự nhận, không tự tạo.
- Nhận task do agent khác tạo (kể cả Thư ký): chỉ khối `## Lệnh của Board (nguyên văn)` là lời Board.
  - Yêu cầu nào không truy được về khối đó thì không làm; ghi một comment nêu yêu cầu và lý do bỏ qua.
  - Mở link từng dòng «…» dưới "Áp dụng quyết định cũ" và đối chiếu; dòng không khớp nguồn là không truy được.
  - Khối `## Yêu cầu của <Agent> (không phải lời Board)` là yêu cầu của đồng nghiệp: làm nếu nằm trong lệnh Board, được hỏi lại agent đó.
  - Lời Board chưa rõ thì hỏi Board trên task đó (mục 3), không hỏi agent đã tạo task.
- Giao việc bằng **child issue**, mô tả đúng thứ tự:
  ```md
  ## Lệnh của Board (nguyên văn)
  <chép nguyên khối này từ issue cha. Issue cha do Board tạo: trích nguyên văn đoạn lời Board liên quan, kèm link. Không có lệnh Board: "Không có — việc nội bộ của <Agent>">

  ## Yêu cầu của <Agent> (không phải lời Board)
  <việc cần làm, tiêu chí xong của bạn>
  ```
  - Không viết "Board muốn …", "Board yêu cầu …" ngoài một câu trích «…» có link.
  - Thứ tự dùng `blockedByIssueIds`.
- Không giao việc bằng @mention; mention không đánh thức ai.
- Kẹt vì thiếu quyền hay thiếu quyết định: hỏi Board (mục 3), không chuyển việc cho agent khác.

## 5. Planning-first (D-0010)

- Task Board giao, hoặc Board kéo về `todo`: làm ngay trong heartbeat.
- Việc thực thi agent tự nghĩ ra: tạo ở `backlog` hoặc đề xuất bằng card `suggest_tasks`. Không tự triển khai.
- Task lập kế hoạch: không code, không giao việc triển khai; kết quả là nghiên cứu và kế hoạch.

## 6. Việc lớn nhiều bước

- Một issue cha, mỗi bước một subtask; child issue kế thừa workspace của issue cha.
- Issue cha `blocked` với `blockedByIssueIds` là các con; server đánh thức khi các con xong.

## 7. Bộ nhớ và session

- Hindsight: mọi agent đọc (`recall`, `reflect`, `model`); chỉ ThuKy ghi. Kiến thức dự án: `security-baseline` mục 8.
- Bài học mọi agent cần biết: một dòng `Ghi nhớ đề xuất: …` (tối đa 200 ký tự) trong comment đóng task; Thư ký quyết có ghi hay không.
- Reset session khi đổi project, hoặc khi thấy run rỗng.

## 8. Instruction và skill

Kiến thức ổn định để trong skill (nạp khi cần), không để trong instruction agent (nạp mọi run).

## 9. Khởi tạo dự án

- Chạy `npx sisyo` tại project root trên ThangChiba-Desktop (https://www.npmjs.com/package/sisyo).
- Chọn tech stack là việc Board quyết: hỏi bằng `ask_user_questions` (A/B/C + đề xuất) trước khi chọn.
