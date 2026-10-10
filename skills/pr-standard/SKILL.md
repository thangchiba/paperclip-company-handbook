---
name: pr-standard
description: Chuẩn branch, commit, PR, merge và GitHub Actions (runner tự host win-dev, GitHub-hosted chỉ để báo tin) của công ty - FullstackDev và InfraEngineer tự merge, việc quan trọng thì mở PR cho Board review với mô tả ngắn gọn theo format dưới, kiểm deploy sau merge. Dùng khi thay đổi code ở bất kỳ repo nào, và khi viết hay sửa workflow GitHub Actions.
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
- **Ban đêm (D-0024):** khi chế độ tự quyết bật (`operating-model` mục 1b), PR quan trọng của chính bạn đạt đủ điều kiện mục 1b.1 thì không gửi card: mở PR theo mục 1, merge, làm mục 3, rồi comment trên task dòng `Quyết thay ban đêm (D-0024): …`. PR chạm secret, auth, phân quyền, `SISYO.md`/`CLAUDE.md`, cổng kiểm hay đường deploy, hoặc thêm dịch vụ trả phí: vẫn chờ Board.

## 3. Sau khi merge

- **Figure:** merge vào odeku `main` là deploy production, tới khi có đơn thật (`security-baseline` mục 5). Người merge xem log deploy trên MacbookServer (runbook `docs/06_infra/deploy.md`, SSH theo `security-baseline` mục 7), kiểm `https://neokun.com/healthz` trả `ok` với đúng SHA ngắn của commit merge, rồi báo trên task.
- **Pro5:** merge vào `main` là deploy production, tới launch M1, trừ PR chỉ sửa `infra/**` hoặc `*.md`. Merge PR `infra/**` không apply gì; apply theo `terraform-plan-only`.

## 4. GitHub Actions (D-0026)

Phút GitHub Actions của repo private là tiền. Build, test, plan, deploy chạy trên máy dev qua runner tự host; GitHub-hosted chỉ để báo tin.
- Job build, test, lint, `verify.sh`, `terraform plan`, deploy: `runs-on: [self-hosted, win-dev]`.
- Chỉ job báo tin (Telegram, comment) được để `runs-on: ubuntu-latest`, và phải nhẹ: không `npm ci`, không build, `timeout-minutes` ≤ 5.
- Service container (vd DynamoDB Local) dùng cổng do Docker chọn (`ports: ["8000"]`), đọc lại bằng `${{ job.services.<tên>.ports['8000'] }}` ở `env` của step, để nhiều runner chạy song song.
- Runner `win-dev` không có sẵn công cụ như máy GitHub (Node hệ thống là v12): job cần runtime thì tự cài bằng action (`actions/setup-node`, `hashicorp/setup-terraform`…). Có sẵn: git, docker, aws, gh, jq, zip, python3.
- Runner giữ workspace giữa các lần chạy: không giả định thư mục sạch; dọn file tạm trong `$RUNNER_TEMP`.
- Repo chưa có runner `win-dev`: workflow mới vẫn viết `self-hosted`, và báo Board đăng ký runner (`~/Workspace/LanServer/ci-runner/register.sh owner/repo N` trên MacbookServer). Không tự đổi sang `ubuntu-latest` cho chạy được.
- Không thêm trigger chạy định kỳ (`schedule`) hay ma trận lớn khi Board chưa đồng ý.
- Sửa `.github/workflows/` là sửa đường deploy: theo mục 2, việc quan trọng.

## Cấm

- Merge khi check đỏ hay GitGuardian đỏ; chỉ Board đánh dấu false positive.
- Force-push branch chung, bỏ qua hook.
- Merge PR của việc quan trọng khi Board chưa đồng ý, trừ ban đêm đúng `operating-model` mục 1b (D-0024).
- Merge PR của agent khác.
