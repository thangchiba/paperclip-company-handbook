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

1. Chế độ tự quyết bật 22:00–09:00 JST mỗi ngày. Board bật, nới hay đổi giờ bằng comment của chính Board trên HOA-395. Thư ký chỉ được chép lời Board (nguyên văn, kèm link) lên đó để tắt hay thu hẹp chế độ. Lời không nêu hạn hết hạn lúc 09:00 JST kế tiếp.
2. Khi bật, với những việc Board chưa nói, agent tự quyết và làm luôn, kể cả việc thuộc danh sách việc quan trọng, đúng ba loại với điều kiện ở `operating-model` mục 1b:
   - merge PR và deploy: chỉ FullstackDev và InfraEngineer, PR của chính mình; check và GitGuardian xanh; hoàn tác được; không migration mất dữ liệu; không đụng secret, auth, phân quyền, cổng kiểm, đường deploy hay file hướng dẫn agent; không dịch vụ trả phí; không đổi điều người ngoài thấy hay nhận trừ phần đúng như lời Board trên task; chỉ deploy do merge vào `main` kéo theo ở Project đang để merge = deploy;
   - apply hạ tầng: chỉ InfraEngineer, chỉ Pro5 (`terraform-plan-only` mục 2c): 0 destroy, 0 replace, không nới bảo mật hay quyền public, không bật gửi ra ngoài, tổng chi AWS của Pro5 vẫn dưới $30/tháng (D-0021 mục 12). Dự án khách vẫn chỉ plan;
   - chọn phương án theo ít nhất 2 quyết định cũ cùng hướng mà Board tự chọn trước đó («ok» trên bản tổng hợp không tính); không chi tiền, giá bán, bảo mật, dữ liệu khách, gửi hay đăng ra ngoài.
3. Vẫn chờ Board, kể cả ban đêm: mọi khoản chi hay dịch vụ trả phí mới (trừ AWS Pro5 ở trên), gửi hay đăng ra ngoài công ty, xoá dữ liệu production, `destroy`, force-push, làm yếu bảo mật, sửa agent/rules/skills/instruction, phần dùng chung ngoài Hoang LLC trên MacbookServer, việc không hoàn tác được. Không dùng D-0024 khi task đang có card chờ Board, khi Board đã bảo chờ, hay để hoãn việc ban ngày tới đêm.
4. Mỗi lần tự quyết, agent ghi một dòng `Quyết thay ban đêm (D-0024)` kèm lý do và cách hoàn tác. 09:00 Thư ký tổng hợp kèm card để Board giữ hay lật từng mục. Mục bị lật được hoàn tác. Một loại việc được giữ 3 lần liên tiếp thì Thư ký đề xuất giao quyền cả ban ngày.

## Sửa đổi sau khi soát (2026-10-03 01:00)

Một lượt soát độc lập tìm ra chỗ bản đầu rộng hơn lời Board: agent nào cũng merge được, apply cả dự án khách, $30 tính theo từng lần thay vì tổng, deploy có thể gửi email cho khách, người khác Board có thể bật chế độ. Bản trên thu hẹp lại cho khớp lựa chọn của Board và các quy tắc cũ (D-0006, D-0013, D-0021 mục 12), sửa `pr-standard`, `terraform-plan-only` (mục 2c), `security-baseline`, `content-draft-protocol`.
