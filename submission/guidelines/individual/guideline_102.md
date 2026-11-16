# Guideline gán nhãn — v1 cá nhân — 102

- Phiên bản: v1 
- Ngày cập nhật: 2026-11-16.
- Loại nhãn: bounding box trên ảnh đường phố 2D.
- Công cụ gán nhãn: CVAT, xuất COCO 1.0.

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện người thật và phương tiện thật hiện diện trực tiếp trong cảnh đường phố. Phân biệt đối tượng thật với hình thể hiện của đối tượng: mô hình, tượng, hình in, quảng cáo, ma-nơ-canh, đồ chơi và ảnh phản chiếu.

## 2. Lớp và thuộc tính

### 2.1. Danh sách lớp

Mỗi đối tượng được chọn một lớp. `ignore_region` là vùng bỏ qua, không phải lớp đối tượng vật lý. Các tên lớp bổ sung dưới đây là tên đề xuất để đưa vào taxonomy ở Pha 2.

| Lớp | Định nghĩa và phạm vi trong bản cá nhân | Ví dụ | Cần phân biệt |
|---|---|---|---|
| `car` | Ô tô con. | Xe con trên xe tải ở `img_002.jpg`. | Xe mô hình ở `img_024.jpg`, xe tượng ở `img_027.jpg`, xe đồ chơi ở `img_038.jpg` không gán. |
| `truck` | Xe tải. | Xe chở xe tại `img_002.jpg`. | Box xe bao cả hàng theo R-04; các xe con được chở vẫn có box `car` riêng theo R-02. |
| `van` | Xe thùng kín cỡ nhỏ. |  |  |
| `motorcycle` | Xe máy, mô tô. |  | Tuk-tuk dùng lớp riêng; người điều khiển có box `person` riêng theo R-04. |
| `person` | Người thật; box bao cả hàng, túi xách, hành lý, xe đẩy tay hoặc đồ vật đang mang/đẩy cùng theo R-03. | Người gánh hàng rong ở `img_003.jpg`. | Người mô hình, tượng, chân dung quảng cáo và ma-nơ-canh không gán. |
| `bicycle` | Xe đạp thật; khi chở hàng, box bao cả xe và hàng theo R-04. | `img_003.jpg`, xe đạp chở hàng ở `img_010.jpg`. | Xích lô dùng lớp riêng; người điều khiển có box `person` riêng, không nhập vào phạm vi box xe. |
| `tuk_tuk`  | Phương tiện tuk-tuk, tách thành một lớp riêng thay vì ép vào lớp xe máy hoặc ô tô. | `img_004.jpg`, `img_016.jpg` theo ghi nhận của 102. | Không tự mở rộng lớp này cho mọi loại xe ba bánh; ranh giới với xích lô cần mô tả thêm. |
| `cyclo`  | Xích lô, dùng lớp riêng với xe đạp và tuk-tuk. | `img_023.jpg`. | Người lái có box `person` riêng; chưa chốt xử lý biến thể có động cơ và hành khách. |
| `ignore_region` | Vùng không cần gán từng vật. |  | Không dùng để thay cho quyết định loại tượng, mô hình hoặc phản chiếu. |

**Đối tượng ngoài các lớp đã định nghĩa:** ghi nhận loại đối tượng và ảnh để thảo luận ở Pha 2; không tự ép vào lớp gần giống. Danh sách bổ sung của 102 chưa giải quyết toàn bộ loại phương tiện có thể xuất hiện trong 40 ảnh.

### 2.2. Thuộc tính

| Thuộc tính | Giá trị | Cách dùng và trạng thái |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Theo tỉ lệ bị che: `0` = 0%; `1` = trên 0% đến dưới 50%; `2` = từ 50% đến dưới 80%; `3` = từ 80% trở lên. Xem thêm quy tắc đám đông tại R-08. |
| `truncated` | `no`, `yes` | `yes` nếu đối tượng bị cắt bởi biên ảnh; bị vật khác che trong ảnh không tự đồng nghĩa với cắt mép.  |
| `can_review` | `false`, `true` | Bật khi còn nghi ngờ về lớp, ranh giới box hoặc áp dụng quy tắc. Đây là cờ cần kiểm lại, không phải lý do để bỏ qua ca chưa giải quyết. |


## 3. Quy tắc vẽ box

### R-01 — Đối tượng thật và các hình thể hiện

Chỉ gán đối tượng thật hiện diện trực tiếp trong cảnh thuộc phạm vi đã chọn. Không gán người/xe mô hình, tượng, ảnh người trên quảng cáo, ma-nơ-canh hoặc xe đồ chơi thành các lớp người/phương tiện tương ứng.

