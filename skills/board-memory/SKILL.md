---
name: board-memory
description: Ghi lại lệnh và quyết định của Board cho mọi agent - Hindsight (khi nào ghi, không bao giờ ghi gì, prompt tóm tắt cố định, chống trùng) và decisions/. Dùng khi Board giao lệnh hay quyết định trực tiếp cho bạn, khi Board bảo bạn ghi nhớ một điều, trước mỗi lần ghi Hindsight hay viết D-file.
---

# board-memory

Mọi agent, D-0021 và D-0022. Lệnh gọi Hindsight ở skill `hindsight-memory`; đây là chính sách ghi.

## 1. Ai ghi

Mọi agent ngang quyền ghi (D-0022). Bạn ghi khi:
- Board giao lệnh hay quyết định **trực tiếp cho bạn**: task Board tạo cho bạn, comment của Board trên task của bạn, câu trả lời của Board trên card bạn hỏi, chat; hoặc
- Board bảo bạn ghi nhớ một điều.

Một lệnh chỉ một người ghi: người nhận lệnh trực tiếp từ Board. Lệnh đi qua Thư ký thì Thư ký ghi; agent nhận child issue từ Thư ký không ghi lại.

Không ghi điều bạn tự nghĩ ra. Bài học bạn muốn mọi agent biết: một dòng `Ghi nhớ đề xuất: …` (tối đa 200 ký tự) trong comment đóng task; chỉ ghi khi Board bảo ghi.

## 2. Ghi gì

**Ghi khi:**
- lệnh có ý muốn, ưu tiên, ràng buộc hay quy tắc dùng lại được → `kind:board-order`, tối đa 1 bản ghi mỗi lệnh;
- Board chọn phương án, đặt quy tắc, hoặc sửa cách agent làm việc → `kind:board-decision`, tối đa 1 bản ghi mỗi quyết định;
- Board bảo ghi một bài học → `kind:lesson`.

Lệnh thuần một lần thì không ghi.

**Không bao giờ ghi:** tiến độ, trạng thái; chi tiết kỹ thuật dự án; tên hay dữ liệu khách hàng; secret; suy luận của agent; điều Board không nói.

Plugin tự lưu câu trả lời của Board trên mọi card (`source:operator-answer`). Không chép lại; chỉ ghi khi cần một bản tóm tắt gọn cho quyết định quan trọng.

## 3. Cách ghi

1. Tóm tắt bằng prompt cố định ở mục 5. Adapter cho phép thì chạy trong subagent riêng chỉ nhận INPUT; không thì tự áp dụng đúng prompt đó. INPUT là lời Board (lệnh, comment, câu trả lời card).
2. Kết quả `SKIP`: không ghi.
3. Đối chiếu với INPUT: mỗi số, tên, ngày phải có trong INPUT; phần trong «» là chữ của INPUT, chỉ được lược bằng …. Bỏ chữ nào không có trong INPUT.
4. Chống trùng: `recall "<project> <chủ đề>" --tags kind:board-decision,kind:board-order`. Cùng ý thì ghi đè đúng doc-id cũ. Board đổi ý thì ghi bản mới có `Thay: <doc-id cũ>`, rồi `forget --doc-id <doc-id cũ>`.
5. Tối đa 300 ký tự. Dài hơn thì rút gọn, không tách một quyết định thành hai bản ghi.
6. Ghi: `node <hindsight-memory>/scripts/hindsight.mjs remember "<bản ghi>" --doc-id <HOA-n|D-xxxx>-<slug> --tags <kind>,issue:HOA-n,project:<slug>` (`project:cong-ty` nếu áp dụng toàn công ty).
7. Chép toàn văn bản ghi vào comment trên task, dòng `Hindsight: đã ghi <doc-id>: «<toàn văn>»`, để Board sửa ngay.

## 4. decisions/

- Quyết định của Board áp dụng rộng hơn một task: người ghi theo mục 1 viết `decisions/D-xxxx-<slug>.md` trong ngày (D-0002), trích nguyên văn lời Board, PR vào repo handbook (`~/workspace/paperclip-company-handbook`, `git pull` trước; số D tiếp theo lấy sau khi pull). Repo public: không IP, secret, tên khách. Quyết định trong một task chỉ ghi Hindsight.
- PR chỉ thêm hoặc sửa file trong `decisions/`: branch `chore/<slug>`, tự merge sau khi quét `secret-hygiene`. PR chạm `rules.md` hay `skills/`: chờ Board (`security-baseline` mục 1).
- Suy luận của agent ghi "<Agent> suy ra", không bao giờ thành quyết định của Board.

## 5. Prompt tóm tắt (cố định, không sửa)

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
- Bỏ dòng thứ 3 nếu không có trường nào.
- Nếu INPUT không chứa gì dùng lại được về sau (ý muốn, ưu tiên, ràng buộc, quy tắc, lựa chọn giữa các phương án, sửa cách làm), chỉ trả về đúng một từ: SKIP
NGÀY: <YYYY-MM-DD>   PROJECT: <tên>   NGUỒN: <HOA-n hoặc D-xxxx>
INPUT:
<<<
<dán nguyên văn>
>>>
```
