# D-0013 — Figure và Pro5: agent tự merge PR khi đủ điều kiện (`auto`)

- **Ngày:** 2026-09-28
- **Phạm vi:** Project Figure (repo `thangchiba/odeku`) và Project Pro5 (repo `thangchiba/pro5`)
- **Nguồn:** Board trả lời card trên HOA-211; CTO đề xuất trên HOA-207; CEO thực hiện tại HOA-211
- **Sửa đổi:** Không. Đây là khai báo mức tự chủ theo `handbook/rules.md` mục 7.
- **Cập nhật:** 2026-09-28, Pro5: lần deploy do merge kéo theo tính vào mức `auto`, tới trước launch M1 (Board trả lời card trên HOA-216). Xem mục "Cập nhật 2026-09-28 — Pro5".
- **Cập nhật:** 2026-09-28, Figure: lần deploy do merge vào odeku `main` kéo theo tính vào mức `auto`, tới trước khi có đơn thật (Board trả lời card trên HOA-215). Xem mục "Cập nhật 2026-09-28 — Figure".

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
   - Deploy production, đăng nội dung công khai, gửi khách hàng. Riêng Pro5: lần deploy do merge kéo theo tính vào mức `auto`, tới trước launch M1 (mục "Cập nhật 2026-09-28 — Pro5"). Riêng Figure: lần deploy do merge vào odeku `main` kéo theo tính vào mức `auto`, tới trước khi có đơn thật (mục "Cập nhật 2026-09-28 — Figure").
   - Mọi hàng "Board duyệt" cứng ở rules.md mục 3, kể cả sửa file hướng dẫn agent trong repo (`SISYO.md`, `CLAUDE.md`). PR chạm các phần này vẫn cần card Board riêng (vd odeku #9).
   - Pro5: apply hạ tầng theo D-0011 (Board duyệt từng lần apply). Merge PR `infra/**` không apply gì.
3. **Merge không kéo theo deploy.** Mức `auto` dựa trên thực tế lúc quyết: merge không deploy (odeku không có GitHub Actions; Pro5 chỉ có CI chạy `plan`). Vì deploy production vẫn cần Board duyệt, PR nào làm merge tự deploy production (vd cài `.github/workflows/app-deploy.yml` của Pro5, hoặc thêm workflow deploy cho odeku) thì cần card Board, và CEO hỏi lại Board mức merge của Project đó trước khi cài. Pro5 đã hỏi lại trên HOA-216, xem mục "Cập nhật 2026-09-28 — Pro5". Với odeku, thực tế "merge không deploy" sai ngay từ đầu vì có webhook trên MacbookServer; Board chọn lại trên HOA-215, xem mục "Cập nhật 2026-09-28 — Figure".

## Cách áp dụng điều kiện a

Theo đề xuất của CTO trên HOA-207. Hai điểm này chỉ làm chặt thêm, không nới điều kiện Board chọn:

- "Người viết" xác định theo Paperclip, không theo GitHub: mọi agent push bằng cùng tài khoản `thangchiba`, nên GitHub không phân biệt được. Người viết là agent đã làm issue Paperclip của PR và push commit, xem theo lịch sử issue (lúc review, issue có thể đã chuyển cho reviewer).
- Reviewer tự push commit sửa vào PR thì thành đồng tác giả. Người viết gốc hoặc một agent thứ ba phải xem các commit đó trước khi merge.

## Hệ quả

- Mô tả Project Figure và Pro5 trên Paperclip ghi mức theo quyết định này.
- CTO được báo trên HOA-207: các PR odeku đang chờ merge theo mức mới khi đủ điều kiện.
- `handbook/rules.md` không đổi: mục 7 đã cho mỗi Project tự khai báo mức. Nếu thêm Project dùng cùng mẫu (merge `auto`; deploy, nội dung, khách hàng `approve`), CEO đề xuất đưa mẫu này vào mục 7 ở buổi rà soát Chủ nhật (D-0002).

## Cập nhật 2026-09-28 — Pro5: merge kéo theo deploy (HOA-216)

- **Nguồn:** Board trả lời card trên HOA-216; CTO nhắc trên HOA-208 sau khi merge pro5 #8; CEO thực hiện tại HOA-216.

### Bối cảnh

Bước 3 của HOA-135 chép `ci/app-deploy.yml` vào `.github/workflows/`. Từ đó mỗi push vào `main` của `thangchiba/pro5` tự deploy production, trừ push chỉ sửa `infra/**` hoặc `*.md`. Workflow có sẵn trigger chạy tay (`workflow_dispatch`); agent chỉ đọc được Actions, không tự chạy hay sửa được file workflow. Theo mục 3 ở trên, CEO hỏi lại Board trước khi cài. Lúc hỏi, prod chưa có user thật: Stripe dùng test key, SES còn sandbox, site đang `noindex`.

### Phương án đã cân nhắc

- A: giữ deploy theo push, merge = deploy, tới trước launch M1. Board chọn.
- B: bỏ trigger `push`, chỉ deploy bằng tay: agent báo SHA, Board bấm chạy `app-deploy` trên `main`. CEO đề xuất.
- C: giữ deploy theo push, Board duyệt từng PR code app bằng card.

### Quyết định

- **Pro5 cài `app-deploy` như hiện có**, không sửa trigger.
- **Mức `auto` của Pro5 tính luôn lần deploy production mà merge kéo theo.** Điều kiện merge vẫn là a–c của mục 1 ở trên; không thêm card cho PR code app. Kiểm soát bù: CTO review mọi PR, và mỗi lần deploy có email cảnh báo (HOA-148). Rủi ro Board chấp nhận: PR code app nào merge cũng lên prod ngay, Board không xem trước.
- **Chỉ tới trước launch M1.** Trước khi bỏ `noindex` (HOA-194), Pro5 chuyển sang phương án B. Từ đó merge không deploy nữa, và deploy production lại cần Board duyệt như mục 2 ở trên.
- **Không đổi:** Figure có mục cập nhật riêng ở dưới (HOA-215). Với Pro5, phần còn lại của mục 2 ở trên giữ nguyên, kể cả apply hạ tầng theo D-0011.

### Hệ quả

- Mô tả Project Pro5 trên Paperclip ghi mức mới.
- Bước 3 của HOA-135 làm theo mô tả hiện có, không chờ PR sửa workflow.
- HOA-194 (go-live M1) có thêm bước chuyển sang phương án B, làm trước khi bỏ `noindex`.
- CTO được báo trên HOA-216.

## Cập nhật 2026-09-28 — Figure: merge kéo theo deploy (HOA-215)

- **Nguồn:** Board trả lời card trên HOA-215; CTO phát hiện và đề xuất trên HOA-215; CEO thực hiện tại HOA-215.

### Bối cảnh

Mục 3 ở trên cho rằng merge odeku không deploy vì odeku không có GitHub Actions. Điều này sai. Trên MacbookServer có webhook `com.hoang.deploy-webhook` (launchd), gắn nhánh `main` của `thangchiba/odeku` với `deploy-odeku.sh`. Script này pull, build rồi `compose up`, khoảng 4 giây sau mỗi push vào `main`. Nguồn: `deploy/macbookserver/deploy-odeku.sh` và runbook `docs/06_infra/deploy.md` trong repo (HOA-171, HOA-185). Ngày 28/09 cả 8 lần push vào `main` đều deploy. Lúc hỏi, `https://neokun.com/healthz` trả `d01120a` (PR #18). Như vậy merge `auto` là deploy mà Board không duyệt, nên CTO giữ merge #17, #11, #16 tới khi Board chọn. Lúc hỏi, Figure chưa có đơn thật và Stripe còn mock.

### Phương án đã cân nhắc

- A: giữ webhook, merge = deploy, tới trước khi có đơn thật. Board chọn.
- B: webhook theo nhánh `release` thay cho `main`. Merge vào `main` vẫn `auto`. Deploy = fast-forward `release` lên một SHA của `main`, Board duyệt bằng card; một card gom được nhiều merge. CTO và CEO đề xuất.
- C: Board duyệt từng PR odeku bằng card, như trước D-0013.

### Quyết định

- **Giữ webhook**, không sửa webhook hay `deploy-odeku.sh`. Mỗi push vào odeku `main` vẫn tự deploy production.
- **Mức `auto` của Figure tính luôn lần deploy production mà merge kéo theo.** Điều kiện merge vẫn là a–c của mục 1 ở trên. Rủi ro Board chấp nhận:
  - Mỗi PR, kể cả PR chỉ sửa docs, đều build lại và restart prod.
  - PR sửa nội dung web lên site ngay, Board không xem trước.
- **Người merge kiểm deploy rồi báo sau.** Sau mỗi merge:
  - Xem log deploy trên MacbookServer (runbook `docs/06_infra/deploy.md`).
  - Kiểm `https://neokun.com/healthz` trả `ok` với đúng SHA ngắn của commit merge.
  - Báo theo khung ①②③ trên issue của PR: SHA đang chạy, deploy OK hay lỗi.
- **Chỉ tới trước khi có đơn thật.** Trước khi nhận đơn thật, Figure chuyển sang phương án B. Từ đó merge không deploy nữa, và deploy production lại cần Board duyệt như mục 2 ở trên.
- **Không đổi:** deploy production không qua merge vào `main` vẫn cần Board duyệt. Phần còn lại của mục 2 ở trên giữ nguyên với Figure.

### Hệ quả

- Mô tả Project Figure trên Paperclip ghi mức mới, thay cho dòng tạm dừng merge odeku.
- CTO merge tiếp #17, #11, #16 theo HOA-214 khi đủ điều kiện, và làm bước kiểm deploy ở trên sau mỗi lần merge.
- Khi Figure chuẩn bị nhận đơn thật, CEO đưa việc chuyển sang B vào quyết định go-live. CTO làm phần kỹ thuật của B:
  - Sửa 1 dòng `config.json` của webhook, rồi restart webhook.
  - Mở PR đổi `deploy-odeku.sh` sang pull `release`.
  - Tạo nhánh `release` tại commit đang chạy trên prod.
