# Skill công ty — v0.1 (chờ Board duyệt, HOA-6)

Nguồn sự thật của skill dùng chung theo vai (D-0005). Sau khi Board duyệt: import vào thư viện Paperclip và gắn cho agent theo bảng dưới. Mỗi skill chỉ có một nguồn — sửa ở đây trước, qua approval, rồi mới sync vào Paperclip.

| Skill | Gắn cho vai |
|---|---|
| `verify-and-report` | Mọi agent |
| `needs-decision-protocol` | Mọi agent |
| `secret-hygiene` | Mọi agent (HOA-177: cấm cả "hình dạng secret" trong git, tự quét trước commit) |
| `pr-standard` | Agent dev (FullstackDev, InfraEngineer…) |
| `terraform-plan-only` | InfraEngineer |
| `content-draft-protocol` | Agent nội dung/marketing |
| `daily-brief` | Thư ký (Chief of Staff) |

Skill tầng project nằm trong `.claude/skills` của repo từng Project, không copy vào đây (D-0005).
