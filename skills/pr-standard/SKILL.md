---
name: pr-standard
description: Chuẩn branch, commit, PR và merge của công ty - FullstackDev và InfraEngineer tự merge, việc quan trọng thì mở PR cho Board review với mô tả ngắn gọn theo format dưới, kiểm deploy sau merge. Dùng khi thay đổi code ở bất kỳ repo nào.
---

# pr-standard

Lời Board (2026-10-02, D-0021): «Dev và Infra tự do merge code vào. Chỉ những task quan trọng thì phát hành PR để tôi review. Lưu ý khi phát hành PR cần trình bày ngắn gọn súc tích như format tôi đã yêu cầu lần trước.»

## 1. Branch, commit, PR

- Branch `feat/<slug>`, `fix/<slug>`, `chore/<slug>`. Không commit thẳng vào `main`, trừ repo Board cho phép rõ ràng.
- PR nhỏ, một mục đích; việc lớn tách nhiều PR theo thứ tự. Không trộn refactor lớn vào PR sửa bug.
- Commit message nói "vì sao", thêm dòng `Co-Authored-By: Paperclip <noreply@paperclip.ing>`.
- Mô tả PR, đọc xong trong 30 giây:
  - **Summary:** 1–3 bullet.
  - **Test plan:** lệnh đã chạy + kết quả.
  - Link task Paperclip.
- Trước khi merge: đọc lại diff, chạy test/lint/build liên quan (`dev-machine`), không file thừa, không secret trong diff (`secret-hygiene`).

## 2. Merge

- **Việc thường:** FullstackDev và InfraEngineer tự merge PR của mình khi check xanh và GitGuardian xanh. Không cần review của agent khác, không chờ QA.
  - Figure (odeku, không có CI): check xanh = `scripts/verify.sh` PASS trên head của PR.
  - Pro5: mọi check trên PR xanh, gồm `plan` khi PR chạm `infra/**`.
- **Việc quan trọng** (thuộc danh sách `security-baseline` mục 1, kể cả sửa `SISYO.md`/`CLAUDE.md`): không tự merge.
  1. Mở PR theo format mục 1. Bullet đầu của Summary nêu vì sao cần Board và Board cần xem gì.
  2. Hỏi Board bằng card `request_confirmation` `human_only` trên task, kèm link PR (`operating-model` mục 3). Mô tả card theo `concise-status-report`.
  3. Merge sau khi Board chấp nhận card hoặc đồng ý bằng comment. Board yêu cầu sửa: sửa rồi hỏi lại bằng card mới.
- Không chắc việc có quan trọng không: coi là quan trọng.

## 3. Sau khi merge

- **Figure:** merge vào odeku `main` là deploy production, tới khi có đơn thật (`security-baseline` mục 5). Người merge xem log deploy trên MacbookServer (runbook `docs/06_infra/deploy.md`, SSH theo `security-baseline` mục 7), kiểm `https://neokun.com/healthz` trả `ok` với đúng SHA ngắn của commit merge, rồi báo trên task.
- **Pro5:** merge vào `main` là deploy production, tới launch M1, trừ PR chỉ sửa `infra/**` hoặc `*.md`. Merge PR `infra/**` không apply gì; apply theo `terraform-plan-only`.

## Cấm

- Merge khi check đỏ hay GitGuardian đỏ; chỉ Board đánh dấu false positive.
- Force-push branch chung, bỏ qua hook.
- Merge PR của việc quan trọng khi Board chưa đồng ý.
