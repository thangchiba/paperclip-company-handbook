# D-0010 — Planning/analysis-first, giữ việc thực thi ở backlog

- **Ngày:** 2026-09-18 (JST)
- **Phạm vi:** Toàn công ty
- **Nguồn:** Chỉ đạo Board tại HOA-36 và xác nhận áp dụng cho Daily brief tại HOA-8
- **Thay thế:** Phần “Engineer chạy batch ban đêm” và phần tự động dồn thực thi vào ban đêm của D-0009

## Bối cảnh

Board định hướng lại dàn agent: trọng tâm là lập kế hoạch, phân tích thị trường và tư vấn chuyên môn để các dự án luôn tiến triển. Việc thực thi không tự chạy theo hàng đợi; user sẽ chủ động kéo task về làm hoặc yêu cầu automation khi cần.

## Phương án đã cân nhắc

1. **Giữ mô hình D-0009:** agent tự chạy batch thực thi ban đêm.
2. **Planning/analysis-first:** agent chuẩn bị nghiên cứu, kế hoạch và task backlog; user chủ động chọn phần cần triển khai hoặc yêu cầu automation.

## Quyết định

Chọn phương án 2:

1. Hoạt động định kỳ ưu tiên phân tích thị trường, lập kế hoạch triển khai tiếp theo, lập lịch và đưa khuyến nghị theo chuyên môn từng vai trò.
2. Có thể tạo sẵn task thực thi, nhưng giữ ở trạng thái `backlog`; không tự động triển khai.
3. User chủ động kéo task về thực hiện hoặc yêu cầu automation sau.
4. Daily brief buổi sáng nêu kế hoạch tiếp theo theo từng Project và hỏi Board phần nào nên chuyển sang triển khai.

## Phạm vi áp dụng

- Áp dụng cho toàn bộ agent và Project của Hoang LLC kể từ ngày quyết định.
- Các phần khác của D-0009 (cấu trúc issue cha/subtask, lưu tri thức bền, giới hạn mention) vẫn có hiệu lực.
- Việc cập nhật `handbook/rules.md` và `skills/` để phản ánh quyết định này phải đi qua approval riêng của Board.