- Ca áp dụng: `img_011.jpg`, `img_017.jpg`, `img_018.jpg`, `img_024.jpg`, `img_027.jpg`, `img_033.jpg`, `img_038.jpg`.
- Quyết định loại áp dụng cho đúng vật mô phỏng/hình in được ghi nhận; không suy ra toàn bộ ảnh đều không có đối tượng cần gán.
- Không tạo `ignore_region` chỉ để đánh dấu các vật đã bị loại theo quy tắc này.
- Nếu chưa xác định chắc thật hay mô phỏng, chuyển reviewer; không ghi thành ca đã được loại chắc chắn.

### R-02 — Xe tải chở xe con

Ở `img_002.jpg`, gán riêng từng xe con thật được chở trên xe tải bằng lớp `car`; không bỏ các xe này chỉ vì chúng đang là hàng vận chuyển. Xe tải vẫn được xem xét dưới lớp `truck`.

- Không gộp toàn bộ các xe con thành một box `car` duy nhất.
- Mỗi xe con có box riêng nếu không thuộc diện bỏ qua theo quy tắc che khuất và vật nhỏ tại R-08.
- Không suy đoán số xe, vị trí hoặc phần không nhìn thấy chỉ từ mô tả này.


### R-03 — Người mang hàng, hành lý hoặc đồ vật: box bao cả vật mang cùng

Ở `img_003.jpg`, giữ lớp `person` và vẽ một box bao cả người cùng hàng đang gánh. Áp dụng cùng cách làm cho túi xách, hành lý, xe đẩy tay và mọi đồ vật người đang mang hoặc đẩy cùng; không giới hạn ở người gánh hàng rong.

- Vẽ box chữ nhật nhỏ nhất bao được phần nhìn thấy của người và hàng, đòn gánh, túi xách, hành lý, xe đẩy tay hoặc đồ vật người đó đang mang/đẩy cùng.
- Hàng đặt rời bên cạnh không tự động được tính vào box người chỉ vì ở gần.
- Không tạo một box `person` khác cho riêng hàng hóa.
- Box có thể chứa khoảng nền trống giữa người và hai đầu gánh; cần phân biệt khoảng trống bắt buộc này với việc cố ý vẽ box lỏng.
- Không mở rộng quy tắc sang đồ vật đặt rời mà người đó không mang/đẩy cùng.

### R-04 — Phương tiện chở hàng: box bao cả hàng, người điều khiển có box riêng

Ở `img_010.jpg`, box `bicycle` bao cả xe đạp và hàng đang được xe chở. Áp dụng tương tự cho các phương tiện khác trong taxonomy, gồm xe máy, ô tô, xe tải, van, tuk-tuk và xích lô. Bổ sung lớp `bicycle` cũng áp dụng cho xe đạp được ghi nhận tại `img_003.jpg`.

- Vẽ box chữ nhật nhỏ nhất bao phần nhìn thấy của xe và hàng được gắn, buộc hoặc đặt trên xe để vận chuyển.
- Không gán hàng thông thường thành một phương tiện riêng; hàng đặt rời gần xe không tự động nhập vào box.
- Nếu hàng nhô cao hoặc vượt ra hai bên, ranh giới box phải bao được phần hàng đó.
- Người điều khiển có box `person` riêng theo phần nhìn thấy; không kéo ranh giới box xe chỉ để bao thêm người điều khiển. Hai box có thể chồng lấn do vị trí thực tế trong ảnh.
- Với xe tải chở xe con tại `img_002.jpg`, box xe tải bao cả cụm hàng đang chở theo quy tắc này, đồng thời từng xe con vẫn có box `car` riêng theo R-02. Đây là trường hợp hàng hóa bản thân là phương tiện thuộc taxonomy.

### R-07 — Không gán đối tượng chỉ xuất hiện qua phản chiếu

Ở `img_020.jpg`, không gán chiếc xe xuất hiện trong gương. Không tạo box `car` cho hình phản chiếu đó.

Phạm vi quy tắc là hình phản chiếu; nếu một đối tượng thật khác hiện diện trực tiếp trong ảnh thì vẫn xét độc lập. Việc khái quát sang kính cửa hàng, mặt nước hoặc các bề mặt phản chiếu khác cần được 102/nhóm xác nhận khi chốt đặc tả.

### R-08 — Phần nhìn thấy, vật nhỏ, đám đông và mức che khuất

