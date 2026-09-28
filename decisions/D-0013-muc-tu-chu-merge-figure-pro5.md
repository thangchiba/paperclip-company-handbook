# D-0013 — Figure và Pro5: agent tự merge PR khi đủ điều kiện (`auto`)

- **Ngày:** 2026-09-28
- **Phạm vi:** Project Figure (repo `thangchiba/odeku`) và Project Pro5 (repo `thangchiba/pro5`)
- **Nguồn:** Board trả lời card trên HOA-211; CTO đề xuất trên HOA-207; CEO thực hiện tại HOA-211
- **Sửa đổi:** Không. Đây là khai báo mức tự chủ theo `handbook/rules.md` mục 7.

## Bối cảnh

Figure và Pro5 chưa khai báo mức tự chủ, nên theo rules.md mục 7 mặc định là `approve`: mỗi lần merge PR cần Board duyệt bằng card. Thực tế CTO đã tự merge sau khi review (odeku #1, #2, #14; các PR Pro5 cũng vậy). Lúc hỏi, odeku có 4 PR chờ merge (#11, #16, #17, #18); giữ `approve` nghĩa là thêm 4 card cho Board.

## Phương án đã cân nhắc

Figure:

- A: `auto` cho merge PR, khi đủ 3 điều kiện ở dưới. Board chọn.
- B: `notify`: báo trước mỗi lần merge, chờ 12h, Board không phản đối thì merge.
- C: giữ `approve`: Board duyệt từng PR bằng card.

Pro5:

- A: cùng mức với Figure. Board chọn.
- B: giữ `approve`.
- C: để sau, CEO hỏi khi có PR Pro5 tiếp theo.

## Quyết định

1. **Figure và Pro5: mức `auto` cho merge PR vào `main`.** Agent tự merge, Board không duyệt từng PR; agent báo cáo sau theo khung ①②③. Áp dụng cho PR code, test và tài liệu trong repo. Chỉ merge khi đủ cả 3 điều kiện:
   - a. Có review của một agent khác người viết PR (thường là CTO).
   - b. Kiểm tra xanh. Figure: `scripts/verify.sh` PASS trên head của PR (odeku không có CI). Pro5: mọi check trên PR xanh (gồm `plan` khi PR chạm `infra/**`).
   - c. GitGuardian xanh.
2. **Vẫn cần Board duyệt, không đổi:**
   - Deploy production, đăng nội dung công khai, gửi khách hàng.
   - Mọi hàng "Board duyệt" cứng ở rules.md mục 3, kể cả sửa file hướng dẫn agent trong repo (`SISYO.md`, `CLAUDE.md`). PR chạm các phần này vẫn cần card Board riêng (vd odeku #9).
   - Pro5: apply hạ tầng theo D-0011 (Board duyệt từng lần apply). Merge PR `infra/**` không apply gì.
3. **Merge không kéo theo deploy.** Mức `auto` dựa trên thực tế lúc quyết: merge không deploy (odeku không có GitHub Actions; Pro5 chỉ có CI chạy `plan`). Vì deploy production vẫn cần Board duyệt, PR nào làm merge tự deploy production (vd cài `.github/workflows/app-deploy.yml` của Pro5, hoặc thêm workflow deploy cho odeku) thì cần card Board, và CEO hỏi lại Board mức merge của Project đó trước khi cài.

## Cách áp dụng điều kiện a

Theo đề xuất của CTO trên HOA-207. Hai điểm này chỉ làm chặt thêm, không nới điều kiện Board chọn:

- "Người viết" là agent được giao issue Paperclip của PR. Mọi agent push bằng cùng tài khoản GitHub `thangchiba`, nên GitHub không phân biệt được.
- Reviewer tự push commit sửa vào PR thì thành đồng tác giả. Người viết gốc hoặc một agent thứ ba phải xem các commit đó trước khi merge.

## Hệ quả

- Mô tả Project Figure và Pro5 trên Paperclip ghi mức theo quyết định này.
- CTO được báo trên HOA-207: các PR odeku đang chờ merge theo mức mới khi đủ điều kiện.
- `handbook/rules.md` không đổi: mục 7 đã cho mỗi Project tự khai báo mức. Nếu thêm Project dùng cùng mẫu (merge `auto`; deploy, nội dung, khách hàng `approve`), CEO đề xuất đưa mẫu này vào mục 7 ở buổi rà soát Chủ nhật (D-0002).
