---
name: content-draft-protocol
description: Quy trình soạn nội dung marketing, blog, social, copy - luôn dừng ở bản draft, chỉ Board duyệt bản cuối trước khi đăng hay gửi. Dùng khi tạo bất kỳ nội dung hướng ra ngoài công ty.
---

# content-draft-protocol

Nội dung công khai đại diện cho công ty hoặc khách hàng: Board duyệt bản cuối trước khi đăng hay gửi (`security-baseline` mục 9), trừ Project khai báo mức `auto` cho việc đăng nội dung.

## Các bước

1. **Trước khi viết:** đối tượng đọc, mục tiêu (SEO, brand, bán hàng), kênh đăng, giọng điệu của Project.
2. **Soạn draft** trong repo hoặc document của đúng Project (NDA, D-0004): tiêu đề (2–3 phương án), nội dung, meta description hoặc caption theo kênh, hình ảnh đề xuất nếu có.
3. **Tự rà:** chính tả; claim có nguồn, không bịa số liệu, testimonial hay tính năng; không lộ thông tin khách, NDA hay secret.
4. **Xin duyệt:** card `human_only` trên task của mình (`operating-model` mục 3), kèm bản draft đầy đủ, nơi đăng và thời điểm đề xuất. MarketingManager review draft nhưng không duyệt đăng.
5. **Sau khi Board duyệt:** đăng đúng bản đã duyệt, ghi link vào task, báo cáo theo `concise-status-report`.

## Cấm

- Đăng hay gửi ra ngoài khi Board chưa duyệt bản cuối.
- Nhắc tên khách hàng hay dự án khác khi chưa được phép.
