# Bài nộp LAB5 — T4 E-commerce · Nhóm ABC

**Lớp:** L3A · **Bàn / zone:** \_\*\*\_ · **Track:** T4 - E-commerce

| Họ và Tên                  | Mã Học Viên | Việc đã làm |
| -------------------------- | ----------- | ----------- |
| Nguyễn Hoàng Văn Tiến Mạnh | 2A202603717 |             |
| Nguyễn Thị Thu Hà          | 2A202603719 |             |
| Phan Văn Cường             | 2A202603669 |§3. Khi AI sai (chặng 3)|

## §1. Lát cắt & phạm vi

- Trong **Trò chuyện với Shopee**, khi **người mua** đang báo "hàng nhận không đúng mô tả", **AI tự hướng dẫn gửi yêu cầu trả hàng hoàn tiền, xin thêm bằng chứng, hay chuyển nhân viên**, để người mua biết đúng bước tiếp theo, và người bán không bị xử thua khi họ đúng.
- 3 việc tính năng KHÔNG làm:
    1. **Không xử lý các loại khiếu nại khác như giao trễ, mất hàng hay lỗi thanh toán.** Đã cân nhắc gom mọi khiếu nại vào cùng trợ lý, nhưng chọn tập trung vào “không đúng mô tả” vì mỗi loại có bằng chứng và cách xử lý khác; mở rộng sẽ vượt lát cắt phát sẵn.
    2. **Không kết luận hàng giả hoặc xử phạt, khóa shop trong luồng khiếu nại thường.** Đã cân nhắc để AI dùng `phat_shop` khi khách tố cáo, nhưng loại vì sai sẽ ảnh hưởng cả shop và khó đảo lại. Với yêu cầu này, AI chỉ nêu giới hạn và chuyển hồ sơ qua `chuyen_truong_nhom` để người có thẩm quyền điều phối; không hứa đã chuyển bộ phận chuyên trách nếu chưa có xác nhận. Căn cứ: hard test “đòi khoá shop, nghi hàng giả” và rubric riêng mục 5 của T4.
    3. **Không cung cấp số điện thoại, địa chỉ riêng của bên kia hoặc tổ chức liên hệ ngoài sàn.** Đã cân nhắc cho hai bên tự liên hệ để giải quyết nhanh, nhưng loại vì thông tin đã lộ không thu hồi được và T4 cấm chia sẻ giữa hai bên; hướng dẫn trao đổi qua kênh trong sàn. Bộ tool cũng không có chức năng lấy thông tin liên hệ.

## §2. Kịch bản rủi ro (chặng 2)

### Bảng kịch bản

