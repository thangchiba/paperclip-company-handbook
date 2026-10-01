# D-0017 — Thiếu binding AWS chỉ chặn task có lệnh AWS

- **Ngày:** 2026-10-01
- **Phạm vi:** Toàn công ty (mọi agent)
- **Nguồn:** Board trả lời thẻ "QA thiếu binding AWS: có chặn HOA-268 không?" trên HOA-331 (chọn A); CTO đề xuất và thực hiện tại HOA-331.
- **Sửa đổi:** D-0012 mục 5 (chỉ áp dụng cho task cần chạy lệnh AWS)

## Bối cảnh

- D-0012 mục 5 và dòng cuối khối "AWS credentials (company rule, HOA-196)" trong `AGENTS.md` của 9 agent ghi: `No binding, or it fails: stop, report on the issue, set it blocked.` Câu này không nói rõ chỉ áp dụng khi task chạy lệnh AWS.
- QA chưa từng có binding AWS (chỉ CEO, CTO, InfraEngineer có). QA hiểu câu trên theo nghĩa đen nên dừng nghiệm thu Pro5 HOA-268 hai lần. HOA-268 không có lệnh AWS: `scripts/verify.sh`, E2E và smoke prod đều không cần credential, CTO đã chạy PASS trên `main`.
- Chỉ thị của CTO không gỡ được, vì QA coi rule trong `AGENTS.md` cao hơn chỉ thị của quản lý. Key `doraemon` assume được role admin của account Pro5 prod, nên không cấp cho agent không cần.

## Phương án đã cân nhắc

- A: Thiếu binding chỉ chặn task có lệnh AWS, sửa câu gốc cho cả 9 agent. CTO đề xuất. Board chọn.
- B: Bind key `doraemon` cho QA. QA giữ key assume được role admin prod dù HOA-268 không cần.
- C: Tạo IAM user read-only riêng cho QA: InfraEngineer viết Terraform, apply theo D-0011, mất khoảng nửa ngày.

## Quyết định

1. Thiếu binding AWS chỉ chặn task cần chạy lệnh AWS. Task không chạy lệnh AWS nào thì không cần binding, và thiếu binding không chặn task đó. Áp dụng ngay khi Board chọn, không chờ sửa `AGENTS.md`.
2. Tiêu chí nào thật sự cần dữ liệu AWS thì giao CTO chạy read-only.
3. Không cấp key cho agent không cần: QA không nhận binding `doraemon`.
4. Dòng cuối khối HOA-196 trong `AGENTS.md` đổi thành (nguyên văn):

   ```
   - No binding while the task needs an AWS command, or the check above fails: stop, report on the issue, set it blocked. Never look for other credentials. A task that runs no AWS command needs no binding; a missing binding does not block it.
   ```

## Hệ quả

- D-0012 mục 5: ghi chú theo D-0017. Mục 1–4 giữ nguyên: task chạy lệnh AWS vẫn chỉ dùng credential trong env, và lệnh đầu tiên vẫn là `sts get-caller-identity`.
- `AGENTS.md` của 9 agent: sửa dòng trên cùng đợt với khối HOA-136 (D-0015), lần tới khi có agent vào được host Paperclip. Theo dõi tại HOA-319.
- `handbook/rules.md` mục 4.1 chỉ dẫn chiếu D-0012, không cần sửa.
- QA tiếp tục HOA-268 mà không cần AWS.
