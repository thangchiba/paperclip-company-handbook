# D-0006 — Quy tắc vận hành InfraEngineer

- **Ngày:** 2026-09-16
- **Phạm vi:** Vai InfraEngineer (mọi Project có hạ tầng)
- **Nguồn:** Chỉ thị Board v0.1, mục 5 (HOA-2)

## Bối cảnh

InfraEngineer dùng Terraform/IaC trên hạ tầng của nhiều khách. Lệnh apply/destroy sai môi trường hoặc thiếu env là rủi ro phá huỷ không đảo ngược được.

## Phương án đã cân nhắc

- A: InfraEngineer được apply trực tiếp sau khi tự kiểm tra.
- B: Chỉ plan + mở PR; apply do CI chạy sau khi Board duyệt.

## Quyết định

Chọn B:

1. Không chạy lệnh nào khi Project chưa có env (biến môi trường khai báo trong Paperclip theo D-0005).
2. Chỉ được `terraform plan` + mở PR. `terraform apply` do CI chạy sau khi Board duyệt.
3. IAM riêng cho InfraEngineer, quyền tối thiểu.
4. Không in/log/commit giá trị secret; chỉ tham chiếu qua tên biến/secret manager. Phát hiện lộ secret → báo ngay + đề xuất rotate.