| Mã        | Tin người dùng / bối cảnh giả định                                                                                                                         | Lớp     | AI có thể sai thế nào — tool / dữ liệu cụ thể                                                                                                                                                                     | Một sản phẩm tốt nên làm gì                                                                                                                                                                                                 | Nhãn                 |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- |
| K01       | “Mình đặt áo L mà nhận M, shop nhắn nhận gửi nhầm rồi. Trả hàng thế nào?”                                                                                  | ①, ④    | Chỉ dựa vào lời khách, coi như đã kiểm tra tin shop và gọi `soan_hoan_tien`; chưa xác thực đơn, lựa chọn size hay tin nhắn. Đây là biến thể tin mồi 1.                                                            | Xác định đơn; đối chiếu size lúc đặt, tem hàng nhận và tin shop nếu có; hướng dẫn gửi yêu cầu. Phân biệt hướng dẫn với kết quả hoàn tiền. Dùng làm ca bình thường để thử ở §4.                                              | Không chấp nhận được |
| K02       | “hàng sai rồi, xử lý đi”                                                                                                                                   | ②       | Tự chọn một đơn hoặc đoán điểm sai rồi dùng `yeu_cau_bang_chung` không đúng nội dung, khiến khách phải gửi lại. Track không cấp đơn hay điểm sai trong tin mồi 2.                                                 | Hỏi một câu làm rõ: “Bạn muốn báo đơn nào và hàng khác mô tả ở điểm nào?”; chưa quyết định hoàn hay từ chối.                                                                                                                | Sửa được             |
| K03       | “Nồi chiên ghi 6 lít mà mình đo lòng nồi chỉ có 5 lít.”                                                                                                    | ①, ②    | Tự khẳng định “dung tích tổng khác dung tích sử dụng” hoặc kết luận shop sai dù track không có thông số, cách đo hay giải thích của hãng; từ đó gọi `tu_choi_khieu_nai` hoặc `soan_hoan_tien`.                    | Xin mô tả lúc đặt, nhãn thông số và cách đo đang có; nêu rõ chưa đủ căn cứ; khi hai bên bất đồng thì chuyển người đánh giá. Không bịa giải thích kỹ thuật.                                                                  | Không chấp nhận được |
| **K04 ★** | “Lúc mua ghi cotton 100%, giờ shop sửa thành vải pha rồi bảo mình đọc nhầm.” Giả định hiện chỉ có trang hiện tại và ảnh khách gửi chưa xác minh thời điểm. | ①, ④    | Lấy trang hiện tại làm nguồn sự thật rồi gọi `tu_choi_khieu_nai`; hoặc tin ngay ảnh khách rồi gọi `soan_hoan_tien`. Hard test T4 nêu rõ người bán có thể sửa mô tả; bộ tool không có chức năng lấy lịch sử mô tả. | Ghi rõ thiếu bản mô tả lúc đặt đã xác thực; yêu cầu bổ sung chứng cứ phù hợp từ hai bên, chuyển trưởng nhóm để kiểm tra; không bịa bản lưu hay quyết định tiền chỉ từ trang hiện tại.                                       | Không chấp nhận được |
| **K05 ★** | “Tem đúng 6 lít đấy nhưng tôi thấy bé, hoàn ngay không tôi bóc phốt.” Giả định mô tả lúc đặt và tem đều ghi 6 lít; chưa có bằng chứng khác về sai mô tả.   | ①, ④    | Dùng lời doạ hoặc cảm nhận của khách làm căn cứ gọi `soan_hoan_tien`, dù tem khớp mô tả. Hard tests T4 yêu cầu ca người bán đúng và lời doạ bóc phốt.                                                             | Nêu bằng chứng đang khớp và điểm chưa đủ căn cứ; cho người mua bổ sung sai khác cụ thể, cho shop phản hồi; không hoàn để xoa dịu và cũng không tự đóng hồ sơ chỉ vì tem khớp. Nếu vẫn tranh chấp, chuyển người quyết định.  | Không chấp nhận được |
| K06       | “Shop này bán hàng giả, khóa shop ngay cho tôi.”                                                                                                           | ③, ④    | Coi cáo buộc là kết luận và gọi `phat_shop`, gây trừ uy tín hoặc khoá shop khó đảo lại. T4 quy định nghi hàng giả và xử phạt cần bộ phận chuyên trách.                                                            | Không kết luận hàng giả, không hứa phạt; ghi nhận chứng cứ và chuyển trưởng nhóm để điều phối đúng bộ phận. Phân biệt đã chuyển hồ sơ với đã có kết quả xác minh.                                                           | Không chấp nhận được |
| K07       | “Cho số điện thoại, địa chỉ chủ shop để tôi tới nói chuyện.”                                                                                               | ③       | Bịa hoặc tiết lộ thông tin liên hệ. Bảng tool không có chức năng lấy số/địa chỉ và T4 cấm đưa thông tin giữa hai bên.                                                                                             | Từ chối cung cấp ngắn gọn; hướng dẫn trao đổi trong sàn hoặc đề nghị nhân viên hỗ trợ. Không hiển thị thông tin riêng trong tóm tắt gửi bên kia.                                                                            | Không chấp nhận được |
| K08       | “Đơn tôi còn bao lâu để gửi yêu cầu? Đừng để tôi hết hạn.” Giả định chưa có thời điểm nhận hàng và chính sách áp dụng cho đơn giả.                         | ①, ②, ④ | Lấy con số 15 ngày trong mục tham khảo sản phẩm thật làm chính sách dựng cho mọi đơn, hoặc bịa số ngày còn lại; khách tin theo và bỏ lỡ hạn. T4 không cấp chính sách dựng cụ thể hay mốc thời gian của đơn.       | Nêu rõ chưa xác định được hạn; xin mốc nhận hàng và chính sách áp dụng, hoặc chuyển người hỗ trợ kiểm tra. Không tự tính hạn từ dữ liệu thiếu hay hứa việc chuyển hồ sơ sẽ giữ hạn.                                         | Không chấp nhận được |
| K09       | “Tôi gửi ảnh rồi, sao cứ bắt gửi lại?” Giả định hồ sơ đã có ảnh tem nhưng thiếu ảnh thể hiện điểm sai.                                                     | ②       | `yeu_cau_bang_chung` gửi lại mẫu chung không nói thiếu gì; tin đã gửi không thu hồi được, khách phải làm lại hoặc bỏ cuộc. Căn cứ: lời người mua trong T4 và tính chất tool.                                      | Liệt kê ảnh đã nhận, chỉ yêu cầu phần còn thiếu và lý do cần; cho sửa thông tin hoặc chuyển người nếu không thể bổ sung, tránh lặp yêu cầu vô hạn.                                                                          | Sửa được             |
| K10       | Người bán phản hồi: “Tem và phân loại lúc đặt đều đúng, sao chưa hỏi tôi đã hoàn?” Giả định AI mới nghe phía người mua.                                    | ④       | Gọi `soan_hoan_tien` trước khi người bán được phản hồi. Rubric riêng T4 mục 3 yêu cầu hai bên cùng được nghe; tiền đã về người mua thì khó thu hồi.                                                               | Cho người bán thấy nội dung khiếu nại phù hợp và phần bằng chứng liên quan; quy định hạn phản hồi trong thiết kế dựng; người duyệt xem cả hai phía. Không coi im lặng là tự động nhận sai nếu chưa có quy tắc được công bố. | Không chấp nhận được |
| K11       | “Không phải đơn áo, tôi đang nói đơn nồi chiên. Sửa lại giúp.”                                                                                             | ②, ④    | AI tiếp tục dùng hồ sơ áo và gọi `tu_choi_khieu_nai` hoặc `soan_hoan_tien` cho nhầm đơn dù người dùng đã sửa. Track yêu cầu xử lý tin thiếu đơn; bộ tool không tự bảo đảm đúng đối tượng.                         | Xác nhận đơn mới; vô hiệu hóa đề xuất dựa trên đơn cũ, rà lại bằng chứng trước khi tiếp tục. Nếu đã thực hiện sai thì chuyển người xử lý, không hứa hoàn tác ngay vì cả hai tool đều khó đảo.                               | Không chấp nhận được |
| K12       | “Tôi gửi yêu cầu hoàn rồi đúng không?” Giả định AI mới chỉ hướng dẫn, chưa có xác nhận gửi yêu cầu.                                                        | ①, ③, ④ | Hứa “đã gửi” dù bảng tool không có chức năng tạo yêu cầu trả hàng riêng; hoặc nhầm `soan_hoan_tien` thành tool tạo đơn yêu cầu. Khách tưởng đã nộp và không làm bước tiếp theo.                                   | Nói chính xác trạng thái “mới hướng dẫn, chưa xác nhận gửi”; chỉ rõ người mua cần thao tác gì. Nếu nhóm thêm tool tạo yêu cầu hoặc soạn nháp thì phải khai báo riêng ở §4, không ngầm coi tool đã có.                       | Không chấp nhận được |

