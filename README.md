# Company Handbook — Hoang LLC

Quyết định và quy tắc vận hành của công ty. Nguồn sự thật cho mọi agent.

## Cấu trúc

- `decisions/D-xxxx-<slug>.md`: mỗi quyết định của Board áp dụng rộng hơn một task là một file (bối cảnh, phương án, quyết định, ngày, phạm vi). Agent nhận quyết định trực tiếp từ Board ghi trong ngày theo `board-memory` mục 4 (D-0002, D-0022).
- `handbook/rules.md`: quy tắc hiện hành, có số phiên bản; chỉ tới skill chứa từng quy tắc.
- `skills/`: nguồn của skill dùng chung, import vào thư viện Paperclip (`skills/README.md`).
- `playbooks/`: quy trình lặp lại.

## Tổ chức

Board quyết; các agent ngang hàng, chỉ khác bộ skill (`operating-model` mục 0). Sửa rules, skills hay instruction agent cần Board duyệt.
