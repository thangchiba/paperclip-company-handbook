# D-0024 — Chế độ tự quyết ban đêm: merge và deploy, apply hạ tầng, chọn theo tiền lệ

- **Ngày:** 2026-10-03
- **Phạm vi:** Toàn công ty Hoang LLC (mọi agent).
- **Nguồn:** Chỉ thị và câu trả lời của Board ngày 2026-10-03 trong phiên Claude Code của Board. Công tắc chế độ: [HOA-395](/HOA/issues/HOA-395).
- **Sửa đổi:** `security-baseline` mục 1 có ngoại lệ ban đêm; `operating-model` mục 1b; `order-dispatch` mục 5b.

## Bối cảnh

14 ngày tới 2026-10-03: agent tạo 115 card trong khung 23:00–08:00, mỗi card chờ trung bình 10 giờ (ban ngày 2,9 giờ). Nhiều nhất là xin merge PR hay deploy (Board đồng ý 39/44) và xin apply hạ tầng (đồng ý 22/24). Các lần Board từ chối là vì chi phí (WAF) hoặc vì agent quên quyết định cũ.

## Lời Board

> "Tôi chỉ muốn là những task buổi đêm tôi ngủ thì AI có năng lực quyết định thay tôi, ít đặt câu hỏi dần đi ấy."

Trả lời câu hỏi áp dụng:
- Khung giờ: "Tuỳ từng lúc tôi yêu cầu ấy. Nhưng thường thì là 22:00-9:00 thì sẽ tự quyết nhiều hơn"
- Loại việc được tự quyết ban đêm: chọn "Merge PR + deploy", "Apply hạ tầng", "Chọn phương án theo tiền lệ"; không chọn "Gửi/đăng ra ngoài".

## Quyết định

1. Chế độ tự quyết bật 22:00–09:00 JST mỗi ngày. Board bật hay tắt lúc khác bằng comment trên HOA-395 (Thư ký chép lời Board từ chat lên đó, nguyên văn kèm link).
2. Khi bật, agent tự quyết và làm luôn, kể cả việc thuộc danh sách việc quan trọng, ba loại việc với điều kiện ở `operating-model` mục 1b:
   - merge PR và deploy (CI/verify xanh, hoàn tác được, không migration mất dữ liệu, không đụng secret/auth/phân quyền, không dịch vụ trả phí mới);
   - apply hạ tầng (0 destroy, không nới IAM/security group/WAF, chi phí thêm dưới $30/tháng, đúng `terraform-plan-only`);
   - chọn phương án theo ít nhất 2 quyết định cũ cùng hướng, không gửi hay đăng ra ngoài.
3. Gửi hay đăng ra ngoài công ty, chi tiền ngoài mức trên, xoá dữ liệu production, `destroy`, làm yếu bảo mật, sửa agent/rules/skills/instruction, việc không hoàn tác được: vẫn chờ Board.
4. Mỗi lần tự quyết, agent ghi một dòng `Quyết thay ban đêm (D-0024)` kèm lý do và cách hoàn tác. 09:00 Thư ký tổng hợp cho Board; Board «ok» hoặc «lật N». Mục bị lật được hoàn tác. Câu trả lời của Board thành tiền lệ; loại việc được giữ nguyên 3 lần thì Thư ký đề xuất giao quyền hẳn.
