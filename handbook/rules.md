# Quy tắc công ty — Hoang LLC

**Phiên bản: v0.4 (D-0021, Board duyệt 2026-10-02)**

Nguồn: Chỉ thị Board v0.1 (HOA-2) và D-0001 → D-0021. Sửa file này cần Board duyệt (D-0002). Agent không đọc file này mỗi run: quy tắc đến với agent qua khối "Decision authority" trong `AGENTS.md` và các skill dẫn dưới đây. Mỗi quy tắc chỉ nằm một chỗ; file này giữ phần không skill nào có và chỉ tới phần còn lại.

## 1. Definition of Done

Task chỉ `done` khi đủ cả 4:

1. Có kết quả kiểm chứng được (PR, file, document, báo cáo), link từ task.
2. Đã tự verify, ghi cách verify trong comment cuối.
3. Comment đóng task theo mục 5.
4. Không còn việc treo: bước lệnh Board phụ thuộc thành task con có người nhận; việc ngoài lệnh thành card `suggest_tasks` hoặc task `backlog` (D-0010).

Việc cần Board chỉ `done` sau khi Board đồng ý và hành động sau đó đã xong. Checklist: `concise-status-report`.

## 2. Không đoán ý Board

`operating-model` mục 2–3.

## 3. Ai quyết

`security-baseline`: mục 1 (danh sách việc Board quyết), mục 5 (merge, PR cho Board), mục 6 (hạ tầng), mục 7 (MacbookServer). Ngoài danh sách đó agent tự quyết và báo cáo (D-0001).

## 4. Secret, dữ liệu khách, nơi chạy code

`security-baseline` mục 2–4 và 8.

## 5. Báo cáo

Format `concise-status-report`. Khung ①②③ là: ① kết quả và bằng chứng, ② cần quyết hay vướng mắc, ③ bước tiếp theo.

## 6. Chi phí

1. Heartbeat timer tắt cho mọi agent (D-0007). Agent chỉ wake khi có việc: assign, comment, blocker xong, routine.
2. Không đặt trần chi phí per-agent (D-0008 mục 4).
3. Chạm 80% budget: chỉ làm task critical. Chạm 100%: auto-pause, báo Board.
4. Dịch vụ trả phí mới: Board duyệt, ghi chi phí ước tính mỗi tháng.

## 7. Mức tự chủ theo Project

Mỗi Project khai báo mức trong mô tả Project; chưa khai báo thì là `approve`.

| Mức | Ý nghĩa |
|---|---|
| `auto` | Agent tự làm trong phạm vi Project, báo cáo sau. |
| `notify` | Agent báo trước, chờ 12 giờ; Board không phản đối thì làm. |
| `approve` | Chờ Board duyệt tường minh. |

- Áp dụng cho deploy, đăng nội dung, gửi khách hàng.
- Merge code không theo mức này: `security-baseline` mục 5.
- Việc Board quyết ở `security-baseline` mục 1 không bị hạ bởi `auto` hay `notify`.

Lịch sử phiên bản: git log của repo.
