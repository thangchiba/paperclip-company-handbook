# D-0002 — Bộ nhớ quyết định: quyết một lần, dùng mãi

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (mọi agent)
- **Nguồn:** Chỉ thị Board v0.1, mục 2 (HOA-2)

*(Gọn 2026-10-03, HOA-418: mục 1 và câu cuối ghi luôn bản đã sửa theo D-0021, D-0022; bỏ mục 4 (rà soát tối Chủ nhật), hết hiệu lực theo D-0021. Bản cũ xem git log.)*

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

1. Board quyết một việc áp dụng rộng hơn một task → agent nhận quyết định trực tiếp từ Board ghi `decisions/D-xxxx-<slug>.md` trong ngày và ghi Hindsight theo `board-memory`. Quyết định trong một task chỉ ghi Hindsight.
2. Quyết định có tính khái quát → đề xuất sửa `rules.md` hoặc SKILL.md tương ứng dưới dạng approval card kèm diff. Board duyệt → merge và cập nhật skill trong thư viện Paperclip.
3. Không bao giờ sửa rules/skills mà không có approval của Board. Không bao giờ ghi quyết định Board chưa nói.
4. *(Đã bỏ: rà soát tối Chủ nhật, D-0021.)*

Agent không đọc `handbook/rules.md` đầu mỗi run: quy tắc đến qua khối "Decision authority" trong `AGENTS.md` và skill `security-baseline`, `operating-model` (D-0021).
