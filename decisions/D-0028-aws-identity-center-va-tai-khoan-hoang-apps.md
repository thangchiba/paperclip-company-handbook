# D-0028 — AWS: đăng nhập bằng Identity Center admin@hoang.jp, app mới vào tài khoản chung hoang-apps

- **Ngày:** 2026-11-07
- **Phạm vi:** Mọi tài khoản AWS của Hoang LLC trong Organization: tài khoản quản lý (trả tiền), tài khoản Hokidon và mọi tài khoản tạo sau; nơi deploy game/app mới.
- **Nguồn:** Board giao [HOA-749](/HOA/issues/HOA-749) cho InfraEngineer và trả lời card «Gom quản lý AWS về admin@hoang.jp» trên task đó (2 câu, đều chọn A). InfraEngineer đề xuất và thực hiện tại HOA-749.
- **Sửa đổi:** Không.

## Bối cảnh

- Organization có 2 tài khoản: tài khoản quản lý (root Gmail của Board, nhận hoá đơn gộp) và tài khoản Hokidon (root `hokidon@hoang.jp`, không có hoá đơn riêng).
- Đăng nhập root chỉ vào được đúng tài khoản của root đó, nên từ root tài khoản quản lý Board không thấy tài nguyên Hokidon.
- `admin@hoang.jp` là hộp thư Microsoft 365; trên AWS chưa có gì cho email này.

## Lời Board

> "Có account admin@hoang.jp chưa? tôi muốn dùng account này để quản lí tất cả resource trong hoang llc cho tiện. vì có nhiều game, app sẽ triển khai nếu chia tài khoản khá phiền"

## Phương án đã cân nhắc

Câu 1: «admin@hoang.jp quản lý toàn bộ AWS bằng cách nào?»

- «A (đề xuất): user Identity Center admin@hoang.jp»: Board chọn. Mô tả trên card: «Tôi bật IAM Identity Center (miễn phí), tạo user, cấp Admin + xem hoá đơn trên ThangChiba, pro5-prod và mọi tài khoản sau này. Board chỉ đặt mật khẩu + MFA từ thư mời (~3 phút), rồi đăng nhập 1 cổng là vào được mọi tài khoản.»
- «B: chỉ đổi email root ThangChiba sang admin@hoang.jp»
- «C: cả A và B»

Câu 2: «Game/app mới deploy vào tài khoản AWS nào?»

- «A (đề xuất): một tài khoản chung mới hoang-apps»: Board chọn. Mô tả trên card: «Setup 1 lần (SES, quota Lambda, OIDC GitHub), mọi app dùng chung, chi phí từng app xem theo tag. Game thử nghiệm không đụng được dữ liệu Hokidon. Tôi chỉ tạo khi app đầu tiên cần, miễn phí.»
- «B: mỗi app một tài khoản con»
- «C: chung vào pro5-prod»

## Quyết định

1. **Đăng nhập AWS hằng ngày:** user IAM Identity Center `admin@hoang.jp`, có quyền Admin và xem hoá đơn trên tài khoản quản lý, tài khoản Hokidon và mọi tài khoản tạo sau. Email root của tài khoản quản lý giữ nguyên (không chọn B, C).
2. **Game/app mới:** deploy vào một tài khoản con chung mới `hoang-apps`, không mỗi app một tài khoản, không vào tài khoản Hokidon. Chi phí từng app xem theo tag. InfraEngineer tạo tài khoản này khi app đầu tiên cần.

## Hệ quả

- Agent vẫn dùng IAM user `doraemon` và `OrganizationAccountAccessRole` như trước (quy tắc Board tại HOA-130); Identity Center là đường đăng nhập của Board.
- AWS chỉ cho bật Identity Center của Organization trên console của tài khoản quản lý (API `CreateInstance` bị từ chối ở đó), nên Board bật một lần trên console (HOA-749).
- InfraEngineer suy ra: mỗi tài khoản mới trong Organization, kể cả `hoang-apps`, được gán quyền Admin cho `admin@hoang.jp` ngay khi tạo.