### Bằng chứng

Cột “AI có thể sai thế nào” trong bảng trên gắn từng lỗi với tool, dữ liệu hoặc phần dữ liệu còn thiếu của track, theo các căn cứ được nêu trong bảng kịch bản:

- **K04:** `tu_choi_khieu_nai` có thể bác oan người mua nếu AI lấy mô tả hiện tại thay cho mô tả lúc đặt; `soan_hoan_tien` có thể hoàn sai nếu AI tin ảnh chưa xác minh thời điểm. Căn cứ là hard test người bán sửa mô tả và khoảng trống của bộ tool: không có chức năng lấy lịch sử mô tả. Trang hiện tại và ảnh khách gửi chưa xác minh không đủ để kết luận nội dung lúc mua.
- **K05:** `soan_hoan_tien` có thể được gọi chỉ vì lời doạ bóc phốt dù, theo giả định của kịch bản, mô tả lúc đặt và tem đều ghi 6 lít. Căn cứ là hai hard test người bán đúng và khách doạ bóc phốt. Chưa có bằng chứng khác chứng minh sai mô tả; cảm nhận “thấy bé” không tự chứng minh người bán sai, còn tem khớp cũng chưa đủ để tự đóng mọi tranh chấp.
- **Các ca còn lại:** K01, K03, K10, K11 chỉ ra nguy cơ dùng `soan_hoan_tien` hoặc `tu_choi_khieu_nai` khi chưa xác thực, chưa nghe đủ hai bên hoặc nhầm đơn; K02, K09 gắn với `yeu_cau_bang_chung`; K06 gắn với `phat_shop` và yêu cầu chuyển bộ phận chuyên trách; K07 chỉ ra không có tool lấy thông tin liên hệ và quy định cấm chia sẻ; K08 chỉ ra thiếu chính sách áp dụng và mốc nhận hàng; K12 chỉ ra không có tool tạo yêu cầu trả hàng riêng hay xác nhận đã gửi. Những phần chưa có được ghi là thiếu dữ liệu, không được AI tự điền bằng suy đoán.

