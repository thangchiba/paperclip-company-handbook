# D-0007 — Heartbeat timer mặc định tắt

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (mọi agent)
- **Nguồn:** Chỉ thị Board v0.1, mục 3 và 6 (HOA-2)

*(Gọn 2026-10-03, HOA-418: bỏ mục 2 (heartbeat của Thư ký cũ, hết hiệu lực theo D-0021) và mục 3 (đề xuất budget theo agent, thay bởi D-0008 mục 4). Bản cũ xem git log.)*

## Bối cảnh

Heartbeat theo giờ tốn budget ngay cả khi không có việc. Đa số agent chỉ cần wake khi được giao task hoặc có comment.

## Phương án đã cân nhắc

- A: Mọi agent chạy heartbeat định kỳ.
- B: Heartbeat timer tắt mặc định; chỉ ngoại lệ có lý do rõ ràng.

## Quyết định

Chọn B:

1. Heartbeat timer tắt mặc định cho mọi agent; agent chỉ wake khi có việc (assign, comment, blocker resolved…).
2. *(Đã bỏ: heartbeat theo giờ của Thư ký cũ, D-0021.)*
3. *(Đã bỏ: đề xuất budget theo agent; Board chọn không đặt trần per-agent, D-0008 mục 4.)*
