# D-0012 — AWS: agent chỉ dùng credential Paperclip inject vào env

- **Ngày:** 2026-09-28
- **Phạm vi:** Toàn công ty (mọi agent, mọi máy agent chạy hoặc ssh vào)
- **Nguồn:** Board trả lời card trên HOA-196; CTO thực hiện tại HOA-196
- **Sửa đổi:** D-0005 mục 5 (với AWS: bỏ `AWS_PROFILE` và profile trên máy chủ)

## Bối cảnh

D-0005 mục 5 cho Project khai báo `AWS_PROFILE`, secret nằm trong profile trên máy chủ. Trên máy dùng chung, profile có sẵn không nhất thiết là credential cấp cho agent, nên agent có thể dùng nhầm. Credential AWS của agent hiện được Paperclip inject vào env của run (secret binding).

Board trả lời (HOA-196): *"… chỉ định các agent chỉ dùng Profile cung cấp trong env, và luôn chạy lệnh với acceskey, token, đó."*

## Phương án đã cân nhắc

- A: Thu hồi credential có sẵn trên máy chủ. Board không chọn: ảnh hưởng nhiều nơi.
- B: Agent chỉ dùng credential trong env, truyền rõ ràng trong từng lệnh, tắt file profile. Board chọn.
- C: Tách user OS riêng cho agent trên máy chủ (kiểm soát cứng). Chưa làm, chỉ làm khi Board yêu cầu.

## Quyết định

Chọn B:

1. Agent chỉ dùng AWS credential Paperclip inject vào env của run. Mọi lệnh AWS (CLI, SDK, Terraform) chạy với key đó truyền rõ ràng, kèm `AWS_CONFIG_FILE=/dev/null AWS_SHARED_CREDENTIALS_FILE=/dev/null`.
2. Không dùng `--profile`, `AWS_PROFILE`, file trong `~/.aws`, hay `profile`/`shared_credentials_files` trong Terraform, trên bất kỳ máy nào. Profile có sẵn trên máy chủ không phải credential của agent: không đọc, không chép, không dùng.
3. Không ghi credential ra file, không in ra log. Chạy lệnh AWS trên máy của run; phải chạy trên dev machine thì chỉ truyền key qua stdin (xem `skills/dev-machine`).
4. Lệnh AWS đầu tiên của mỗi run: `sts get-caller-identity` phải ra đúng principal được cấp. Khác (nhất là `:root`) → dừng, báo cáo.
5. Không có credential trong env, hoặc credential lỗi → dừng, báo trên issue, không đi tìm credential khác. *(Sửa bởi D-0017: chỉ áp dụng khi task cần chạy lệnh AWS. Task không chạy lệnh AWS nào thì không cần binding, và thiếu binding không chặn task đó.)*

## Hệ quả

- Block "AWS credentials (company rule, HOA-196)" trong `AGENTS.md` của mọi agent.
- `skills/terraform-plan-only`: bỏ `AWS_PROFILE`, thêm quy tắc chỉ dùng credential trong env.
- `skills/dev-machine`: thêm quy tắc AWS (key chỉ qua stdin, `bash -ls`).
- D-0005 mục 5 và `handbook/rules.md` mục 4.1: ghi chú AWS theo D-0012.
- Credential AWS cho Project mới: cấp qua secret binding của Paperclip (env), không qua `AWS_PROFILE`.
- Quy tắc này chặn việc dùng nhầm, không phải kiểm soát cứng (phương án C).