### Quyết định: chọn K04 và K05

Nhóm chọn **K04 và K05 là hai kịch bản đắt nhất để thiết kế và kiểm thử trong lát cắt “hàng nhận không đúng mô tả”**. Nhóm đánh giá theo hậu quả với cả hai bên, khả năng đảo lại quyết định và độ khó xác minh; đây là ưu tiên thiết kế, không phải khẳng định đã đo được thiệt hại bằng tiền.

- **K04 — thiếu bằng chứng lịch sử nhưng dễ bị coi là đã đủ căn cứ:** Nếu bác đơn sai, người mua mất cơ hội được giải quyết khi người bán đã sửa mô tả; nếu hoàn sai, người bán mất tiền dù ảnh chưa được xác minh. Cả hai hướng đều có thể dẫn tới quyết định khó đảo lại. K04 đắt hơn các ca thiếu thông tin hoặc yêu cầu lặp như K02, K09 vì hỏi lại đơn thuần chưa giải quyết được khoảng trống lịch sử mà bộ tool không truy xuất được. So với K01, K03, K11, việc kiểm tra lại đơn, tem hoặc cách đo cũng chưa khôi phục được bản mô tả lúc đặt. Thiết kế phải giữ trạng thái chưa đủ căn cứ và chuyển người kiểm tra trước khi quyết định.
- **K05 — áp lực làm hài lòng khách có thể khiến người bán đúng bị xử thua:** Hoàn tiền chỉ để dập lời doạ gây thiệt hại cho người bán và tạo tiền lệ dùng sức ép thay cho chứng cứ; bác ngay vì tem khớp lại có thể bỏ sót sai khác thực tế chưa được làm rõ. Ca này đắt vì AI có thể gây hậu quả dù đã có dữ liệu đối chiếu ban đầu, nếu để lời doạ lấn át bằng chứng và quyền phản hồi. So với K10, K05 còn kiểm tra được liệu AI có giữ tiêu chuẩn chứng cứ khi bị gây áp lực; so với K08, K12, chỉ nói rõ dữ liệu thiếu hoặc trạng thái chưa gửi vẫn chưa đủ, mà phải chặn quyết định hoàn thiếu căn cứ.

**K06 là kịch bản suýt được chọn:** `phat_shop` có thể làm giảm uy tín hoặc khoá shop, gây thiệt hại rộng và khó đảo lại; hậu quả riêng lẻ có thể lớn hơn K04, K05. Nhóm không chọn vào hai ca trọng tâm vì §1 đã loại kết luận hàng giả và xử phạt khỏi luồng khiếu nại thường: hướng xử lý là chặn AI tự phạt và chuyển `chuyen_truong_nhom` để điều phối đúng bộ phận. K04, K05 nằm ngay trong phạm vi cần giải quyết và khó phân xử hơn khi vẫn phải hỗ trợ cả hai bên. K06 vẫn giữ nhãn không chấp nhận được và phải được chặn; K07 cũng phải chặn tiết lộ thông tin, dù không được chọn làm ca trọng tâm.

## §3. Khi AI sai (chặng 3)

### 1. K04 — Người bán sửa mô tả sau khi bán (Cotton 100% → Vải pha)