- **Phần nhìn thấy:** box chỉ bao phần nhìn thấy, không suy ra phần bị che. Nếu một đối tượng có nhiều phần nhìn thấy rời nhau nhưng xác định được cùng một vật, dùng một box nhỏ nhất bao các phần đó; không tạo nhiều box cho cùng một vật. Với R-03/R-04, bao phần nhìn thấy của cả đối tượng và đồ vật/hàng được tính cùng.
- **Vật nhỏ:** vật có chiều cao box phần nhìn thấy dưới 20 px không được gán box theo lớp đối tượng; đánh dấu vùng chứa vật bằng `ignore_region`. Vùng ignore ôm sát phần nhìn thấy cần bỏ qua, tránh phủ sang đối tượng khác còn đủ điều kiện gán. Chiều cao đúng 20 px không bị loại bởi ngưỡng này.
- **Đám đông:** khi phần nhìn thấy của một người nhỏ hơn 20% toàn bộ người đó, không gán box `person` riêng cho người đó; đánh dấu vùng tương ứng bằng `ignore_region`. Người có phần nhìn thấy từ 20% trở lên vẫn xét gán riêng nếu đáp ứng ngưỡng kích thước và nhận dạng được. Không bỏ qua toàn bộ đám đông chỉ vì một vài người bị che nhiều.
- **Mức che khuất:** dùng tỉ lệ bị che để chọn `occluded_level`: `0` khi không bị che; `1` khi bị che trên 0% và dưới 50%; `2` khi bị che từ 50% đến dưới 80%; `3` khi bị che từ 80% trở lên. Như vậy đúng 50% thuộc mức `2`, đúng 80% thuộc mức `3`.
- Ngưỡng đám đông xét phần **nhìn thấy**, còn `occluded_level` xét phần **bị che**. Người trong đám đông có đúng 20% nhìn thấy vẫn có thể được gán mức `3`; dưới 20% nhìn thấy thì chuyển sang `ignore_region` theo quy tắc trên.

Tỉ lệ che khuất cần được ước lượng từ hình thể đối tượng, không thể tính trực tiếp chỉ từ diện tích box phần nhìn thấy. Khi box bao cả hàng theo R-03/R-04, cách chọn mẫu số để ước lượng che khuất của riêng người/xe hay cả cụm vẫn cần thống nhất. Cách xử lý xe chủ/ego và hành khách cũng chưa chốt. Giữ nguyên ngưỡng vật nhỏ 20 px trong taxonomy.

## 4. Ca khó và quyết định ban đầu

Các mô tả ảnh dưới đây lấy từ khảo sát do 102 cung cấp; bảng không thay thế bước đối chiếu ảnh gốc khi vẽ box. Lý do là phần diễn giải từ quyết định, chưa phải biên bản đồng thuận của nhóm.

| Ảnh | V0 chưa trả lời rõ / tình huống | Rule ID | Quyết định ban đầu của 102 | Lý do và phần cần chú ý |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải đang chở nhiều xe con; chưa rõ xe được chở có còn là đối tượng cần gán. | R-02, R-04, R-08 | Gán riêng các xe con ở trên xe tải; box xe tải bao cả hàng đang chở. | Xe con vẫn là phương tiện thật; các box có thể chồng lấn. Chỉ bao phần nhìn thấy và áp dụng ngưỡng vật nhỏ. |
| `img_003.jpg` | Thiếu lớp xe đạp. | R-04 | Thêm lớp `bicycle` và gán xe đạp theo lớp này. | Tránh bỏ sót hoặc ép xe đạp vào lớp xe máy. |
| `img_003.jpg` | Người gánh hàng rong: box chỉ bao người hay cả hàng? | R-03 | Box `person` bao cả người và hàng đang gánh. | Áp dụng tương tự cho túi xách, hành lý, xe đẩy tay và đồ vật người đang mang/đẩy cùng; mẫu số tính che khuất cần thống nhất thêm. |
| `img_004.jpg` | Tuk-tuk chưa có lớp riêng trong v0. | Mục 2.1 | Thêm lớp `tuk_tuk`. | Cần định nghĩa riêng thay vì gán vào lớp phương tiện gần giống. |
| `img_016.jpg` | Tuk-tuk theo ghi nhận của 102; v0 thiếu lớp tương ứng. | Mục 2.1 | Dùng lớp bổ sung `tuk_tuk`. | Đối chiếu ảnh gốc khi gán và thống nhất tiêu chí phân biệt ở Pha 2. |
| `img_010.jpg` | Xe đạp chở hàng: box bao riêng xe hay cả hàng? | R-04 | Box `bicycle` bao cả xe và hàng đang chở; người điều khiển có box `person` riêng. | Áp dụng cùng cách bao hàng cho các xe khác; không kéo box xe để bao thêm người điều khiển. |
| `img_011.jpg` | Xe mô hình và người mô hình trông giống các lớp cần phát hiện. | R-01 | Không gán các mô hình đó. | Vật mô phỏng nằm ngoài phạm vi người/phương tiện thật. |
| `img_017.jpg` | Tượng có hình dạng giống đối tượng thật. | R-01 | Không gán tượng. | Không đồng nhất hình dạng người/phương tiện với đối tượng thật. |
| `img_018.jpg` | Ảnh người trên bảng quảng cáo. | R-01 | Không gán chân dung quảng cáo thành `person`. | Đây là hình in, không phải người hiện diện trực tiếp trong cảnh. |
| `img_020.jpg` | Xe xuất hiện trong gương. | R-07 | Không gán chiếc xe trong gương. | Loại hình phản chiếu khỏi phạm vi; không có nghĩa bỏ mọi xe khác trong ảnh. |
| `img_023.jpg` | Xích lô chưa có lớp phù hợp. | Mục 2.1, R-04 | Thêm lớp `cyclo`; người điều khiển có box `person` riêng. | Tách khỏi xe đạp và tuk-tuk; cách xử lý hành khách cần chốt thêm. |
| `img_024.jpg` | Xe car mô hình theo ghi nhận của 102. | R-01 | Không gán xe mô hình đó. | Ngoài phạm vi phương tiện thật; không tự loại các đối tượng thật khác trong ảnh. |
| `img_027.jpg` | Xe được thể hiện bằng tượng. | R-01 | Không gán xe tượng. | Không dùng lớp `car`/`truck` cho một vật chỉ mô phỏng phương tiện. |
| `img_033.jpg` | Ma-nơ-canh có hình người. | R-01 | Không gán ma-nơ-canh thành `person`. | Ma-nơ-canh không phải người thật. |
| `img_038.jpg` | Xe đồ chơi. | R-01 | Không gán xe đồ chơi. | Đồ chơi nằm ngoài phạm vi phương tiện thật. |

