# D-0027 — Máy chủ prod chuyển từ MacbookServer sang ubuntu1 (192.168.1.97)

- **Ngày:** 2026-10-15
- **Phạm vi:** Mọi dịch vụ trước đây chạy trên MacbookServer (192.168.1.95): Paperclip, Hindsight, Keycloak Hoang Auth, Odeku/Figure, Yomasete, SiriSugi, FIRE, LenS, Umami, content-agent, DB dùng chung, monitoring, tunnel Cloudflare, deploy webhook.
- **Nguồn:** Chỉ thị Board ngày 2026-10-15 trong phiên Claude Code của Board.

## Bối cảnh

Board có máy mới (Intel i5-12400) chạy Proxmox (host `hoangserver1`, 192.168.1.96). Trên đó là VM Ubuntu 26.04 `ubuntu1` (192.168.1.97, amd64, 12 core, 12 GB RAM). Cutover xong tối 2026-10-15: cả hai tunnel Cloudflare chạy trên ubuntu1, mọi domain trỏ về đó.

## Lời Board

> "Tôi có 1 máy chạy server mới, 12400 CPU. Tôi dự kiến chuyển server chạy từ mac qua bên đó cho khoẻ."

> "Tôi muốn bạn migrate hết sang… Lưu ý data chỉ copy nhé. chưa xoá bên mac."

> "Cut ok."

## Quyết định

1. Máy chủ prod là **ubuntu1** (`ssh thangchiba@192.168.1.97`, `hostname` ra `ubuntu1`). Cùng đường dẫn như trên Mac: `~/Workspace/Hoang/...`, `~/Workspace/LanServer/...`, `~/srv/...`.
2. MacbookServer (192.168.1.95) không còn là prod. Các stack trên đó đã dừng và giữ làm bản dự phòng để Board rollback. Agent không SSH, không deploy, không sửa gì trên Mac.
3. Quyền SSH vào máy chủ prod theo D-0018/D-0021 (InfraEngineer, FullstackDev, ChiefOfStaff) áp dụng cho ubuntu1 với cùng điều kiện (`security-baseline` mục 7).
4. ubuntu1 là amd64 như ThangChiba-Desktop. Deploy webhook build image ngay trên ubuntu1 sau khi merge.
5. Hindsight: `http://192.168.1.97:26352`.

## Hệ quả

- Runbook hay docs của dự án còn ghi MacbookServer hoặc 192.168.1.95 là cũ; làm theo skill, sửa docs qua PR khi đụng tới.
- Quy tắc cũ "không đưa image amd64 lên Mac" hết hiệu lực cho prod.
- Code, build, test của dự án vẫn chỉ chạy trên ThangChiba-Desktop (`security-baseline` mục 2), không chạy trên ubuntu1.
