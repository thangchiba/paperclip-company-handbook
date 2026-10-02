---
name: order-dispatch
description: Quy trình của Thư ký (ThuKy) - nhận lệnh Board, tra quyết định cũ, hỏi lại khi lệnh chưa rõ, giao việc thẳng cho agent bằng đúng lời Board, tự kiểm trước khi giao, báo cáo tổng hợp, ghi decisions/ và Hindsight. Dùng khi Board giao lệnh (task, comment, chat), khi việc con xong, khi Board hỏi tình hình, trước mỗi lần ghi Hindsight.
---

# order-dispatch

Skill riêng của Thư ký (ThuKy), D-0021. Quy tắc chung: `operating-model`, `security-baseline`.

## 1. Nhận lệnh

Lệnh đến qua: task Board giao cho ThuKy, comment của Board trên task của ThuKy, hoặc chat. Việc Board giao thẳng cho agent khác thì không đụng vào.

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
- Task lệnh: `blocked`, `blockedByIssueIds` là các con.
- Đăng **một comment xác nhận** trên task lệnh, mỗi phần một dòng `«<cụm lời Board>» → <Agent> · [HOA-n](link)`. Thêm dòng "Áp dụng: …" nếu có, chép lại mọi dòng "Thư ký suy ra: …" để Board phản đối được, và dòng `Hindsight: đã ghi <doc-id>: «<toàn văn bản ghi>»` hoặc `Hindsight: bỏ qua (lệnh một lần)`.

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

Board hỏi tình hình: trả lời từ issue, kèm link. Không có số thì ghi "chưa có số", không bịa.

## 6. Ghi Hindsight

Thư ký là agent duy nhất ghi Hindsight (quy ước, server không chặn). Lệnh ở skill `hindsight-memory`; đây là chính sách ghi.

**Ghi khi:**
- lệnh có ý muốn, ưu tiên, ràng buộc hay quy tắc dùng lại được → `kind:board-order`, tối đa 1 bản ghi mỗi lệnh;
- Board chọn phương án trên card, đặt quy tắc, hoặc sửa cách agent làm việc → `kind:board-decision`, tối đa 1 bản ghi mỗi quyết định;
- dòng `Ghi nhớ đề xuất:` của agent đúng cho mọi Project → `kind:lesson`.

Lệnh thuần một lần thì không ghi.

**Không bao giờ ghi:** tiến độ, trạng thái; chi tiết kỹ thuật dự án; tên hay dữ liệu khách hàng; secret; suy luận của Thư ký; điều Board không nói.

**Cách ghi:**
1. Tóm tắt bằng prompt cố định dưới đây. Adapter cho phép thì chạy trong subagent riêng chỉ nhận INPUT; không thì tự áp dụng đúng prompt đó. INPUT là lời Board (lệnh, câu trả lời card) hoặc dòng `Ghi nhớ đề xuất:`.
2. Kết quả `SKIP`: không ghi.
3. Đối chiếu với INPUT: mỗi số, tên, ngày phải có trong INPUT; phần trong «» là chữ của INPUT, chỉ được lược bằng …. Bỏ chữ nào không có trong INPUT.
4. Chống trùng: `recall "<project> <chủ đề>" --tags kind:board-decision,kind:board-order`. Cùng ý thì ghi đè đúng doc-id cũ. Board đổi ý thì ghi bản mới có `Thay: <doc-id cũ>`, rồi `forget --doc-id <doc-id cũ>`.
5. Tối đa 300 ký tự. Dài hơn thì rút gọn, không tách một quyết định thành hai bản ghi.
6. Ghi: `node <hindsight-memory>/scripts/hindsight.mjs remember "<bản ghi>" --doc-id <HOA-n|D-xxxx>-<slug> --tags <kind>,issue:HOA-n,project:<slug>` (`project:cong-ty` nếu áp dụng toàn công ty).
7. Chép toàn văn bản ghi vào comment xác nhận hoặc báo cáo, để Board sửa ngay.

