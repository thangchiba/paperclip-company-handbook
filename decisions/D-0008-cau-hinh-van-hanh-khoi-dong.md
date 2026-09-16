# D-0008 — Cấu hình vận hành giai đoạn khởi động

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty
- **Nguồn:** Board trả lời card `ask_user_questions` trên HOA-2

## Bối cảnh

CEO gửi 5 câu hỏi cấu hình vận hành (repo handbook, vendor QA, lịch Thư ký, budget, timezone). Board đã trả lời đầy đủ.

## Quyết định

1. **Repo handbook:** remote GitHub `thangchiba/paperclip-company-handbook` (đã cấu hình, origin https).
2. **Vendor QA:** `codex_local` — khác vendor với dev (Claude) để review chéo.
3. **Lịch heartbeat Thư ký (ChiefOfStaff):** 09:00, 12:00, 18:00 mỗi ngày (JST).
4. **Budget:** chạy local adapter, không đặt trần chi phí per-agent ở giai đoạn này.
5. **Timezone chuẩn công ty:** Asia/Tokyo (JST).

## Ghi chú áp dụng

- QA và ChiefOfStaff đã được tuyển đúng vendor codex_local (HOA-4).
- Các quyết định 2–5 xác nhận đề xuất mặc định của CEO; không cần hỏi lại.
