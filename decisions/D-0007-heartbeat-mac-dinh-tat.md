# D-0007 — Heartbeat timer mặc định tắt

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (mọi agent)
- **Nguồn:** Chỉ thị Board v0.1, mục 3 và 6 (HOA-2)

## Bối cảnh

Heartbeat theo giờ tốn budget ngay cả khi không có việc. Đa số agent chỉ cần wake khi được giao task hoặc có comment.

## Phương án đã cân nhắc

- A: Mọi agent chạy heartbeat định kỳ.
- B: Heartbeat timer tắt mặc định; chỉ ngoại lệ có lý do rõ ràng.

## Quyết định

Chọn B:

1. Heartbeat timer tắt mặc định cho mọi agent; agent chỉ wake khi có việc (assign, comment, blocker resolved…).
2. Ngoại lệ duy nhất hiện tại: **Thư ký (Chief of Staff)** có heartbeat theo giờ để chạy Morning brief 07:00 và tổng kết 18:00 (chi tiết vai Thư ký theo mục 3 chỉ thị; instruction phải được Board duyệt trước khi kích hoạt).
3. Budget tháng theo agent: CEO đề xuất con số trong `handbook/rules.md` v0.1 (chờ Board duyệt).
