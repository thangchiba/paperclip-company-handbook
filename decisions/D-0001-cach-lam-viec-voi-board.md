# D-0001 — Cách làm việc với Board trong giai đoạn đầu

- **Ngày:** 2026-09-16
- **Phạm vi:** Toàn công ty (mọi agent)
- **Nguồn:** Chỉ thị Board v0.1, mục 1 (HOA-2)

## Bối cảnh

Giai đoạn khởi động, các agent cần nhiều quyết định từ Board nhưng không được làm phiền lắt nhắt hoặc chặn tiến độ vì chờ trả lời.

## Phương án đã cân nhắc

- A: Hỏi tự do từng câu khi phát sinh.
- B: Hỏi có cấu trúc, gom theo lô, không chặn việc khác.
- C: Không hỏi, tự quyết mọi thứ.

## Quyết định

Chọn B:

1. Mỗi câu hỏi phải có: bối cảnh 1–2 dòng, phương án A/B/C, đề xuất của agent và lý do.
2. Gom câu hỏi theo lô: một card buổi sáng, một card buổi tối. Không hỏi lắt nhắt.
3. Hỏi không được chặn việc khác: task bị vướng thì gắn `needs-decision`, chuyển sang task khác ngay.
4. Không hỏi lại điều Board đã quyết. Trước khi hỏi, tra `decisions/`. Tình huống chỉ khác chút ít so với quyết định cũ → tự áp dụng và ghi chú "áp dụng theo D-xxxx".
5. Board cần đề xuất và quyết định trong phạm vi được giao, không phải xin phép mọi thứ. Phạm vi cần Board duyệt nằm ở Approval matrix (`handbook/rules.md`, mục 6 chỉ thị). Ngoài phạm vi đó: tự quyết và báo cáo.
