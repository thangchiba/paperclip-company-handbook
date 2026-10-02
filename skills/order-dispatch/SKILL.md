---
name: order-dispatch
description: Quy trình của Thư ký (ChiefOfStaff) - nhận lệnh Board giao cho Thư ký, tra quyết định cũ, hỏi lại khi lệnh chưa rõ, giao việc thẳng cho agent bằng đúng lời Board, tự kiểm trước khi giao, báo cáo và tổng hợp tình hình cả công ty, đề xuất giao quyền. Dùng khi Board giao lệnh hay trò chuyện với Thư ký (task, comment, chat), khi việc con xong, khi Board hỏi tình hình hay nhờ tổng hợp.
---

# order-dispatch

Skill riêng của Thư ký (ChiefOfStaff), D-0021, D-0022. Quy tắc chung: `operating-model`, `security-baseline`. Ghi Hindsight và `decisions/`: `board-memory`.

Board trò chuyện và nhờ Thư ký tổng hợp nhiều nhất, nhưng Thư ký không phải cửa bắt buộc: Board giao thẳng việc cho agent bất kỳ (D-0022).

## 1. Nhận lệnh

Lệnh đến qua: task Board giao cho ChiefOfStaff, comment của Board trên task của ChiefOfStaff, hoặc chat. Việc Board giao thẳng cho agent khác là việc của agent đó: không giao lại, không thêm yêu cầu; chỉ đọc khi tổng hợp (mục 5).

Lệnh qua chat:
- Document `plan` chỉ chứa nguyên văn tin nhắn và câu trả lời của Board.
- Bàn giao: một issue lệnh giao cho chính mình, không `parentId`; `description` và `initialPlan` là nguyên văn tin nhắn Board kèm link chat. Rồi giao việc theo mục 4.

## 2. Tra cứu

1. Đọc hết lệnh và mọi comment của Board trên task.
2. Tìm task đang mở trùng việc: `GET /api/companies/{companyId}/issues?q=<từ khoá>`. Không tạo trùng (HOA-152).
3. Tra `decisions/` và Hindsight theo `operating-model` mục 2 (repo clone tại `~/workspace/paperclip-company-handbook`, `git pull` trước).

## 3. Hỏi lại Board

Chỉ hỏi khi lệnh còn để ngỏ:
- thuộc Project nào;
- kết quả thế nào là xong, khi việc không tự hiểu được;
- lời Board hiểu được theo hai cách;
- lệnh mâu thuẫn với một quyết định hay quy tắc;
- lệnh cần một việc Board quyết (`security-baseline` mục 1) mà lời Board chưa nói rõ;
- không agent nào có bộ skill phù hợp (tuyển agent chỉ khi Board ra lệnh).

Cách hỏi: một card theo `operating-model` mục 3, tối đa 3 câu; đề xuất suy từ quyết định cũ ghi "Thư ký suy ra từ …" kèm nguồn.

Board trả lời qua chat: rút card bằng `POST /api/issues/{id}/interactions/{interactionId}/withdraw`, `reason` "Board đã trả lời tại <link>".

## 4. Giao việc

Mỗi agent, mỗi phần việc: một child issue (`parentId` = task lệnh) cho agent có bộ skill phụ trách; tiêu đề lấy từ lời Board, chỉ được rút gọn. Thư ký tự làm phần thuộc việc của mình (đọc, kiểm, tổng hợp) hoặc phần Board giao đích danh Thư ký.

Mô tả theo đúng thứ tự:

```md
## Lệnh của Board (nguyên văn)
> <đoạn lời Board mà phần việc này xuất phát, chép nguyên văn>
Nguồn: <link task hoặc comment>

## Việc giao
<1–2 câu, chỉ nêu điều trích dẫn trên yêu cầu>

## Tiêu chí xong
<chỉ những gì Board nói; Board không nói thì ghi đúng câu:
"Board không nêu — agent tự đặt tiêu chí và ghi trong comment đầu tiên.">

## Đã làm rõ
<nguyên văn câu trả lời của Board trên card hoặc comment, trong «», kèm link; hoặc "Không">

## Áp dụng quyết định cũ
<mỗi dòng: «<lời Board nguyên văn>» — <link tới D-file, hoặc comment/card gốc của Board>
 trường hợp không giống hệt thì thêm dòng: Thư ký suy ra: áp dụng vì <điểm giống>; khác ở <điểm khác>
 hoặc "Không">
```

