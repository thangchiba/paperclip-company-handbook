---
name: terraform-plan-only
description: Quy tắc an toàn hạ tầng cho InfraEngineer - chỉ terraform plan và mở PR, không bao giờ apply/destroy; apply do CI chạy sau khi Board duyệt. Dùng cho mọi việc Terraform/IaC.
---

# terraform-plan-only

Áp dụng D-0006. Hạ tầng của khách là rủi ro không đảo ngược — agent không bao giờ tự apply.

## Quy tắc

1. **Kiểm tra env trước:** Project phải có biến môi trường khai báo (AWS_PROFILE, region, backend/workspace, tên môi trường — theo D-0005). Thiếu env → dừng, gắn `needs-decision`, không chạy lệnh nào.
2. **Được phép:** `terraform init` (backend đã khai báo), `terraform validate`, `terraform plan`, mở PR kèm output plan (đã che giá trị nhạy cảm).
3. **Cấm tuyệt đối:** `terraform apply`, `terraform destroy`, `terraform import` làm thay đổi state, sửa state thủ công, xoá resource qua console/CLI.
4. **Apply** do CI chạy sau khi Board duyệt PR (Approval matrix, `handbook/rules.md` mục 3).
5. Trong PR ghi rõ: môi trường nào, resource thay đổi (add/change/destroy count), rủi ro.
6. Không in/log/commit giá trị secret; chỉ tham chiếu tên biến. Phát hiện lộ secret → báo ngay + đề xuất rotate.
