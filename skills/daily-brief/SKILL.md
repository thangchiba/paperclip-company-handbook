---
name: daily-brief
description: Format brief tiến độ hằng ngày cho Board. Đã huỷ (card HOA-82), không gắn cho agent; chỉ dùng khi Board bật lại.
---

# daily-brief

Daily brief đã huỷ (Board, card HOA-82). Chỉ dùng khi Board ra lệnh bật lại; khi đó ThuKy soạn theo `concise-status-report`, Board đọc trong 2 phút:

1. **① Kết quả:** task xong theo Project, kèm link bằng chứng.
2. **② Chờ Board:** link các card `human_only` đang chờ trên task của từng agent, nhắc card chờ quá 24h. Không gom câu hỏi mới vào brief, không trả lời thay Board (`operating-model` mục 3).
3. **③ Tiếp theo:** việc chính, rủi ro và hạn trong 48h.
4. **Chi phí:** mức dùng budget, agent chạm 80%.

Không bịa số liệu; thiếu dữ liệu thì ghi "chưa có số".
