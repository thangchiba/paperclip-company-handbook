# D-0003 — Cấu trúc project và cô lập ngữ cảnh

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (tổ chức project + mọi agent)
- **Nguồn:** Chỉ thị Board v0.1, mục 4 (HOA-2)

## Bối cảnh

Công ty vừa làm SaaS riêng vừa outsource nhiều khách. Cần cách tách các dự án để không lẫn ngữ cảnh, đồng thời không phình bộ máy agent.

## Phương án đã cân nhắc

- A: Mỗi khách hàng một Company riêng, nhân bản agent theo công ty.
- B: Một Company, tách ở tầng Project; agent là vai trò dùng chung.
- C: Một Company, một Project chung, phân biệt bằng label.

## Quyết định

Chọn B:

1. Giữ một Company. Mỗi repo/sản phẩm/khách hàng là một Project, đặt tên `<khách>-<sản phẩm>` (ví dụ `own-saas-x`, `clientA-app`, `personal-clientB`). Mỗi Project gắn cwd + repoUrl + biến môi trường riêng.
2. Agent là vai trò dùng chung cho mọi Project. KHÔNG tạo PM hay dev riêng theo project. Khi tải lớn: thêm agent thứ hai cùng vai (ví dụ FullstackDev-2), không nhân bản theo project.
3. Ngữ cảnh dự án nằm trong repo: `CLAUDE.md`/`AGENTS.md`, `.claude/skills`, `docs/`. Agent phải chạy đúng cwd của Project. Kiến thức dự án học được → ghi vào `docs/` của repo đó qua PR, không giữ trong bộ nhớ agent.
4. Cô lập ghi đè: bật worktree isolation cho từng Project nếu Paperclip hỗ trợ (cần kiểm tra và báo Board); nếu không, quy tắc cứng: một dev agent trên một Project tại một thời điểm.
