---
name: terraform-plan-only
description: Quy tắc an toàn hạ tầng (D-0006, D-0011, D-0021). Dự án khách - chỉ terraform plan và mở PR, apply do CI sau khi Board duyệt. Dự án nội bộ Hoang (Pro5) - tới launch M1, thay đổi giữ chi phí AWS của Pro5 dưới $30/tháng và không ảnh hưởng security thì InfraEngineer tự apply rồi báo cáo; còn lại apply chỉ sau card request_confirmation human_only Board chấp nhận. Dùng cho mọi việc Terraform/IaC/AWS CLI có thay đổi.
---

# terraform-plan-only

Thay đổi hạ tầng khó hoàn tác, nên mặc định agent không tự apply, trừ các trường hợp ở mục 2.

## 0. Loại dự án

| Loại | Dự án | Quy tắc |
|---|---|---|
| Nội bộ Hoang | `Pro5` (repo `thangchiba/pro5`; AWS account riêng của Pro5 theo HOA-111, và AWS management account của Hoang trong phạm vi tạo hay sửa account đó) | Mục 2 |
| Khách | Mọi dự án còn lại, kể cả dự án Board chưa xếp loại (ví dụ `Sirisugi`, `Figure`) | Mục 1 |

Chỉ Board thêm dự án vào danh sách nội bộ. Chưa có trong danh sách thì là dự án khách.

## Quy tắc chung

- Trước lệnh có thay đổi: Project có đủ env (credential Paperclip inject, `AWS_DEFAULT_REGION`, backend/workspace, tên môi trường; D-0005), và `sts get-caller-identity` đúng như khối HOA-196 trong `AGENTS.md`. Thiếu env: dừng, gắn `needs-decision`, không chạy lệnh nào.
- Provider và backend Terraform không dùng `profile` hay `shared_credentials_files`.
- Luôn được phép: `terraform init` (backend đã khai báo), `validate`, `plan`, `show`; lệnh AWS CLI chỉ đọc (`describe-*`, `list-*`, `get-*`); mở PR kèm output plan.
- Output plan hay apply dán lên PR, issue: che access key, password, token, connection string.
- Không đụng tài nguyên của dự án khác, kể cả khi credential nhìn thấy được.

## 1. Dự án khách: chỉ plan (D-0006)

- Cấm: `terraform apply`, `destroy`, `import`, `state` làm đổi state, sửa state tay, mọi thay đổi qua console hay CLI. Kể cả ban đêm (D-0024).
- Apply do CI chạy sau khi Board duyệt PR. PR ghi rõ môi trường, số resource add/change/destroy, rủi ro.

## 2. Dự án nội bộ (Pro5)

InfraEngineer apply. Luôn `terraform plan -out=<plan-file>`, đọc diff, rồi `terraform apply <plan-file>` đúng file đó. Không apply khi không có plan file, không `-auto-approve` trên plan mới.

### 2a. Tự apply, không cần card (tới launch M1)

Lời Board: «Phần infra nếu không phát sinh nhiều chi phí thì không cần hỏi ý kiến tôi» (HOA-220, 29/09); «Dưới $30/tháng thì khỏi hỏi» (2026-10-02, D-0021).

Tự apply khi đủ cả 4 điều:

1. Sau thay đổi, dự báo tổng chi AWS của Pro5 vẫn dưới $30/tháng.
2. Không ảnh hưởng security: không đổi IAM (role, policy, user, trust), security group hay ingress, secret, auth, mã hoá, quyền truy cập public.
3. Không destroy hay replace data store, không đụng dữ liệu production, không có gì không hoàn tác được (ví dụ tạo AWS account).
4. Không `import`, `state mv/rm/push`, sửa state tay.

Sau apply, comment theo `concise-status-report`: số resource add/change/destroy, dự báo chi phí mỗi tháng, link PR hoặc plan. Thiếu một điều kiện, hay không chắc, thì làm theo 2b. Từ launch M1 (HOA-194), mọi apply Pro5 theo 2b.

