# D-0011 — InfraEngineer được apply ở dự án nội bộ Hoang sau khi Board duyệt bản tóm tắt thay đổi

- **Ngày:** 2026-09-25
- **Phạm vi:** Vai InfraEngineer; dự án nội bộ của Hoang (hiện tại: Pro5)
- **Nguồn:** Board trả lời card trên HOA-111 (Pro5 hạ tầng); CEO thực hiện tại HOA-128
- **Sửa đổi:** D-0006 (thêm ngoại lệ cho dự án nội bộ; dự án khách giữ nguyên)

## Bối cảnh

D-0006 cấm tuyệt đối agent `terraform apply` và mọi thay đổi qua CLI; apply chỉ do CI chạy sau khi Board duyệt PR. Pro5 là sản phẩm nội bộ của Hoang, chưa có CI/pipeline hạ tầng, và cần InfraEngineer tự tạo AWS Organization/member account rồi bootstrap. Với quy tắc cũ, InfraEngineer bị kẹt dù Board đã chọn phương án.

Board trả lời (HOA-111): *"Tôi nghĩ nên yêu cầu CEO sửa lại skill, có cho phép apply ở những dự án nội bộ của Hoang. Tuy nhiên thì trước khi apply cần tóm tắt lại những thứ sẽ modify để tôi ra quyết định."*

## Phương án đã cân nhắc

- A: Giữ nguyên D-0006, dựng CI cho Pro5 trước rồi mới apply qua CI. Chậm, và bootstrap account vẫn phải làm tay.
- B: Cho phép apply ở dự án nội bộ sau khi Board duyệt bản tóm tắt thay đổi cho từng lần apply; dự án khách giữ plan-only.
- C: Cho phép apply tự do ở dự án nội bộ. Rủi ro cao, Board không chọn.

## Quyết định

Chọn B:

1. **Dự án khách (outsourcing):** giữ nguyên D-0006 — chỉ plan + PR, apply do CI sau khi Board duyệt.
2. **Dự án nội bộ của Hoang** (danh sách trong `skills/terraform-plan-only/SKILL.md` mục 0; hiện tại chỉ `Pro5`): InfraEngineer được `terraform apply` hoặc chạy lệnh CLI có thay đổi khi đủ cả 3 bước:
   - a. Đã `terraform plan -out=<file>` (hoặc liệt kê chính xác lệnh CLI) và đăng **bản tóm tắt thay đổi** lên issue bằng card `request_confirmation` với `resolverPolicy: human_only`. Tóm tắt gồm: tài khoản/môi trường, số add/change/destroy, danh sách resource, quyền IAM mới, chi phí, rủi ro, cách rollback, hash plan file.
   - b. Board chấp nhận card đó.
   - c. Apply **đúng file plan đã duyệt** (so hash). Plan thay đổi → hỏi lại bằng card mới. Mỗi card có hiệu lực cho một lần apply.
3. Vẫn cấm nếu không có card riêng cho từng lần: `destroy`, destroy/replace data store, sửa/import state, thao tác trên tài nguyên dự án khác, cấp quyền IAM Admin cho principal mới.
4. Không in/commit secret; kết quả apply dán lên issue phải che giá trị nhạy cảm.
5. Chỉ Board mới thêm dự án vào danh sách "nội bộ" (sửa skill qua approval theo rules mục 3). Chưa có trong danh sách = dự án khách.

## Hệ quả

- `skills/terraform-plan-only/SKILL.md` viết lại theo D-0011 và sync vào Paperclip cho InfraEngineer.
- `handbook/rules.md` mục 3: hàng "terraform apply" bổ sung nhánh dự án nội bộ.
- Card tóm tắt cho Pro5 (tạo AWS account + bootstrap) do InfraEngineer đăng trên HOA-111.
