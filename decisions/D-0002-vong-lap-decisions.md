# D-0002 — Bộ nhớ quyết định: quyết một lần, dùng mãi

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (mọi agent)
- **Nguồn:** Chỉ thị Board v0.1, mục 2 (HOA-2)

## Bối cảnh

Quyết định của Board bị hỏi lại nhiều lần nếu không được ghi lại có hệ thống. Cần một nơi lưu duy nhất mà mọi agent tra cứu trước khi hỏi.

## Phương án đã cân nhắc

- A: Lưu quyết định rải rác trong comment các issue.
- B: Repo `company-handbook` với `decisions/`, `handbook/rules.md`, `playbooks/`, `skills/` và vòng lặp cập nhật bắt buộc.
- C: Lưu trong bộ nhớ riêng của từng agent.

## Quyết định

Chọn B. Project `company-handbook` gắn repo git (cwd trên máy chủ), cấu trúc:

- `decisions/D-xxxx-<slug>.md` — mỗi quyết định một file: bối cảnh, phương án đã cân nhắc, quyết định, ngày, phạm vi áp dụng (toàn công ty / một project / một vai).
- `handbook/rules.md` — bộ quy tắc hiện hành, có số phiên bản.
- `playbooks/` — quy trình lặp lại (release, onboarding project mới, xử lý sự cố, đăng nội dung).
- `skills/` — nguồn sự thật của skill dùng chung theo vai, dạng SKILL.md.

Vòng lặp bắt buộc:

1. Board quyết → trong cùng ngày ghi `decisions/D-xxxx.md`.
2. Quyết định có tính khái quát → đề xuất sửa `rules.md` hoặc SKILL.md tương ứng dưới dạng approval card kèm diff. Board duyệt → merge và cập nhật skill trong thư viện Paperclip.
3. Không bao giờ sửa rules/skills mà không có approval của Board. Không bao giờ ghi quyết định Board chưa nói.
4. Tối Chủ nhật: rà lại decisions trong tuần, gộp trùng, đề xuất nâng thành rule.

Mọi agent đọc `handbook/rules.md` ở đầu mỗi run (đưa vào instruction của từng agent).
