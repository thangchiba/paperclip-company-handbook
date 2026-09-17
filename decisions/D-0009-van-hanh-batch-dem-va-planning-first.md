# D-0009 — Vận hành batch ban đêm và planning-first

- **Ngày:** 2026-09-17
- **Phạm vi:** Toàn công ty
- **Nguồn:** Chỉ thị Board trên HOA-26 (comment 2026-09-17)

## Bối cảnh

Board nhận thấy: (1) mỗi task ban ngày assign lẻ tẻ tạo một run riêng, tốn budget; (2) ProductManager khi được nhờ lên kế hoạch lại nhảy sang code/thực thi ngay; (3) session transcript không phải nơi lưu kiến thức bền.

## Quyết định

1. **Engineer chạy batch ban đêm:** tắt wake-on-assignment, đặt timer 02:00 (JST). Prompt run đêm: "Xử lý lần lượt mọi issue `todo` được assign cho bạn theo ưu tiên cho đến khi hết hoặc chạm budget; mỗi issue một comment kết quả." Task ban ngày chỉ xếp hàng (`todo`), không tạo run ngay.
2. **Việc lớn nhiều bước:** một issue cha, các bước là subtask — giữ session của issue cha, không mở session mới cho từng mảnh.
3. **Reset session định kỳ:** ChiefOfStaff reset tối Chủ nhật (sau weekly review); Engineer reset khi đổi project hoặc khi thấy run rỗng. Kiến thức cần "nhớ" để trong file (handbook, docs repo, skill), không dựa vào transcript.
4. **Stable → skill:** cái gì ổn định để trong skill (nạp khi cần), không để trong instruction agent (nạp mọi run).
5. **Hạn chế @mention / comment qua lại giữa agent** — mỗi mention là một run.
6. **Planning-first:** ban ngày ưu tiên nghiên cứu + kế hoạch triển khai chi tiết, đặc biệt dự án mới. Không thực thi/code ngay trong giai đoạn planning. Khi planning: tóm tắt rõ, điểm chưa chắc chắn hỏi Board (A/B/C + đề xuất). Thực thi dồn vào ban đêm (~02:00).

## Ghi chú áp dụng

- Chi tiết thao tác nằm trong skill `skills/operating-model/` (cùng PR với quyết định này).
- Thay đổi adapterConfig/runtimeConfig của agent cần Board thao tác (hoặc cấp quyền `agents:configure` cho CEO) — agent run không có quyền PATCH agent config.
