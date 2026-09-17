---
name: operating-model
description: Mô hình vận hành Hoang LLC - planning-first ban ngày, thực thi batch ban đêm, session hygiene, khởi tạo dự án. Dùng khi lên kế hoạch, nhận task thực thi, delegate, hoặc bắt đầu dự án mới.
---

# operating-model

Thoả thuận vận hành toàn công ty theo chỉ thị Board (D-0009). Áp dụng cho mọi agent.

## 1. Planning-first — thực thi ban đêm

- **Ban ngày = nghiên cứu + kế hoạch. Ban đêm (~02:00 JST) = thực thi.**
- Trong giai đoạn planning (đặc biệt ProductManager, dự án mới): **không code, không delegate implementation, không thực thi ngay**. Deliverable là nghiên cứu + kế hoạch triển khai chi tiết.
- Output planning phải: tóm tắt rõ ràng; mọi điểm chưa chắc chắn hỏi Board qua card `ask_user_questions` (A/B/C + đề xuất kèm lý do). Không đoán (theo `needs-decision-protocol`).
- Việc thực thi xếp hàng dạng issue `todo`, để run batch ban đêm xử lý — không tạo run lẻ ban ngày.

## 2. Engineer — protocol batch ban đêm

- Một run đêm xử lý **lần lượt mọi issue `todo`** được assign, theo ưu tiên, đến khi hết hoặc chạm budget.
- **Mỗi issue một comment kết quả.** Không mở session riêng cho từng task nhỏ.

## 3. Việc lớn nhiều bước → issue cha + subtask

- Tạo một issue cha; mỗi bước là subtask. Agent giữ session/workspace của issue cha thay vì mở session mới cho từng mảnh.

## 4. Session hygiene

- ChiefOfStaff: reset session tối Chủ nhật, sau weekly review.
- Engineer: reset khi đổi project hoặc khi thấy run rỗng.
- Cái gì cần "nhớ" thì để trong file (handbook, docs repo, skill) — **không dựa vào transcript**.

## 5. Stable → skill

- Kiến thức ổn định để trong skill (chỉ nạp khi cần), không để trong instruction của agent (nạp mọi run). Giữ instruction agent tối giản.

## 6. Hạn chế chatter giữa agent

- Tránh @mention và comment qua lại giữa agent — mỗi mention là một run tốn budget. Chỉ mention khi người đó phải hành động. Ưu tiên child issue với mô tả tự chứa.

## 7. Khởi tạo dự án

- Mọi dự án mới khởi tạo bằng bộ skill **sisyo**: chạy `npx sisyo` tại project root (https://www.npmjs.com/package/sisyo).
- **Trước khi chọn bất kỳ tech stack nào** (DB, backend, FE, infra, ...) phải hỏi Board qua `ask_user_questions` (A/B/C + đề xuất). Không tự quyết.
