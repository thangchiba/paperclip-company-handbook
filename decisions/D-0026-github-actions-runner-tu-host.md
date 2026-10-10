# D-0026 — GitHub Actions: build, test, deploy chạy trên máy dev; GitHub-hosted chỉ báo tin

- **Ngày:** 2026-10-11
- **Phạm vi:** Mọi repo private của Hoang LLC có GitHub Actions.
- **Nguồn:** Chỉ thị Board ngày 2026-10-11 trong phiên Claude Code của Board.

## Bối cảnh

30 ngày tới 2026-10-11, repo `pro5` dùng khoảng 1.650 phút GitHub-hosted: app-ci ~1.150 (295 lượt), app-deploy ~430 (70), infra ~55 (66), deploy-notify ~10 (56).

## Lời Board

> "Có vẻ như github action hiện đang bị dùng khá nhiều. Tôi chủ yếu muốn action bắn notify sang telegram thôi chứ còn build hay các thứ thì gọi sang máy dev 192.168.1.111 để build cấu hình rất khoẻ. Thay đổi skill hay gì đó cho tôi nhé. chứ tốn khá nhiều tiền cho git"

## Quyết định

1. Job build, test, plan, deploy chạy trên runner tự host ở ThangChiba-Desktop (WSL, user `ghrunner`, label `win-dev`): `runs-on: [self-hosted, win-dev]`.
2. GitHub-hosted (`ubuntu-latest`) chỉ cho job báo tin nhẹ (Telegram), để vẫn báo khi máy dev tắt.
3. Đăng ký runner cho repo là việc của Board: `~/Workspace/LanServer/ci-runner/register.sh owner/repo N` trên MacbookServer; gỡ: `unregister.sh owner/repo`.
4. Quy tắc viết workflow: `pr-standard` mục 4.

## Hệ quả

- Máy dev tắt thì job build/deploy xếp hàng tới khi máy bật (GitHub huỷ sau 24 giờ).
- `pro5`: PR #135 chuyển app-ci, app-deploy, infra sang `win-dev`.
