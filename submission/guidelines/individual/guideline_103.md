# Guideline gán nhãn — bản cá nhân — 103

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v0 cá nhân
- Ngày: 2026-11-16
- Người viết / duyệt: 103 / chưa review

## 1. Mục tiêu
Nhãn bounding box 2D dùng để huấn luyện mô hình phát hiện người và phương tiện trên ảnh đường phố. Cần hạn chế bỏ sót người, nhầm lớp phương tiện và vẽ box sai nhiều; đặc biệt lưu ý ảnh có người hoặc phương tiện bị che khuất.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con dùng chở người. | Các ô tô con được chở ở `img_002.jpg`. | Xe chở ô tô ở `img_002.jpg` là `truck`, không phải `car`. |
| `truck` | Xe tải dùng chở hàng hoặc phương tiện. | Xe tải ở `img_007.jpg`. | Xe buýt ở `img_009.jpg` thuộc `bus`. |
| `van` | Xe thùng kín cỡ nhỏ. | Xe van ở `img_005.jpg`; xe bán tải nắp thùng cao ở `img_037.jpg` được xếp `van`. | Xe tải lớn ở `img_007.jpg` thuộc `truck`. |
| `motorcycle` | Xe hai bánh có động cơ. | Xe máy ở `img_006.jpg`. | Xe đạp không động cơ ở `img_029.jpg` thuộc `bicycle`. |
| `person` | Người thật trong ảnh, không phải hình tượng/hình ảnh mô phỏng. | Người đi xe máy ở `img_005.jpg`. | Tượng ở `img_017.jpg` không gán là `person`. |
| `bus` | Lớp riêng cho xe cơ giới thiết kế để chở nhiều hành khách. | Xe buýt ở `img_009.jpg`. | Xe tải chở hàng ở `img_007.jpg`. |
| `tuk_tuk` | Lớp riêng cho xe ba bánh có động cơ dùng chở người/hàng. | Tuk-tuk ở `img_004.jpg`. | Xe máy hai bánh ở `img_006.jpg`. |
| `bicycle` | Xe đạp không có động cơ. | Xe đạp ở `img_029.jpg`. | Xe máy có động cơ ở `img_006.jpg`. |
| `cyclo` | Xích lô/xe đạp ba bánh không động cơ dùng chở người. | Xích lô ở `img_023.jpg`. | Tuk-tuk có động cơ ở `img_004.jpg`. |
| `ignore_region` | Vùng cụm đông/chồng lấp không thể tách từng vật để gán riêng; chỉ khoanh phần không phân biệt được. | Phần chồng lấp dày trong đám đông ở `img_021.jpg` (chọn đúng vùng khi gán). | Người nhìn rõ, tách được như người đi bộ ở `img_034.jpg`. |

Vật không thuộc các lớp đã liệt kê thì không gán; nếu chưa chắc vật có thuộc phạm vi hay không, đánh dấu `can_review` và ghi lý do. Không dùng `ignore_region` thay cho một vật đơn lẻ có thể nhận diện.

### Thuộc tính
| Thuộc tính | Mức | Định nghĩa |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che. |
| `occluded_level` | `1` | Bị che 1–30% diện tích vật. |
| `occluded_level` | `2` | Bị che 31–70% diện tích vật. |
| `occluded_level` | `3` | Bị che trên 70% nhưng vẫn nhận dạng được. |
| `truncated` | `no` / `yes` | `yes` nếu mép ảnh cắt mất một phần vật; `no` nếu vật nằm trọn trong ảnh. |
| `can_review` | `false` / `true` | Tick khi chưa thể quyết định chắc chắn sau khi áp dụng guideline; ghi lý do để reviewer xem. |

