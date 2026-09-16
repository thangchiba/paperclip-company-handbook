---
name: pr-standard
description: Chuẩn mở pull request của công ty - branch, commit, mô tả PR, self-review và điều kiện merge theo mức tự chủ Project. Dùng khi thay đổi code bất kỳ repo nào.
---

# pr-standard

Mọi thay đổi code đi qua PR. Không commit thẳng vào main (trừ repo được Board cho phép rõ ràng).

## Quy tắc

1. **Branch:** `feat/<slug>`, `fix/<slug>`, `chore/<slug>` — ngắn, mô tả đúng việc.
2. **Commit:** message tập trung "vì sao", thêm dòng `Co-Authored-By: Paperclip <noreply@paperclip.ing>`.
3. **PR description** gồm:
   - Summary: 1–3 bullet.
   - Test plan: đã verify bằng gì (lệnh + kết quả).
   - Link task Paperclip liên quan.
4. **Self-review trước khi xin review:** đọc lại diff, chạy test/lint, không để file thừa, không secret trong diff.
5. **Merge theo mức tự chủ Project** (`handbook/rules.md` mục 7): `auto` → tự merge sau khi xanh; `notify` → báo trước, chờ khoảng đệm; `approve` (mặc định) → chờ Board duyệt.
6. PR nhỏ, một mục đích. Việc lớn → tách nhiều PR theo thứ tự.

## Cấm

- Force-push branch chung, skip hook, merge khi CI đỏ.
- Trộn refactor lớn vào PR fix bug.
