# D-0023 — Effort mặc định max; người giao việc chọn effort cho từng task

- **Ngày:** 2026-10-02
- **Phạm vi:** Toàn công ty Hoang LLC (agent Claude).
- **Nguồn:** Chỉ thị Board ngày 2026-10-02 trong phiên Claude Code của Board.

## Lời Board

> "Tối muốn thêm 1 phần là khi tôi chỉ định thư kí tạo nhiều task cho các agent khác thì có thể chỉ định được cả effort sử dụng không nhỉ? Về cơ bản thì dùng opus 5.5 vì tôi có 2 tài khoản max 20 nên chắc quota không thành vấn đề nhưng effort hight hay max đôi khi ko phải lúc nào cũng cần max."

> "Tôi muốn mặc định là max luôn, và thư ký cũng có thể quyết effort nếu chính nó giao task."

## Quyết định

1. Mọi agent Claude mặc định chạy Opus 5.5 (`claude-opus-5-5`), effort `max`.
2. Effort của từng task đặt bằng `assigneeAdapterOverrides.adapterConfig.effort` trên issue (low, medium, high, xhigh, max). Board nêu mức thì đặt đúng mức đó.
3. Thư ký tự chọn effort cho task chính nó giao khi Board không nêu, theo bảng ở `operating-model` mục 4. Theo D-0022 (agent ngang quyền), agent khác tạo child issue cũng chọn theo cùng bảng.
4. Trong `assigneeAdapterOverrides` chỉ đặt `effort`, và `model` khi Board chỉ định.
