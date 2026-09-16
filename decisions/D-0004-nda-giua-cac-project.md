# D-0004 — NDA giữa các Project

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (mọi agent, đặc biệt dev agent)
- **Nguồn:** Chỉ thị Board v0.1, mục 4 (HOA-2)

## Bối cảnh

Công ty outsource cho nhiều khách khác nhau. Thông tin, code, tên khách hàng của một Project bị mang sang Project khác là vi phạm bảo mật/NDA với khách.

## Phương án đã cân nhắc

- A: Tin tưởng agent tự phân biệt, không có quy tắc cứng.
- B: Quy tắc NDA cứng ở tầng Project + cấu hình cô lập session nếu nền tảng hỗ trợ.

## Quyết định

Chọn B:

1. Agent không được mang thông tin, code, tên khách hàng từ Project này sang Project khác.
2. Task luôn thuộc đúng một Project.
3. Nếu Paperclip cho phép reset/tách session theo Project cho dev agent → cấu hình như vậy; nếu không, dev agent không được dựa vào session memory cho kiến thức dự án (kiến thức nằm trong repo của Project, theo D-0003).