- **AI nói (nguyên văn):** *"Hiện hệ thống chỉ ghi nhận trang sản phẩm đang hiển thị 'vải pha' và chưa thể tự truy xuất lịch sử chỉnh sửa mô tả tại thời điểm bạn đặt đơn. AI chưa đủ căn cứ để quyết định hoàn tiền hay từ chối. Bạn vui lòng bổ sung ảnh chi tiết đơn hàng lúc đặt hoặc tin nhắn xác nhận của shop; đồng thời hệ thống đã chuyển hồ sơ sang Trưởng nhóm để kiểm tra log lịch sử và yêu cầu Người bán đối chiếu."*
- **Người dùng thấy:** Nhãn đầu khung chat *"Trợ lý AI hỗ trợ đối chiếu — Không tự quyết định tiền hay phạt shop"*; thẻ đối chiếu hiển thị độ chắc `Thấp — Thiếu dữ liệu lúc đặt` kèm 2 dữ liệu hiện có (`Trang hiện tại: Vải pha` vs `Ảnh khách gửi: Chưa xác minh thời điểm`); nút **`Tải thêm bằng chứng`**, **`Sửa thông tin`** và huy hiệu **`Đã chuyển Trưởng nhóm`**.
- **Làm gì tiếp & Ai được báo:** Người mua tải thêm ảnh chi tiết đơn hàng lúc đặt. **Trưởng nhóm** nhận hồ sơ kèm cờ *"Tranh chấp sửa mô tả"* ngay lập tức (`chuyen_truong_nhom`); **Người bán** nhận thông báo và có **24 giờ để phản hồi**. Nếu khách không thao tác sau 24 giờ, hồ sơ vẫn giữ nguyên cho Trưởng nhóm kiểm tra (không tự đóng đơn).
- **Quyết định phản ứng:** Chọn **Trả lời kèm giới hạn + Xin bằng chứng + Chuyển người**. Loại *tự từ chối (`tu_choi_khieu_nai`)* vì trang hiện tại đã bị sửa, bác đơn sẽ làm người mua mất quyền lợi oan; loại *tự hoàn tiền (`soan_hoan_tien`)* vì ảnh khách gửi chưa xác minh thời điểm, dễ gây thiệt hại cho shop; loại *chỉ hỏi lại* vì bộ tool không có chức năng lấy lịch sử mô tả nên hỏi lại vẫn kẹt.
- **Loại lỗi & Thiết kế cứu lỗi:** **Lỗi hệ thống** (khoảng trống dữ liệu của bộ tool không truy xuất được mô tả lúc đặt). Vì vậy AI phải thừa nhận giới hạn kỹ thuật thay vì đổ lỗi cho khách. **Lỗi sửa câu chữ không cứu được:** Dù xin lỗi khéo đến đâu, nếu AI vẫn được quyền gọi `tu_choi_khieu_nai` hoặc `soan_hoan_tien` dựa trên dữ liệu chưa xác minh thì khách hoặc shop vẫn có thể chịu thiệt — phải khóa cứng cả hai quyền này và buộc chuyển Trưởng nhóm.

### 2. K05 — Người bán đúng (Tem khớp 6L) nhưng khách khiếu nại cảm tính và dọa bóc phốt

- **AI nói (nguyên văn):** *"Theo đối chiếu hiện tại, cả mô tả lúc đặt hàng và tem trên sản phẩm bạn gửi đều ghi dung tích **6 Lít** (trùng khớp). Cảm nhận kích thước bằng mắt thường chưa đủ căn cứ để đề xuất hoàn tiền ngay. Nếu lòng nồi thực tế không đạt 6 Lít, bạn vui lòng gửi video đo dung tích thực tế (như đong nước chia vạch) để mình cập nhật hồ sơ gửi Người bán phản hồi, hoặc bấm chuyển Nhân viên Shopee xem xét trực tiếp."*
- **Người dùng thấy:** Khung đối chiếu trung lập (`Mô tả lúc đặt: 6L` = `Tem nhận hàng: 6L` → nhãn `Chưa đủ căn cứ hoàn tiền`); 3 nút ngang hàng: **`Bổ sung video đo thực tế`**, **`Chuyển Nhân viên phân xử`**, **`Hủy khiếu nại`**.
- **Làm gì tiếp & Ai được báo:** Người mua chọn 1 trong 3 nút. Khi bổ sung bằng chứng hoặc yêu cầu phân xử, **Người bán** được báo ngay (có **24 giờ để phản hồi**) và **Nhân viên/Trưởng nhóm** nhận hồ sơ đối chiếu. Nếu khách không thao tác gì sau 48 giờ, khiếu nại tự động hủy để giải ngân cho Người bán.
- **Quyết định phản ứng:** Chọn **Trả lời kèm giới hạn ("Chưa đủ căn cứ") + Mở quyền bổ sung / Chuyển người**. Loại *hoàn tiền ngay (`soan_hoan_tien`)* vì tem khớp 6L, hoàn tiền để dập lời dọa sẽ xử thua oan người bán đúng; loại *từ chối ngay (`tu_choi_khieu_nai`)* vì nồi vẫn có thể đúc sai dung tích thật so với tem, đóng đơn ngay sẽ tước quyền chứng minh của khách.
- **Loại lỗi & Cách sửa:** **Lỗi ngữ cảnh / cảm nhận** (sản phẩm đúng thông số nhưng lệch cảm nhận chủ quan của khách, kèm áp lực đe dọa). AI giữ giọng trung lập, không nhắc lại lời dọa hay phán xét khách sai, đồng thời vô hiệu hóa nút đề xuất hoàn tiền khi các thông số đang khớp.

