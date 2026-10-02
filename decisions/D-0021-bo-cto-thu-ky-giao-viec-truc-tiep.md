# D-0021 — Bỏ CTO và ChiefOfStaff; CEO thành Thư ký (ThuKy); agent ngang hàng, tự tuân thủ security, chỉ Board quyết; Hindsight gọn

- **Ngày:** 2026-10-02
- **Phạm vi:** Toàn công ty Hoang LLC (mọi agent). Không áp dụng cho công ty ThangChiba.
- **Nguồn:** Chỉ thị Board ngày 2026-10-02 trong phiên Claude Code của Board, và câu trả lời của Board cho các câu hỏi áp dụng cùng ngày. Cả hai chép nguyên văn vào task "Áp dụng D-0021: bàn giao việc của CTO và ChiefOfStaff".
- **Sửa đổi:** D-0001, D-0002, D-0005, D-0007, D-0008, D-0009, D-0010, D-0011, D-0013, D-0016, D-0017, D-0018, D-0020 (chi tiết ở mục "Sửa quyết định cũ").

## Bối cảnh

- CTO là trạm trung chuyển: tạo 101 task cho dev và QA, nhận lại 53 task "review + merge", tự quyết 11 card hỏi Board.
- 88 card để `resolverPolicy: anyone`, nên agent trả lời thay Board được. Instruction nhiều agent bảo xin duyệt CTO hay CEO cho việc `handbook/rules.md` mục 3–4 giao Board.
- ChiefOfStaff gần như không chạy từ khi daily brief bị huỷ (card HOA-82).
- Hindsight nhiều dữ liệu thừa: gần nửa là ghi chép agent tự lưu, phần lớn còn lại từ một mô tả task rất dài (HOA-12).

## Lời Board

Chỉ thị:

> "Bỏ vị trí CTO, CEO sẽ trực tiếp giao nhiệm vụ cho các agent bên dưới, tránh tối đa việc tam sao thất bản.
> CEO sẽ không làm việc nhiều nữa, chủ yếu là hoạt động như vai trò của thư kí(có thể đổi tên thành thư kí và bỏ thư kí cũ).
> Nhiệm vụ của nó chủ yếu là nhận lệnh của tôi, phân việc cho agent mà không bịa đặt thêm thắt, tuy nhiên thì có thể hỏi lại tôi để làm rõ hơn. Vì có triển khai hindsight nên sẽ đưược suy luận dựa theo các quyết định của tôi trong quá khứ.
> Sửa lại các bộ skill để các agent hoạt động hiệu quả hơn, sẽ thay đổi cấu trúc organization như trên. Thư kí sau khi nhận lệnh sẽ lưu lại dần trong hindsight để sau này có thể quyết định nhiều hơn thay tôi. Vì vậy nhiệm vụ chính thư kí chỉ là báo cáo, tổng hợp, các agent khác đều phải biết về security rule thay vì hỏi agent cấp trên. Tôi mới là người quyết.
>
> Những việc ghi vào hindsight cần compact hơn, hiện tại tôi thấy khá nhiều dữ liệu thừa, hãy dùng agent để tóm tắt lại công việc khi lưu."

> "Hiện chưa cần migration vội vì thấy macos cũng ko phải bottleneck. chỉ cần lưu ý chạy code trên máy win desktop."

> "nếu đơn giản thì clone và tạo giúp tôi HoangLLCV2 cũng được, nếu phức tạp thì hỏi lại tôi, tôi không muốn thao tác quá nhiều"

Trả lời câu hỏi áp dụng:

- Ai được SSH vào MacbookServer thay CTO: "Infra, dev, thư kí, tức là các agent sẽ gần như bình đẳng, chỉ là quản lí bộ skill khác nhau. Và đều tuân thủ security"
- Ai review PR thay CTO: "Dev và Infra tự do merge code vào. Chỉ những task quan trọng thì phát hành PR để tôi review. Lưu ý khi phát hành PR cần trình bày ngắn gọn súc tích như format tôi đã yêu cầu lần trước."
- Ngoại lệ hạ tầng chi phí thấp của Pro5: "Dưới $30/tháng thì khỏi hỏi"

## Phương án đã cân nhắc

- **Cách làm:** A, sửa tại chỗ trong Hoang LLC, giữ issue, secret, project, routine và bank Hindsight. B, tạo HoangLLCV2: bank Hindsight gắn theo id công ty nên bắt đầu rỗng, phải chuyển tay 49 issue đang mở, secret, project, routine, skill. B không đơn giản nên làm A.
- **SSH MacbookServer:** chỉ InfraEngineer / không ai / chỉ FullstackDev. Board chọn InfraEngineer, FullstackDev và Thư ký.
- **Review PR:** QA hoặc agent dev khác người viết review mọi PR / Board duyệt từng PR Pro5. Board chọn: dev và infra tự merge, chỉ task quan trọng mở PR cho Board review.
- **Hạ tầng Pro5:** mỗi lần apply một card Board (D-0011) / giữ ngoại lệ chi phí thấp HOA-220. Board giữ ngoại lệ, mức dưới $30/tháng.

