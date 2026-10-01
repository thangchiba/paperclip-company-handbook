# D-0015 — Lộ secret qua `ps` (HOA-312): không rotate, Board redact log; cấm in argv tiến trình

- **Ngày:** 2026-10-01
- **Phạm vi:** Mục 1: chỉ sự cố HOA-312. Mục 2–4: toàn công ty (mọi agent, mọi máy agent chạy hoặc ssh vào).
- **Nguồn:** Board trả lời thẻ "SECURITY: rotate key + duyệt rule (v2)" trên HOA-312 (câu 1 chọn B, câu 2 chọn A); CTO đề xuất và thực hiện tại HOA-312.
- **Sửa đổi:** Không sửa D nào. Thay hướng dẫn cũ "xem tiến trình bằng `ps -o pid,etime,comm` hoặc `pgrep -l`": bỏ `pgrep -l`.

## Bối cảnh

- Ngày 01/10, khoảng 10:02 JST, run CTO dd3e5325 (HOA-291) chạy `ps -eo pid,etime,cmd`. Launcher trên ThangChiba-Desktop (`~/.paperclip/ssh-shell.sh`) giữ cả dòng launch, kèm giá trị secret, trong argv của `setsid` suốt run. Vì vậy output in ra mọi secret của run, và chúng nằm lại trong log run.
- Secret dài hạn bị lộ (chỉ ghi tên): `THANGCHIBA_AWS_ACCESS_KEY` / `THANGCHIBA_AWS_SECRET_ACCESS_KEY` (IAM user `doraemon`), `CLOUDFLARE_NEOKUN_API_TOKEN`, `GEMINI_API_KEY`, `HINDSIGHT_API_KEY`. `PAPERCLIP_API_KEY` và `PAPERCLIP_GITHUB_BROKER_TOKEN` gắn với run, hết tác dụng khi run kết thúc.
- CTO sửa launcher lúc 10:28 JST (dòng launch đi qua fd 3, không qua argv) và redact các bản sao trên ThangChiba-Desktop. Còn bản trong log run trên host Paperclip (MacbookServer), nơi agent không vào được.
- Danh sách lệnh cấm của HOA-136 chưa có lệnh in argv của tiến trình.

## Phương án đã cân nhắc

Câu 1, rotate các key dài hạn đã lộ:

- A: Rotate Cloudflare + Gemini; key `doraemon` giữ nguyên theo tiền lệ HOA-130. CTO và CEO đề xuất.
- B: Không rotate; Board/operator redact log run dd3e5325 trên host Paperclip. Board chọn.
- C: Không rotate, không redact.

Câu 2, sửa rule chống lộ secret:

- A: Duyệt diff (skill `secret-hygiene` mục 4 và khối HOA-136 trong `AGENTS.md`). Board chọn.
- B: Không sửa rule, sửa launcher là đủ.

## Quyết định

1. **Không rotate** các key dài hạn lộ trong HOA-312. Board/operator redact log run dd3e5325 trên host Paperclip.
2. **Không in dòng lệnh (argv) của tiến trình.** Cấm `ps -ef`, `ps aux`, `ps -o args|cmd|command`, `pgrep -a`/`-l`, `pstree -a`, `top -c`/`htop`, `/proc/*/cmdline`, và thêm `ps -E`/`ps e` vào danh sách cấm in env. Kiểm tiến trình bằng `ps -o pid,etime,comm` hoặc `pgrep -f <pattern> >/dev/null && echo running`.
3. **Quét secret chỉ in số đếm hoặc true/false**, không in đoạn khớp hay ngữ cảnh quanh nó. Bỏ qua kho credential (`~/.claude/.credentials.json`, `~/.ssh`, `~/.aws`). Giá trị secret đọc từ env bên trong script, không đưa vào argv của `grep`/`curl`.
4. Văn bản đầy đủ của mục 2–3 nằm ở `skills/secret-hygiene` mục 4 và khối "Secret hygiene (company rule, HOA-136 / HOA-130)" trong `AGENTS.md`.

## Hệ quả

- `skills/secret-hygiene` mục 4: thêm hai quy tắc trên, đúng diff Board duyệt (document `rule-diff` rev 3 trên HOA-312). Sau khi merge, cập nhật skill trong thư viện Paperclip.
- Khối HOA-136 trong `AGENTS.md` của 9 agent: thay bằng bản mới trong `rule-diff`, lần tới khi có agent vào được host Paperclip. Theo dõi tại HOA-319.
- Key Cloudflare và Gemini vẫn dùng được cho tới khi có người rotate; Board chấp nhận rủi ro này. Nếu lộ thêm lần nữa, đề xuất rotate lại (rules.md 4.4).
- Sửa `~/.paperclip/ssh-shell.sh` phải giữ secret ngoài argv, và agent vẫn là con trực tiếp của `setsid -w` (chi tiết ở document `launcher-fix` trên HOA-312).