- Phần việc phụ thuộc nhau: xếp thứ tự bằng `blockedByIssueIds`.
- Effort từng child issue theo `operating-model` mục 4: lời Board nêu mức ("effort high", "cái này low thôi") thì đặt đúng mức đó; không nêu thì Thư ký tự chọn theo bảng. Comment xác nhận ghi mức effort ở cuối mỗi dòng giao việc.
- Task lệnh: `blocked`, `blockedByIssueIds` là các con.
- Đăng **một comment xác nhận** trên task lệnh, mỗi phần một dòng `«<cụm lời Board>» → <Agent> · [HOA-n](link) · effort <mức>`. Thêm dòng "Áp dụng: …" nếu có, chép lại mọi dòng "Thư ký suy ra: …" để Board phản đối được, và dòng `Hindsight: đã ghi <doc-id>: «<toàn văn bản ghi>»` hoặc `Hindsight: bỏ qua (lệnh một lần)`.

**Tự kiểm từng child issue trước khi tạo (bắt buộc):**
- [ ] Mỗi yêu cầu trong "Việc giao" và "Tiêu chí xong" truy được về một cụm chữ trong phần trích dẫn.
- [ ] Mỗi dòng "Áp dụng quyết định cũ" chép từ D-file hoặc comment/card gốc (không từ Hindsight), có link đã mở đối chiếu; không giống hệt thì có dòng "Thư ký suy ra".
- [ ] Số, tên, ngày, tiền, tên máy, tên dự án giống hệt lời Board.
- [ ] Không có từ mở rộng phạm vi mà Board không nói ("toàn bộ", "và các phần liên quan", "đồng thời", "tối ưu luôn", "nâng cấp", "chuẩn hoá").
- [ ] Không biến tính từ của Board thành con số hay yêu cầu cụ thể: "nhanh hơn" không thành "< 200 ms", "gọn" không thành "tối đa 5 dòng".
- [ ] Không có bước, công cụ, hạn chót, phạm vi hay task nào Board không yêu cầu. Không làm yêu cầu mạnh hơn hay yếu hơn.

Ô nào không đạt: sửa, hoặc hỏi Board (mục 3). Không giao.

## 5. Báo cáo

Khi được đánh thức vì các con đã xong (`issue_blockers_resolved`, `issue_children_completed`):
1. Đọc comment cuối của từng con, kiểm link bằng chứng có thật.
2. Đăng **một** báo cáo trên task lệnh theo `concise-status-report`: mỗi con một link; mọi card chờ Board, kèm link, ở mục **Cần chỉ thị**.
3. Task lệnh `done`, hoặc `in_review` khi Board phải kiểm gì đó.

Board hỏi tình hình hay nhờ tổng hợp: trả lời từ issue, kèm link, gồm cả việc Board giao thẳng cho agent khác. Không có số thì ghi "chưa có số", không bịa.

## 6. Ghi Hindsight và đề xuất giao quyền

Ghi lệnh và quyết định Board giao cho Thư ký theo `board-memory` (mọi agent ghi phần Board giao trực tiếp cho mình). Comment xác nhận ở mục 4 có dòng `Hindsight: đã ghi <doc-id>: «<toàn văn bản ghi>»` hoặc `Hindsight: bỏ qua (lệnh một lần)`.

**Đề xuất giao quyền (cách Board bớt bị hỏi), việc tổng hợp của Thư ký:** khi ghi hoặc gặp một quyết định, chạy `recall "<loại việc>" --tags source:operator-answer,kind:board-decision`. Nếu có từ 3 quyết định cùng loại việc mà Board chọn giống nhau (kể cả lần này):
1. Gửi Board một card `request_confirmation` trên task của mình. Nội dung: đề xuất thành quy tắc chung, nguyên văn từng quyết định kèm link, và phạm vi đề xuất (loại việc, điều kiện, ngưỡng).
2. Board đồng ý: viết D-file theo `board-memory` mục 4. Từ đó agent tự quyết loại việc đó.
3. Board từ chối: ghi `kind:board-decision` «không giao quyền cho <loại việc>», rồi không đề xuất lại loại đó.

## 7. decisions/

Theo `board-memory` mục 4. Suy luận của Thư ký ghi "Thư ký suy ra", không bao giờ thành quyết định của Board.
