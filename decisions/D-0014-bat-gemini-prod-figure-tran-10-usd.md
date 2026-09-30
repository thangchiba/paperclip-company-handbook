# D-0014 — Figure: bật Gemini ảnh thật trên prod, trần $10/tháng

- **Ngày:** 2026-10-01
- **Phạm vi:** Project Figure (repo `thangchiba/odeku`, prod `neokun.com`), chỉ phần ảnh 2D do Gemini tạo. Không áp dụng cho model 3D.
- **Nguồn:** Board trả lời thẻ câu hỏi trên HOA-254 (chọn A); CEO đề xuất và thực hiện tại HOA-254; CTO làm phần kỹ thuật tại HOA-269.
- **Sửa đổi:** Không.

## Bối cảnh

Trên HOA-254, operator phàn nàn ảnh figure sau khi chọn trông "mờ".

- **Nguyên nhân:** prod đang chạy provider ảnh mock (`https://api.neokun.com/healthz` trả `providers.image = mock`). Mock làm mịn ảnh gốc bằng OpenCV, nên ảnh trông như bị làm mờ.
- **Provider thật đã có:** Gemini đã có trong code (`packages/figure_pipeline/.../adapters/gemini.py`). Ngày 21/09 đã chạy thử bằng key operator tạo; key hiện là secret `GEMINI_API_KEY` của Project Figure.
- **Chi phí:** smoke test đo được khoảng $0,034/ảnh với `gemini-3.1-flash-lite-image`.

Bật key trên prod tạo chi phí định kỳ, nên CEO hỏi Board.

## Phương án đã cân nhắc

- A: bật trên prod, trần $10/tháng. CEO đề xuất, Board chọn.
- B: chỉ bật cho người test.
- C: giữ mock trên prod.

## Quyết định

1. **Bật Gemini ảnh thật trên prod Figure**, dùng key operator tạo ngày 21/09.
   - Board duyệt luôn việc sửa `.env` prod và deploy lại để bật key và đặt trần.
   - Đây là deploy không qua merge, nên theo D-0013 mục 2 cần Board duyệt. Quyết định này là lần duyệt đó.
2. **Trần chi phí: $10/tháng** cho mọi lần gọi Gemini ảnh trên prod: tạo ảnh theo style, chat sửa ảnh, sửa model bằng prompt, và ảnh thử của agent.
3. **Cách giữ trần** (CEO chọn, thuộc thẩm quyền):
   - Trần cứng trong app, tính bằng USD theo số token Google báo trong response. Response không có số token thì tính theo bảng giá. Tháng và ngày tính theo giờ JST.
   - Mặc định $10/tháng và $2/ngày. Trần ngày để một ngày bị lạm dụng không ăn hết ngân sách cả tháng.
   - Chạm trần: dùng mock có nhãn ảnh mẫu, báo khách bằng một câu nhẹ nhàng, ghi cảnh báo cho staff.
   - Key chỉ được bật sau khi code trần đã chạy trên prod.
4. **Key phải ở gói trả phí.**
   - Chính sách riêng tư hứa 「お客様の写真を、AIモデルの学習に利用することはありません」.
   - Gói miễn phí của Gemini API cho phép Google dùng dữ liệu để cải thiện sản phẩm.
   - Vì vậy, không xác nhận được key ở gói trả phí thì không bật trên prod.
5. **Không đổi:** model 3D vẫn mock (chưa có key Meshy); Stripe vẫn mock.

## Hệ quả

- CTO làm phần kỹ thuật trong HOA-269 (wave 1 của HOA-254): kiểm giá thật, kiểm gói trả phí, làm trần trong app, chọn model, bật key, thử trên prod.
- HOA-263 (chọn style, chat sửa ảnh) và HOA-266 (sửa model bằng prompt) gọi Gemini qua trần này, không làm trần riêng.
- Agent được tạo vài đơn thử bằng Gemini thật trên prod, trong trần, và chỉ dùng ảnh test.
- Tăng trần, hoặc bật provider trả phí khác (ví dụ Meshy), là quyết định chi phí mới, cần Board.
- Trước khi Figure nhận đơn thật, xem lại cách xử lý khi chạm trần: trả ảnh mock cho khách thật là không phù hợp. Việc này đưa vào quyết định go-live, cùng với việc chuyển sang phương án B của D-0013.
