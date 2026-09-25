---
name: terraform-plan-only
description: Quy tắc an toàn hạ tầng cho InfraEngineer (D-0006 + D-0011). Dự án khách - chỉ terraform plan và mở PR, apply do CI sau khi Board duyệt. Dự án nội bộ Hoang (Pro5) - được apply hoặc chạy lệnh CLI có thay đổi CHỈ SAU KHI đăng bản tóm tắt thay đổi bằng card request_confirmation (human_only) và Board chấp nhận. Dùng cho mọi việc Terraform/IaC/AWS CLI.
---

# terraform-plan-only

Áp dụng D-0006 và D-0011. Thay đổi hạ tầng là rủi ro khó đảo ngược, nên mặc định agent **không tự apply**. Ngoại lệ duy nhất: dự án **nội bộ của Hoang**, và chỉ sau khi Board duyệt **bản tóm tắt thay đổi** của đúng lần apply đó.

## 0. Xác định loại dự án trước khi làm gì

| Loại | Dự án | Quy tắc |
|---|---|---|
| **Nội bộ Hoang** | `Pro5` (repo `thangchiba/pro5`; AWS account riêng của Pro5 do InfraEngineer tạo theo HOA-111 và AWS management account của Hoang trong phạm vi tạo/sửa account đó) | Mục 2 — apply sau khi Board duyệt card tóm tắt |
| **Khách (outsourcing)** | Mọi dự án còn lại, kể cả dự án chưa được Board xếp loại (ví dụ `Sirisugi`, `Figure`) | Mục 1 — plan-only, apply do CI |

Chỉ Board mới thêm dự án vào danh sách "nội bộ" (sửa file này qua approval theo `handbook/rules.md` mục 3). Chưa có trong danh sách = coi là dự án khách.

## Quy tắc chung (mọi dự án)

- **Kiểm tra env trước:** Project phải có biến môi trường khai báo (`AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` hoặc `AWS_PROFILE`, `AWS_DEFAULT_REGION`, backend/workspace, tên môi trường — theo D-0005). Thiếu env → dừng, gắn `needs-decision`, không chạy lệnh nào. Chạy `aws sts get-caller-identity` để chắc đang ở đúng tài khoản trước mọi lệnh có thay đổi.
- **Luôn được phép:** `terraform init` (backend đã khai báo), `terraform validate`, `terraform plan`, `terraform show`, các lệnh AWS CLI **chỉ đọc** (`describe-*`, `list-*`, `get-*`, `sts get-caller-identity`), mở PR kèm output plan (đã che giá trị nhạy cảm).
- **Không bao giờ** in/log/commit giá trị secret; chỉ tham chiếu tên biến. Output plan/apply dán lên PR hoặc issue phải che access key, password, token, connection string. Phát hiện lộ secret → báo ngay + đề xuất rotate.
- Không thao tác lên tài nguyên thuộc dự án khác (NDA D-0004), kể cả khi credential hiện tại nhìn thấy được.

## 1. Dự án khách — plan-only (D-0006, không đổi)

1. **Cấm tuyệt đối:** `terraform apply`, `terraform destroy`, `terraform import`, `terraform state *` làm thay đổi state, sửa state thủ công, mọi thay đổi qua console/CLI (create/update/delete).
2. **Apply** do CI chạy sau khi Board duyệt PR (Approval matrix, `handbook/rules.md` mục 3).
3. Trong PR ghi rõ: môi trường nào, resource thay đổi (add/change/destroy count), rủi ro.

## 2. Dự án nội bộ Hoang — apply sau khi Board duyệt tóm tắt (D-0011)

Agent được chạy `terraform apply` hoặc lệnh CLI có thay đổi (ví dụ `aws organizations create-account`, `aws iam create-role`) **chỉ khi đủ cả 3 bước a → b → c**.

### Bước a — Chuẩn bị bằng chứng

- Terraform: chạy `terraform plan -out=<plan-file>` rồi `terraform show -no-color <plan-file>` (che giá trị nhạy cảm). Ghi lại đường dẫn plan file và `sha256sum <plan-file>`.
- CLI/console không qua Terraform: liệt kê **chính xác** từng lệnh sẽ chạy, theo thứ tự, kèm tham số (che secret). Không ghi chung chung kiểu "tạo account".

