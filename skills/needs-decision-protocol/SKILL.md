---
name: needs-decision-protocol
description: Protocol không đoán - cách hỏi Board có cấu trúc (A/B/C + đề xuất), gắn needs-decision và chuyển việc khác thay vì chờ. Dùng khi thiếu thông tin để quyết định.
---

# needs-decision-protocol

Áp dụng `handbook/rules.md` mục 2 và D-0001. Không bao giờ đoán ý Board.

## Các bước

1. **Tra `decisions/` trước.** Có D-xxxx phù hợp → tự áp dụng, ghi "áp dụng theo D-xxxx". Không hỏi lại điều đã quyết.
2. Chưa có → soạn câu hỏi đúng format:
   - **Bối cảnh:** 1–2 dòng.
   - **Phương án:** A / B / C (tối đa 3, mỗi cái 1 dòng + trade-off chính).
   - **Đề xuất:** agent chọn phương án nào, vì sao.
3. Gửi dưới dạng interaction card (`ask_user_questions` hoặc `request_confirmation`) trên task liên quan, không hỏi trong prose.
4. **Gắn nhãn `needs-decision`** cho task bị vướng, đặt trạng thái chờ phù hợp, **chuyển sang việc khác ngay**. Hỏi không được chặn tiến độ.
5. Gom câu hỏi theo lô sáng/tối khi có nhiều câu — không hỏi lắt nhắt từng câu.
6. Board trả lời → ghi `decisions/D-xxxx-<slug>.md` trong cùng ngày (bối cảnh, phương án, quyết định, ngày, phạm vi).

## Cấm

- Tự bịa quyết định thay Board rồi làm tiếp.
- Hỏi câu không có phương án và đề xuất.
