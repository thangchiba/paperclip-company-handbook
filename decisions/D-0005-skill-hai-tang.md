# D-0005 — Skill hai tầng và env theo project

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (quản lý skill + cấu hình env)
- **Nguồn:** Chỉ thị Board v0.1, mục 5 (HOA-2)

## Bối cảnh

Skill vừa có loại gắn với một repo cụ thể, vừa có loại dùng chung theo vai. Cần tránh trùng nguồn (một skill hai bản lệch nhau) và tránh secret lọt vào Paperclip.

## Phương án đã cân nhắc

- A: Copy mọi skill vào thư viện Paperclip.
- B: Skill hai tầng: tầng project ở trong repo, tầng công ty ở `company-handbook/skills/` rồi import vào Paperclip.

## Quyết định

Chọn B:

1. **Tầng project:** giữ nguyên `.claude/skills` trong repo của project, không copy vào Paperclip.
2. **Tầng công ty:** skill dùng chung theo vai, nguồn sự thật là `company-handbook/skills/`, import vào thư viện Paperclip và gắn cho agent tương ứng.
3. Danh sách skill công ty v0.1 (CEO đề xuất, tối thiểu): `verify-and-report`, `needs-decision-protocol`, `pr-standard`, `terraform-plan-only`, `content-draft-protocol`, `daily-brief`.
4. Mỗi skill chỉ có một nguồn. Skill kéo từ GitHub/skills.sh phải ghim theo commit.
5. Env theo project: mỗi Project khai báo biến môi trường riêng trong Paperclip (AWS_PROFILE, region, Terraform backend/workspace, tên môi trường). Không có giá trị secret nào trong Paperclip hay trong prompt; secret nằm trong profile/secret manager trên máy chủ theo từng khách, agent chỉ tham chiếu tên.