### Bước b — Đăng bản tóm tắt thay đổi và chờ Board

Tạo card trên issue đang làm bằng `POST /api/issues/{issueId}/interactions`:

- `kind`: `request_confirmation`
- `resolverPolicy`: `human_only` (bắt buộc — chỉ Board duyệt, agent khác không được chấp nhận thay)
- `continuationPolicy`: `wake_assignee`
- `idempotencyKey`: `infra-apply:{issueId}:{sha256 plan file, hoặc hash danh sách lệnh}`
- `payload.detailsMarkdown` phải có đủ 8 mục:
  1. **Tài khoản / môi trường:** AWS account id, region, Terraform workspace, dự án.
  2. **Số lượng thay đổi:** `N to add, M to change, K to destroy` (hoặc số lệnh CLI).
  3. **Danh sách resource / lệnh:** từng resource address (`aws_iam_role.x`) hoặc từng lệnh CLI.
  4. **Quyền IAM mới:** role/policy/user nào được tạo hoặc mở rộng, mức quyền.
  5. **Chi phí:** ước tính USD/tháng; dịch vụ trả phí mới cũng là mục chi tiêu cần Board duyệt (rules mục 3).
  6. **Rủi ro:** điều gì có thể hỏng, ảnh hưởng dữ liệu/downtime, có đảo ngược được không.
  7. **Rollback:** cách hoàn tác cụ thể (lệnh/PR), hoặc ghi rõ "không đảo ngược được" (ví dụ tạo AWS account).
  8. **Plan file:** đường dẫn + sha256 (hoặc hash danh sách lệnh) sẽ apply.

Sau khi tạo card: chuyển issue sang `in_review` (kèm `reviewInteractionId`) với comment nêu rõ đang chờ Board chấp nhận card. Không apply khi card còn `pending`, hoặc đã `rejected`, `expired`, `withdrawn`, `superseded_by_comment`.

### Bước c — Apply đúng cái đã duyệt

- Terraform: `terraform apply <plan-file>` — **đúng file plan đã duyệt** (so lại sha256 trước khi apply). Không `terraform apply` không có plan file, không `-auto-approve` trên plan mới.
- CLI: chạy **đúng danh sách lệnh** đã đăng, đúng thứ tự, không thêm lệnh.
- Plan thay đổi (plan lại ra kết quả khác, drift, sửa code, đổi biến) hoặc phải thêm/bớt lệnh → **dừng, tạo card mới**, chờ duyệt lại. Card cũ không dùng lại.
- Sau apply: dán tóm tắt kết quả lên issue (số resource thực tế add/change/destroy, output đã che giá trị nhạy cảm, lỗi nếu có). Apply thất bại giữa chừng → ghi rõ trạng thái hiện tại và phương án, không tự "sửa nhanh" bằng lệnh chưa duyệt.
- Mỗi card chỉ có hiệu lực cho **một lần apply**. Lần apply tiếp theo (kể cả cùng môi trường) cần card mới.

### Vẫn cấm ở dự án nội bộ nếu không có card riêng, tường minh, cho từng lần

- `terraform destroy`, hoặc plan có destroy/replace data store (DynamoDB, S3 bucket có dữ liệu, RDS/Aurora, EBS, Neon/DB bên ngoài). Plan có các mục này → card phải **tách riêng**, tiêu đề ghi in đậm "CÓ DESTROY/REPLACE DATA STORE"; không gộp vào card apply thường.
- `terraform import`, `terraform state mv/rm/push`, sửa state thủ công.
- Xoá/sửa tài nguyên của dự án khác, hoặc của AWS management account ngoài phạm vi đã duyệt.
- Cấp quyền IAM Admin (`AdministratorAccess`, `*:*`) cho user/role mới; mở public ingress (0.0.0.0/0) tới data store.

## 3. Báo cáo

Kết thúc mỗi phiên infra, comment theo khung ①②③ (`verify-and-report`): ① kết quả + link PR/plan/card, ② việc chờ Board (card nào), ③ bước tiếp theo.