## Quyết định

1. **Bỏ agent CTO và ChiefOfStaff.** Tạm dừng, bàn giao việc trên task bàn giao D-0021, rồi terminate.
2. **Agent CEO đổi tên thành ThuKy, chức danh "Thư ký", giữ role `ceo`.**
   - Việc chính: nhận lệnh Board, hỏi lại Board khi chưa rõ; giao việc thẳng cho agent bằng đúng lời Board, không bịa thêm (`order-dispatch`); báo cáo, tổng hợp; ghi `decisions/` và Hindsight.
   - ThuKy vẫn được làm việc và chạy lệnh như agent khác, theo cùng quy tắc an toàn. ThuKy không duyệt hay quyết thay Board.
   - Quyền nền tảng của role `ceo` (tuyển agent, đổi quyền agent khác, import/export công ty) chỉ dùng khi Board ra lệnh đúng việc đó. Công ty bật "tuyển agent cần Board duyệt". ThuKy không sửa thẳng instruction hay skill; mọi thay đổi là đề xuất chờ Board duyệt.
3. **Các agent gần như bình đẳng.** Agent khác nhau ở bộ skill mình quản lý, không ở quyền duyệt. Mọi agent `reportsTo` ThuKy chỉ để định tuyến việc; chức danh hay quan hệ báo cáo không cho quyền duyệt.
4. **Chỉ Board quyết.**
   - Mọi agent tự nắm và tự tuân thủ quy tắc an toàn: khối "Decision authority" trong `AGENTS.md`, skill `security-baseline`.
   - Việc cần Board quyết thì agent tự hỏi Board bằng card `human_only` trên task của mình (`operating-model`).
   - Không agent nào, kể cả ThuKy, duyệt, chấp nhận hay trả lời card hỏi Board. Công ty khoá mọi loại card ở `human_only`.
5. **Áp dụng quyết định cũ.** Agent bất kỳ được áp dụng quyết định cũ của Board cho trường hợp giống hoặc chỉ khác chút ít, ghi "Áp dụng theo D-xxxx / HOA-n" kèm lời Board nguyên văn và link nguồn; trường hợp không giống hệt thì ghi rõ khác ở đâu. Suy luận rộng hơn chỉ được làm phương án đề xuất trong card hỏi Board. Đây là quyền tự quyết duy nhất Board giao cho Thư ký; chỉ mở rộng bằng D-file mới.
6. **Hindsight gọn.** Chỉ bật cho Hoang LLC (cài đặt plugin `compactMemory`); ThangChiba giữ nguyên.
   - Chỉ ThuKy ghi. Mỗi lệnh hay quyết định của Board tối đa một bản ghi, không quá 300 ký tự, tóm tắt bằng prompt cố định (`order-dispatch`). Dòng nội dung chỉ chứa lời Board nguyên văn trong «». Comment xác nhận chép toàn văn bản ghi để Board sửa được.
   - Agent khác không ghi; bài học đề xuất viết thành dòng `Ghi nhớ đề xuất: …` trong comment đóng task. Đây là quy ước trong script và instruction; server chưa chặn.
   - Plugin vẫn tự lấy lời Board. Mô tả task cắt ở 1.500 ký tự, ngữ cảnh comment trước cắt ở 300 ký tự.
   - Mỗi mục nhớ trong prompt có nhãn nguồn: [Board], [Thư ký tóm tắt], [Agent đề xuất].
7. **Code chạy trên ThangChiba-Desktop** (máy Windows, WSL). Không chạy code, build, test của dự án trên MacbookServer. QA và ContentCreator dùng codex, đã đăng nhập trên Desktop ngày 2026-10-02, nên chạy trên Desktop như mọi agent. Lệnh `docker` trong Ubuntu WSL chưa dùng được, vì Docker Desktop chưa bật WSL integration cho Ubuntu. Thiếu công cụ thì báo trên task, không chuyển sang Mac.
8. **Việc của CTO chuyển như sau:**
   - a. **SSH vào MacbookServer (D-0018):** InfraEngineer, FullstackDev và ThuKy, cùng key, cùng điều kiện: lệnh chỉ đọc và lệnh phục vụ dự án Hoang LLC thì tự chạy; sửa phần dùng chung ngoài Hoang LLC phải hỏi Board trước. QA, ProductManager, MarketingManager, ContentCreator không SSH. Không agent nào đọc key, secret hay DB của Paperclip trên Mac (`security-baseline`).
   - b. **Merge PR (thay điều kiện a của D-0013):** FullstackDev và InfraEngineer tự merge code của mình, không cần review của agent khác hay QA. Vẫn tự kiểm: kiểm tra xanh và GitGuardian xanh (điều kiện b, c của D-0013).
     - Task quan trọng thì mở PR cho Board review và chỉ merge khi Board đồng ý. Task quan trọng là việc thuộc phần Board quyết (`security-baseline` mục 1): ảnh hưởng security, tiền, thay đổi lớn, dữ liệu production, hạ tầng ngoài phần mục 12 cho tự làm, việc không hoàn tác được.
     - PR viết ngắn gọn, súc tích, theo format Board đã yêu cầu (`pr-standard`).
     - QA kiểm khi được giao; QA không phải điều kiện merge.
   - c. Kiểm AWS read-only cho agent khác (D-0017 mục 2): InfraEngineer.
   - d. Hoang Auth (D-0016 mục 4): InfraEngineer tạo client theo mặc định trong skill `hoang-auth`. Thiết kế khác mặc định đó do Board quyết; agent làm việc đề xuất A/B/C.
   - e. Policy IP trang quản trị Keycloak (D-0020): InfraEngineer.
   - f. Phần kỹ thuật chuyển Figure sang nhánh `release` (D-0013): InfraEngineer. ThuKy đưa việc này vào câu hỏi go-live cho Board.
   - g. Routine kiểm purge hằng ngày (HOA-201) và HOA-344: InfraEngineer.
