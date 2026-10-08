# D-0025 — Figure: bật Meshy 3D thật trên prod, trần $20/tháng

- **Ngày:** 2026-10-08
- **Phạm vi:** Project Figure (repo `thangchiba/odeku`, prod `neokun.com`), chỉ phần model 3D do Meshy tạo. Trần ảnh 2D Gemini vẫn theo D-0014.
- **Nguồn:** Board trả lời card trên [HOA-463](/HOA/issues/HOA-463) (chọn A). FullstackDev đề xuất và thực hiện tại HOA-463. Lệnh gốc tại HOA-462.
- **Sửa đổi:** Không.

## Bối cảnh

- Board đã setup `MESHY_API_KEY` và giao triển khai code Meshy cho Odeku (HOA-462).
- Code Meshy đã có sẵn nhưng chưa có trần chi phí. HOA-463 thêm trần 3D riêng, cùng sổ chi với Gemini, mặc định $0.
- Mỗi model 3D tốn khoảng 30 credit (~$0,60).
- Theo D-0014, bật một provider trả phí khác (ví dụ Meshy) là quyết định chi phí mới, cần Board.

## Phương án đã cân nhắc

Câu hỏi trên card: «Bật Meshy (tạo model 3D thật) trên prod Odeku không, và trần chi phí bao nhiêu?»

- «A: bật, trần $20/tháng, $3/ngày + 1 task thử (đề xuất)»: Board chọn.
- «B: bật, trần $10/tháng, $2/ngày»
- «C: chưa bật, chỉ 1 task thử ~$0.60»

## Quyết định

1. **Bật Meshy 3D thật trên prod Figure**, dùng key Board đã setup.
   - Card này là lần Board duyệt việc sửa `.env` prod và deploy lại để đặt key và trần (D-0013 mục 2).
2. **Trần chi phí Meshy:** $20/tháng và $3/ngày, tính theo giờ JST.
3. **Một task thử** trên prod được phép, trong trần.

## Hệ quả

- Agent được tạo đơn thử bằng Meshy thật trên prod, trong trần, chỉ dùng ảnh test.
- Tăng trần Meshy là quyết định chi phí mới, cần Board.
