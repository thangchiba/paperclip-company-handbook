---
name: verify-and-report
description: Quy trình bắt buộc trước khi đánh done một task - tự verify kết quả và báo cáo theo khung ①②③. Dùng khi kết thúc bất kỳ task nào.
---

# verify-and-report

Áp dụng Definition of Done (`handbook/rules.md` mục 1) trước khi đóng task.

## Các bước

1. **Đối chiếu acceptance criteria** của task, từng dòng một. Thiếu dòng nào → chưa done.
2. **Tự verify bằng bằng chứng chạy được:**
   - Code: chạy test/lint/build liên quan, ghi lại lệnh + kết quả.
   - Tài liệu/nội dung: đọc lại bản cuối, kiểm tra link.
   - Hạ tầng: `terraform plan` sạch (không apply — theo `terraform-plan-only`).
3. **Đính bằng chứng vào task:** PR/commit link, work product, document, hoặc file upload. Đường dẫn local không tính là bằng chứng.
4. **Báo cáo theo khung ①②③** trong comment đóng task:
   - ① Kết quả + bằng chứng
   - ② Cần quyết / vướng mắc (hoặc "Không")
   - ③ Bước tiếp theo + người nhận
5. Việc phát sinh → tạo task mới có assignee, không để treo trong comment.

## Cấm

- Đánh `done` khi chưa verify hoặc chỉ có mô tả suông.
- Báo cáo "đã gửi/đã chạy" khi không xác nhận được kết quả ghi thành công.
