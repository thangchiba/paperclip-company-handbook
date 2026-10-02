# D-0019 — Figure: hàng RAW gửi trong 5営業日 sau khi thanh toán

- **Ngày:** 2026-10-02
- **Phạm vi:** Project Figure (repo `thangchiba/odeku`, prod `neokun.com`), hạn gửi hàng RAW (未塗装) mà khách thấy: 特商法表記「商品の引渡時期」, màn xác nhận thanh toán, FAQ trang chủ. Hàng tô màu (彩色済み) không đổi.
- **Nguồn:** Board trả lời thẻ câu hỏi trên HOA-372 (chọn B). CTO phát hiện khi review PR #41 (HOA-262, finding 6 ở HOA-364) và đề xuất B; CEO thực hiện tại HOA-372; CTO làm phần code tại HOA-381.
- **Sửa đổi:** Có. Thay hạn RAW 3営業日 mà Board chốt ở HOA-165 (2026-09-25).

## Bối cảnh

- 特商法表記 ghi 「お支払い確認後、未塗装（RAW）商品は3営業日以内、彩色済み商品は3週間以内に発送します」. Con số 3営業日 Board chốt ở HOA-165 với ý "in thô 3 ngày". RAW là SKU duy nhất đang bán (HOA-173).
- PR #41 (HOA-262, merge `2d5c1ec` lúc 2026-10-02 04:36 JST) thêm bước staff kiểm tra in được sau khi khách thanh toán: 「スタッフが印刷可能か確認し、2営業日以内にご連絡します」. Câu này nằm trên cùng màn xác nhận thanh toán với hạn 3営業日.
- Nếu staff dùng hết 2 ngày kiểm tra, chỉ còn 1 ngày để in, hậu kỳ và gửi. Gửi trễ thì 特商法表記 sai với thực tế: rủi ro pháp lý và khiếu nại.
- Lúc hỏi, prod chưa có đơn thật, nên đổi văn bản không ảnh hưởng khách nào.

## Phương án đã cân nhắc

- A: giữ 3営業日; staff cố kiểm tra xong trong ngày. Không sửa văn bản, nhưng ngày đông đơn dễ gửi trễ.
- B: đổi RAW thành 5営業日 = 2 ngày kiểm tra + 3 ngày in như HOA-165. CTO và CEO đề xuất, Board chọn.
- C: rút bước kiểm tra còn 1営業日, giữ 3営業日. Staff chịu áp lực hơn; ngày đông đơn vẫn dễ trễ.

## Quyết định

1. **Hàng RAW gửi trong 5営業日 sau khi xác nhận thanh toán.** Văn bản mới: 「未塗装（RAW）商品は5営業日以内、彩色済み商品は3週間以内」.
2. **Không đổi:**
   - Staff kiểm tra in được trong 2営業日 (HOA-262).
   - Hàng tô màu gửi trong 3週間; miễn phí ship (HOA-165).

## Hệ quả

- CTO làm phần code tại HOA-381:
  - Sửa `business.leadTime` trong `apps/web/src/app/(site)/legal/business.ts`. Hằng số này hiện ở 特商法表記, màn xác nhận thanh toán và FAQ.
  - Hạn gửi là văn bản khách đồng ý lúc thanh toán, nên bump `CONSENT_TEXT_VERSION` và 制定日 (`effectiveDate`), sửa test và docs.
  - Merge theo D-0013. Với Figure, merge vào `main` là deploy prod.
- PR #41 đã merge trước khi Board chọn, nên đây là PR riêng và `CONSENT_TEXT_VERSION` bump thêm một lần.
- HOA-372 đóng sau khi prod hiện 5営業日.