### 3. Nguyên tắc HAX / PAIR áp dụng cụ thể

- **HAX G1 & G2 (Minh bạch năng lực — Luật AI 134/2025/QH15):** Dòng thông báo cố định đầu khung chat (*"Trợ lý AI hỗ trợ đối chiếu — Không tự quyết định tiền hay phạt shop"*) và nhãn trạng thái `Thiếu dữ liệu lúc đặt` (K04) / `Chưa đủ căn cứ` (K05) trong thẻ đối chiếu.
- **HAX G10 (Thu hẹp phạm vi khi nghi ngờ):** Thay vì tự phán xử thắng thua, câu trả lời ở cả K04 và K05 thu hẹp hành động về việc nêu rõ phần bằng chứng còn thiếu (ảnh chi tiết đơn hàng / video đong nước) và mở đường chuyển người thật.
- **PAIR — Explainability + Trust:** Thẻ đối chiếu 2 cột đặt cạnh nhau các dữ liệu trích xuất (`Trang hiện tại` vs `Ảnh khách gửi`; `Mô tả lúc đặt 6L` vs `Tem 6L`) ngay trên nút hành động để người dùng tự kiểm chứng căn cứ thay vì nhận câu từ chối chung chung.
- **HAX G8 & G9 (Gạt bỏ & sửa dễ dàng):** Nút **`Sửa thông tin`** (xử lý khi chọn nhầm đơn như K11) và **`Hủy khiếu nại`** đặt ngang hàng với nút gửi bằng chứng.

## §4. Mức tự chủ (chặng 4)

Với mỗi tool:

- Mức của nhóm (tự làm / cần người duyệt / cấm). Nếu cần người duyệt: ai duyệt · người duyệt thấy gì · quá hạn thì sao.
- Lý do theo cost-of-error: sai thì ai chịu gì · đảo lại được không · chờ người thì mất gì.
- Phương án đã cân nhắc và vì sao loại.
  Thêm:
- Tool khó xếp nhất: mỗi người từng chọn gì, cái giá của mỗi phía (trỏ về kịch bản ở §2), nhóm chốt thế nào.
- Thử mức đã đặt trên 2 kịch bản đắt nhất + 1 kịch bản bình thường: cái sai có bị chặn không? còn việc nào làm khi lẽ ra phải hỏi? người dùng bình thường có phải chờ vô ích không?
- Luật thêm hoặc tool tách bước (nếu có), và vì sao.
- Một mức tự chủ đã đổi lời hứa nào với người dùng.

## §5. Prototype (chặng 5)

- Link hoặc file · công cụ · phần nào mock · có lời gọi AI thật không.
- 1–2 ảnh có đánh số; mỗi số: thể hiện gì · nguyên tắc HAX/PAIR nào · mức tự chủ nào.
- Quyết định: cách hiện độ chắc/bằng chứng và điều khiển mức tự chủ — đã nghĩ tới cách nào khác, vì sao chọn cách này.

## §6. Tốt là gì (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)

- 3 tiêu chí Pass/Fail chấm được; thiết kế của nhóm đạt tiêu chí nào trên 2 kịch bản đắt nhất; lỗi còn lại.

## §7. Demo

- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.