## 3. Quy tắc vẽ box
Box ôm sát phần nhìn thấy. Khi vật bị che hoặc bị mép ảnh cắt, box chỉ bao phần vật nhìn thấy; ghi `occluded_level` hoặc `truncated` tương ứng. Người đang đi/ngồi trên xe máy được gán bằng hai box riêng: một box `person` và một box `motorcycle` (**R-01**); không gộp người với xe vào một box. Với xe chở xe, gán một box cho xe chở và một box riêng cho từng ô tô được chở nhìn thấy (ví dụ `img_002.jpg`, **R-02**). Vật đơn lẻ có chiều cao dưới 20 px không gán box riêng và không tạo `ignore_region`; với cụm đông/chồng lấp không thể tách, chỉ khoanh vùng không phân biệt được bằng `ignore_region`, các vật nhìn rõ xung quanh vẫn gán riêng (**R-03**). Không gán tượng, ma-nơ-canh, đồ chơi, hình in hoặc phản chiếu như người/phương tiện thật (**R-04**). Dùng lớp riêng `bus`, `tuk_tuk`, `bicycle` và `cyclo` theo bảng lớp (**R-05**); xe bán tải nắp thùng cao ở `img_037.jpg` gán `van` (**R-06**). Chỉ ở `img_035.jpg`, box tuk-tuk bao gồm cả hàng hóa cồng kềnh chở trên xe (**R-07**); không tự áp dụng quy tắc này cho xe khác, ca chưa rõ thì dùng `can_review`. Xe diễu hành hình ô tô ở `img_019.jpg` và ô tô thật đang trưng bày ở `img_024.jpg` vẫn gán `car` (**R-08**).

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_005.jpg`, `img_006.jpg` | Người đi/ngồi trên xe máy | `R-01` | Một box `person` và một box `motorcycle` riêng | Không gộp hai đối tượng thành một box. |
| `img_002.jpg` | Xe tải chở nhiều ô tô | `R-02` | Gán box xe chở và box riêng từng ô tô nhìn thấy | Các phương tiện nhìn thấy là các vật riêng biệt. |
| `img_004.jpg`, `img_016.jpg` | Tuk-tuk/xe ba bánh có động cơ | `R-05` | Gán lớp riêng `tuk_tuk` | Phân biệt với xe máy hai bánh. |
| `img_035.jpg` | Tuk-tuk chở hàng hóa cồng kềnh | `R-07` | Box xe bao gồm cả xe và hàng hóa trên xe | Quy tắc này chỉ áp dụng cho ca `img_035.jpg`; hàng không có box riêng. |
| `img_009.jpg` | Xe buýt | `R-05` | Gán lớp riêng `bus` | Phân biệt xe chở khách với xe tải chở hàng. |
| `img_017.jpg`, `img_027.jpg`, `img_028.jpg` | Tượng có hình người/xe | `R-04` | Không gán là người/phương tiện thật | Đây là tượng, không phải đối tượng thật cần phát hiện. |
| `img_020.jpg` | Phản chiếu trong gương | `R-04` | Không gán phản chiếu như đối tượng thật | Đối tượng chỉ xuất hiện trong phản chiếu. |
| `img_018.jpg`, `img_033.jpg`, `img_038.jpg` | Hình người in, ma-nơ-canh và xe đồ chơi | `R-04` | Không gán là người/phương tiện thật | Đây là hình ảnh/vật mô phỏng, không phải người/phương tiện thật. |
| `img_021.jpg` | Đám đông phương tiện/người chồng lấp | `R-03` | Chỉ dùng `ignore_region` cho phần không thể tách; vật rõ vẫn gán riêng | Tránh tạo box riêng thiếu tin cậy; vật đơn lẻ dưới 20 px thì bỏ qua. |
| `img_029.jpg` | Xe đạp không động cơ | `R-05` | Gán lớp riêng `bicycle` | V0 chưa có lớp phù hợp. |
| `img_023.jpg` | Xích lô không động cơ | `R-05` | Gán lớp riêng `cyclo` | Phân biệt với tuk-tuk có động cơ. |
| `img_037.jpg` | Xe bán tải có nắp thùng cao | `R-06` | Gán `van` | Áp dụng định nghĩa xe thùng kín cỡ nhỏ của v0. |
| `img_019.jpg`, `img_024.jpg` | Xe hình ô tô diễu hành và ô tô thật trưng bày | `R-08` | Gán `car` | Gán theo loại phương tiện, dù không chạy trong giao thông thường ngày. |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản
Ngưỡng lab áp dụng: độ phủ lượt gán phải đủ 100% trước khi tính IAA; box được ghép 1–1 khi IoU từ `0.5`, IoU trung bình mục tiêu ít nhất `0.85` và đồng thuận lớp ít nhất `95%` (chỉ tính trên các cặp ghép). QC bàn giao phải duyệt toàn lô; tỷ lệ ảnh lỗi không quá `2%`, lỗi lặp cần Rework. Số liệu luôn ghi tử số/mẫu số. Script không phát hiện được lỗi cả nhóm cùng mắc hoặc đối tượng cả nhóm cùng bỏ sót nên reviewer phải đối chiếu ảnh gốc và guideline. Người gán đánh dấu `can_review` và ghi vấn đề; nhóm/reviewer chốt ở calibration. Các ngưỡng QC theo `docs/QC_CHEATSHEET.md`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0 cá nhân` | Bổ sung định nghĩa lớp, ngưỡng thuộc tính và quy tắc `R-01` đến `R-08` từ guideline v0. | Các ca khó khảo sát trong 40 ảnh, gồm người đi xe máy, xe chở xe, phương tiện đặc thù, vật mô phỏng và vùng đông. |
