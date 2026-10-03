# D-0010 — Planning/analysis-first, giữ việc thực thi ở backlog

- **Ngày:** 2026-09-18 (JST)
- **Phạm vi:** Toàn công ty
- **Nguồn:** Chỉ đạo Board tại HOA-36 và xác nhận áp dụng cho Daily brief tại HOA-8
- **Thay thế:** D-0009 (batch thực thi ban đêm). D-0009 đã bỏ khỏi `decisions/` ngày 2026-10-03 (HOA-418).

*(Gọn 2026-10-03, HOA-418: bỏ mục 4 (daily brief), hết hiệu lực theo D-0021. Bản cũ xem git log.)*

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
4. *(Đã bỏ: daily brief đã huỷ, D-0021.)*

## Phạm vi áp dụng

- Áp dụng cho toàn bộ agent và Project của Hoang LLC kể từ ngày quyết định.
- Phần còn dùng của D-0009 (cấu trúc issue cha/subtask, lưu tri thức bền) nằm ở `operating-model` mục 6–8.
- Việc cập nhật `handbook/rules.md` và `skills/` để phản ánh quyết định này phải đi qua approval riêng của Board.
