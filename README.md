# Company Handbook — Hoang LLC

Bộ nhớ quyết định và quy tắc vận hành của công ty. Nguồn sự thật cho mọi agent.

## Cấu trúc

- `decisions/D-xxxx-<slug>.md` — mỗi quyết định của Board một file: bối cảnh, phương án đã cân nhắc, quyết định, ngày, phạm vi áp dụng.
- `handbook/rules.md` — bộ quy tắc hiện hành, có số phiên bản. Mọi agent đọc file này ở đầu mỗi run.
- `playbooks/` — quy trình lặp lại (release, onboarding project mới, xử lý sự cố, đăng nội dung).
- `skills/` — nguồn sự thật của các skill dùng chung theo vai (dạng SKILL.md), import vào thư viện Paperclip.

## Vòng lặp bắt buộc (theo D-0002)

1. Board quyết → trong cùng ngày ghi `decisions/D-xxxx.md`.
2. Quyết định có tính khái quát → đề xuất sửa `rules.md`/SKILL.md qua approval card kèm diff. Board duyệt → merge và cập nhật skill trong thư viện Paperclip.
3. Không sửa rules/skills khi chưa có approval của Board. Không ghi quyết định Board chưa nói.
4. Tối Chủ nhật: rà lại decisions trong tuần, gộp trùng, đề xuất nâng thành rule.