9. **Việc của Thư ký cũ:**
   - Daily brief đã huỷ (card HOA-82), không bật lại.
   - Báo giá thẻ NTAG213 (HOA-20, HOA-244): ThuKy giữ đúng như task đang ghi, vì chỉ ThuKy có mailbox `HOANG_MAIL_*`. Không cấp quyền mới. Đây là ngoại lệ duy nhất cho việc ThuKy gửi ra ngoài: đọc mailbox, gửi tối đa một thư nhắc đúng phạm vi task (theo card HOA-82, câu 1).
10. **Dọn Hindsight một lần** (Board: "Những việc ghi vào hindsight cần compact hơn"): sao lưu trước; agent tóm tắt các ghi chép agent tự lưu thành tối đa 25 bài học gọn; xoá bản gốc và các observation sinh từ chúng; gửi lại HOA-12 ở dạng đã cắt.
11. **Sửa tài liệu:** `handbook/rules.md` v0.4; `skills/README.md` v0.2 (thêm `security-baseline`, `order-dispatch`; `needs-decision-protocol` gộp vào `operating-model`; `verify-and-report` gộp vào `concise-status-report`); instruction agent; mô tả Project Pro5 và Figure.
12. **Hạ tầng Pro5, tới launch M1:** thay đổi hạ tầng mà dự báo chi AWS của Pro5 vẫn dưới $30/tháng thì không cần card Board; apply rồi báo cáo sau. Thay đổi ảnh hưởng security (kể cả IAM) hoặc vượt $30/tháng vẫn cần card Board cho từng lần apply (D-0011). Các lệnh `terraform-plan-only` cấm khi không có card riêng vẫn cấm. Từ launch M1 quay về D-0011.

## Sửa quyết định cũ

- D-0001 mục 2: thay "một card buổi sáng, một card buổi tối" bằng: mỗi agent hỏi Board bằng card `human_only` trên task của mình, mỗi task tối đa một card đang chờ. Mục 4: tra cả Hindsight.
- D-0002: ThuKy ghi `decisions/`. Vòng lặp mục 1: chỉ quyết định áp dụng rộng hơn một task mới thành D-file; quyết định trong một task chỉ ghi Hindsight. Mục 4 (rà soát tối Chủ nhật) hết hiệu lực: ChiefOfStaff đã bỏ, không có routine rà soát hằng tuần. Câu cuối: agent không đọc `rules.md` đầu mỗi run; quy tắc đến qua `AGENTS.md` và skill.
- D-0005 mục 3: danh sách skill theo `skills/README.md` v0.2.
- D-0007 mục 2, D-0008 mục 3, D-0009 mục 3, D-0010 mục 4: hết hiệu lực (ChiefOfStaff đã bỏ, daily brief đã huỷ).
- D-0011: Pro5 theo mục 12 tới launch M1.
- D-0013: mục 8b, 8f. Kiểm soát bù "CTO review mọi PR" của Pro5 bỏ; email cảnh báo mỗi lần deploy giữ nguyên.
- D-0016 mục 4, D-0017 mục 2, D-0018 (người được SSH), D-0020 (người sửa policy IP): theo mục 8.
- Bỏ chỉ thị "nhờ CTO hoặc CEO trước khi hỏi tôi" (HOA-142, 25/09; HOA-233, 29/09). HOA-236 giữ nguyên.
- HOA-220 (29/09: "Phần infra nếu không phát sinh nhiều chi phí thì không cần hỏi ý kiến tôi"): mức chi phí là dưới $30/tháng (mục 12).

## Hệ quả

- Danh sách bàn giao từng issue và routine nằm trên task bàn giao D-0021.
- Ghi chú "Sửa bởi D-0021" ở đầu mỗi D-file trong mục "Sửa quyết định cũ".