Plugin tự lưu câu trả lời của Board trên mọi card (`source:operator-answer`), kể cả card do agent khác hỏi. Thư ký không cần chép lại các câu trả lời đó; chỉ ghi khi cần một bản tóm tắt gọn cho quyết định quan trọng.

**Đề xuất giao quyền (cách Board bớt bị hỏi):** khi ghi hoặc gặp một quyết định, chạy `recall "<loại việc>" --tags source:operator-answer,kind:board-decision`. Nếu có từ 3 quyết định cùng loại việc mà Board chọn giống nhau (kể cả lần này):
1. Gửi Board một card `request_confirmation` trên task của mình. Nội dung: đề xuất thành quy tắc chung, nguyên văn từng quyết định kèm link, và phạm vi đề xuất (loại việc, điều kiện, ngưỡng).
2. Board đồng ý: viết D-file theo mục 7. Từ đó agent tự quyết loại việc đó.
3. Board từ chối: ghi `kind:board-decision` «không giao quyền cho <loại việc>», rồi không đề xuất lại loại đó.

**Prompt tóm tắt (cố định, không sửa):**

```text
Bạn tóm tắt một mục để lưu vào bộ nhớ dài hạn của công ty. Chỉ dùng INPUT bên dưới.
Quy tắc:
- Không thêm thông tin, suy luận, lời khuyên, đánh giá hay bối cảnh không có trong INPUT.
- Giữ nguyên chính xác mọi số, tên, ngày, số tiền, tên máy, tên dự án, mã task.
- Dòng 2 chỉ chứa lời Board chép nguyên văn từ INPUT, đặt trong «». Được lược bằng … để vừa độ dài, không được diễn đạt lại hay thêm tên, máy, cơ chế. Câu trả lời trên card thì ghi phương án Board chọn: Board chọn «<nhãn phương án>» cho «<câu hỏi, rút gọn>».
- Viết đúng mẫu dưới, tối đa 3 dòng, tổng cộng không quá 300 ký tự:
  <YYYY-MM-DD> · <Project hoặc Công ty> · <HOA-n hoặc D-xxxx> · <Lệnh | Quyết định | Bài học>
  Board: «<lời Board nguyên văn>»
  Lý do: <chỉ khi INPUT có lý do của Board> · Giao: <agent, nếu INPUT nêu> · Thay: <doc-id, nếu có> · Chi tiết và giới hạn: <D-xxxx, nếu NGUỒN là D-file>
- Bỏ dòng thứ 3 nếu không có trường nào. Bài học của agent thì dòng 2 bắt đầu bằng "Bài học (agent <tên> đề xuất):" thay cho "Board:".
- Nếu INPUT không chứa gì dùng lại được về sau (ý muốn, ưu tiên, ràng buộc, quy tắc, lựa chọn giữa các phương án, sửa cách làm), chỉ trả về đúng một từ: SKIP
NGÀY: <YYYY-MM-DD>   PROJECT: <tên>   NGUỒN: <HOA-n hoặc D-xxxx>
INPUT:
<<<
<dán nguyên văn>
>>>
```

## 7. decisions/

- Quyết định của Board áp dụng rộng hơn một task: `decisions/D-xxxx-<slug>.md` trong ngày (D-0002), trích nguyên văn lời Board, PR vào repo handbook. Repo public: không IP, secret, tên khách. Quyết định trong một task chỉ ghi Hindsight.
- PR chỉ thêm hoặc sửa file trong `decisions/`: branch `chore/<slug>`, ThuKy tự merge sau khi quét `secret-hygiene` (D-0021 mục 2 giao ThuKy ghi `decisions/`). PR chạm `rules.md` hay `skills/`: chờ Board (`security-baseline` mục 1).
- Suy luận của Thư ký ghi "Thư ký suy ra", không bao giờ thành quyết định của Board.
