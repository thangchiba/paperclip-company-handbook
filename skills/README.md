# Skill công ty — v0.3 (D-0021, D-0022, Board duyệt 2026-10-02)

Nguồn sự thật của skill dùng chung (D-0005). Sửa ở đây trước, Board duyệt, rồi mới sync vào thư viện Paperclip và gắn cho agent theo bảng dưới.

| Skill | Gắn cho |
|---|---|
| `operating-model` | Mọi agent |
| `security-baseline` | Mọi agent |
| `secret-hygiene` | Mọi agent |
| `board-memory` | Mọi agent |
| `order-dispatch` | ChiefOfStaff |
| `pr-standard` | FullstackDev, InfraEngineer |
| `dev-machine` | ChiefOfStaff, FullstackDev, InfraEngineer, QA |
| `terraform-plan-only` | InfraEngineer |
| `content-draft-protocol` | MarketingManager, ContentCreator |
| `hoang-auth` | InfraEngineer, FullstackDev khi dự án cần đăng nhập (D-0016); chưa vào thư viện (D-0020) |
| `daily-brief` | Không gắn: daily brief đã huỷ (HOA-82) |
| `needs-decision-protocol`, `verify-and-report` | Không gắn: đã gộp vào `operating-model` và `concise-status-report` |

`concise-status-report` là skill riêng của công ty trong thư viện Paperclip (không nằm trong repo này), gắn cho mọi agent.

Skill tầng project nằm trong `.claude/skills` của repo từng Project, không copy vào đây (D-0005).
