# D-0022 — Agent ngang quyền ghi Hindsight; Board giao việc thẳng cho agent bất kỳ; Thư ký lấy lại tên ChiefOfStaff

- **Ngày:** 2026-10-02
- **Phạm vi:** Toàn công ty Hoang LLC (mọi agent).
- **Nguồn:** Chỉ thị Board ngày 2026-10-02 trong phiên Claude Code của Board.
- **Sửa đổi:** D-0021 mục 2, 3, 6; D-0002.

## Lời Board

> "Tôi nghĩ nên để tất cả các agent có quyền bình đẳng, nên con agent nào cũng có thể viết vào hindsight được nếu như là chỉ định của tôi. Khi có task có thể tôi sẽ chỉ định trực tiếp QA hoặc Dev làm việc chứ không phải lúc nào cũng qua thư kí. tuy nhiên tôi có lẽ sẽ nói chuyện và nhờ thư kí tổng hợp nhiều nhất. Còn nữa là tên ThuKy hơi lạc quẻ. có thể giữ lại tên thư kí bằng tiếng anh như cũ thì hay hơn nhỉ."

## Quyết định

1. **Mọi agent ngang quyền ghi Hindsight** khi là chỉ định của Board: lệnh hay quyết định Board giao trực tiếp cho agent đó (task, comment, câu trả lời card, chat), hoặc điều Board bảo agent ghi. Một lệnh một người ghi: người nhận lệnh trực tiếp từ Board. Agent không tự ghi điều mình nghĩ ra; dòng `Ghi nhớ đề xuất:` chỉ được ghi khi Board bảo ghi. Chính sách ghi và prompt tóm tắt chuyển từ `order-dispatch` sang skill mới `board-memory`, gắn cho mọi agent. Người ghi Hindsight một quyết định áp dụng rộng hơn một task thì cũng viết D-file (D-0002).
2. **Board giao việc thẳng cho agent bất kỳ**, ví dụ QA hay FullstackDev, không phải qua Thư ký. Agent nhận làm như lệnh Board: tự hỏi Board, tự ghi lại, báo cáo trên task đó. Thư ký không giao lại hay thêm yêu cầu vào việc đó.
3. **Thư ký** là nơi Board trò chuyện và nhờ tổng hợp nhiều nhất: nhận lệnh Board giao cho Thư ký, giao việc, báo cáo, tổng hợp tình hình cả công ty (kể cả việc Board giao thẳng cho agent khác), đề xuất giao quyền khi thấy 3 quyết định giống nhau (`order-dispatch` §6).
4. **Đổi tên agent ThuKy thành ChiefOfStaff**, chức danh "Thư ký / Chief of Staff", tên tiếng Anh như trước D-0021. Agent ChiefOfStaff cũ đã terminate. Role, quyền và giới hạn theo D-0021 mục 2 giữ nguyên.
5. Nhãn bộ nhớ trong prompt `[Thư ký tóm tắt]` đổi thành `[Agent tóm tắt]`.

## Hệ quả

- Skill: thêm `board-memory`; sửa `operating-model`, `order-dispatch`, `security-baseline`, `skills/README.md` v0.3.
- `AGENTS.md` của mọi agent: khối Decision authority ghi quyền ghi Hindsight mới; tên ChiefOfStaff.
- Quy tắc Hindsight nhắc "chỉ ThuKy ghi" được sửa theo mục 1.