### 2b. Apply sau card Board (D-0011)

1. **Bằng chứng:** plan file, `terraform show -no-color <plan-file>` (đã che), `sha256sum <plan-file>`. Lệnh CLI không qua Terraform: liệt kê chính xác từng lệnh theo thứ tự, kèm tham số (che secret).
2. **Card** trên issue đang làm: `request_confirmation`, `resolverPolicy: human_only`, `continuationPolicy: wake_assignee`, `idempotencyKey: infra-apply:{issueId}:{sha256}`. `payload.detailsMarkdown` đủ 8 mục:
   1. account id, region, workspace, dự án;
   2. `N to add, M to change, K to destroy`, hoặc số lệnh CLI;
   3. từng resource address hoặc từng lệnh;
   4. quyền IAM mới và mức quyền;
   5. chi phí USD/tháng;
   6. rủi ro: dữ liệu, downtime, hoàn tác được không;
   7. rollback cụ thể, hoặc "không hoàn tác được";
   8. plan file và sha256.
   Chuyển issue `in_review`. Chỉ apply khi card được chấp nhận, hoặc Board đồng ý rõ ràng bằng comment trên task (link).
3. **Apply** đúng plan file đã duyệt (so lại sha256), hoặc đúng danh sách lệnh. Plan đổi hay cần thêm lệnh: dừng, tạo card mới. Mỗi card chỉ cho một lần apply.
4. Sau apply, dán kết quả lên issue (đã che). Lỗi giữa chừng: ghi trạng thái hiện tại và phương án, không sửa nhanh bằng lệnh chưa duyệt.

### 2c. Ban đêm (D-0024)

Khi chế độ tự quyết bật (`operating-model` mục 1b), InfraEngineer tự apply Pro5 mà 2b đòi card, không cần card, khi đủ cả:
1. Sau thay đổi, dự báo tổng chi AWS mỗi tháng của Pro5, cộng mọi apply từ 22:00, vẫn dưới $30/tháng (D-0021 mục 12); ghi số trước và sau.
2. Plan 0 destroy, 0 replace; không thuộc mục "Luôn cần card riêng" dưới.
3. Không nới IAM, security group, ingress, WAF hay quy tắc chặn; không tạo IAM user, access key hay principal có quyền mới; không đổi secret, KMS hay mã hoá, quyền truy cập public (S3 public access block, bucket policy, CloudFront/OAC, API không auth), DNS hay tunnel Cloudflare; không bật hay nới gửi ra ngoài (SES ra khỏi sandbox, quyền SendMail, SNS/SMS).

Làm đúng bước 1, 3, 4 của 2b (plan file, `sha256sum`, apply đúng file đó, không `-auto-approve`); thay card bằng comment `Quyết thay ban đêm (D-0024): apply Pro5 · <sha256> (N add, M change, 0 destroy, $trước→$sau/tháng) — vì điều kiện 1–3 đạt. Hoàn tác: <plan đảo ngược>`. Dự án khách (mục 1): ban đêm vẫn chỉ plan.

### Luôn cần card riêng cho từng lần (kể cả trong 2a)

- `terraform destroy`, hoặc plan có destroy/replace data store (DynamoDB, S3 có dữ liệu, RDS/Aurora, EBS, DB bên ngoài): card tách riêng, tiêu đề in đậm "CÓ DESTROY/REPLACE DATA STORE".
- `terraform import`, `state mv/rm/push`, sửa state tay.
- Tài nguyên của dự án khác, hay của management account ngoài phạm vi đã duyệt.
- Cấp IAM Admin (`AdministratorAccess`, `*:*`) cho user hay role mới; mở `0.0.0.0/0` tới data store.

## 3. Báo cáo

Kết thúc mỗi phiên hạ tầng: comment theo `concise-status-report`, gồm kết quả kèm link PR, plan, card; việc chờ Board; bước tiếp theo.