### Các điểm cần mang ra họp Pha 2

1. **Định nghĩa đối tượng khi bao hàng:** box người bao cả đồ vật mang/đẩy cùng, box phương tiện bao cả hàng. Nhóm cần chấp nhận rõ quy ước này hoặc ghi nhận quyết định nhóm khác với bản cá nhân; không âm thầm đổi lại bản cá nhân sau họp.
2. **Hàng bản thân là phương tiện:** R-04 bao hàng trong box xe chở, đồng thời R-02 vẫn gán riêng các xe con được chở; nhóm cần áp dụng nhất quán cách biểu diễn có box chồng lấn này.
3. **Người và phương tiện:** người điều khiển đã có box `person` riêng; cách xử lý hành khách trên phương tiện và xe chủ/ego còn cần chốt.
4. **Lớp mới:** thống nhất các tên `bicycle`, `tuk_tuk`, `cyclo`, tiêu chí phân biệt và việc cần thêm lớp khác khi khảo sát đủ bộ ảnh.
5. **Thuộc tính:** các mức che khuất đã có ngưỡng ở R-08; còn cần thống nhất mẫu số ước lượng khi box bao cả hàng và cách xét `truncated` nếu chỉ hàng bị cắt mép.
6. **Vật nhỏ và đám đông:** đối chiếu cách vẽ vùng `ignore_region` theo ngưỡng dưới 20 px hoặc phần người nhìn thấy dưới 20% ở R-08, tránh che phủ đối tượng khác vẫn đủ điều kiện gán.

Nếu có box nhưng chưa chắc, bật `can_review`. Nếu chưa thể vẽ box, ghi tên ảnh, vùng và lý do để tổng hợp vào `submission/qc/image_issues.csv` sau khi khóa nhãn độc lập. Không biến ca chưa quyết định thành ca “không gán” để tránh phải review.

## 5. QC và phiên bản

| Hạng mục | Ngưỡng / cách áp dụng |
|---|---|
| Vật nhỏ | Chiều cao phần nhìn thấy dưới 20 px: không gán box lớp đối tượng, đánh dấu vùng `ignore_region` theo R-08. |
| Ghép box để đo IAA | Ghép 1–1, không phân biệt lớp, IoU tối thiểu 0,5. |
| Kiểm độ khớp box | Box cao từ 50 px: IoU ≥ 0,85; box nhỏ hơn: IoU ≥ 0,7. |
| IAA trước calibration | IoU trung bình ≥ 0,85 và đồng thuận lớp ≥ 95%, chỉ trên các cặp box ghép được. Không có cặp thì ghi N/A. |
| Độ phủ phân công | Đủ 100% lượt gán hợp lệ trước khi đo IAA. |
| QC bàn giao | Duyệt toàn bộ 40 ảnh; tỉ lệ ảnh lỗi ≤ 2%, không có lỗi lặp và đủ bằng chứng để reviewer xác nhận Release. Một ảnh lỗi trên 40 ảnh là 2,5%, vượt ngưỡng. |



