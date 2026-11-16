# Guideline gán nhãn — v1.0 (thống nhất) — Group79

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Nguồn: hợp nhất `individual/guideline_101.md`, `guideline_102.md`, `guideline_103.md`
- Người tổng hợp / duyệt: 101 / 103
- Trạng thái: chờ cả nhóm (101, 102, 103) xác nhận tại họp CP2. Sau khi xác nhận, **khóa** bản này và `taxonomy.json` v1.0 cho toàn bộ lượt gán độc lập ở Pha 3.

## 1. Mục tiêu

- Nhãn bounding box 2D dùng để huấn luyện mô hình **phát hiện người thật và phương tiện thật** hiện diện trực tiếp trong cảnh đường phố (camera giao thông, xe tự lái).
- Hai lỗi nguy hiểm nhất:
  - **Bỏ sót** người hoặc xe thật: mô hình học cách bỏ qua vật thật trên đường.
  - **Gán vật mô phỏng** (tượng, mô hình, ma-nơ-canh, đồ chơi, hình in, phản chiếu): mô hình học báo động giả.
- Ngoài phạm vi: phân đoạn pixel, nhận dạng danh tính người, biển số, hàng hóa.

## 2. Lớp và thuộc tính

### Lớp

Mỗi vật thuộc đúng một lớp. Lớp được chọn theo **loại vật**, không theo việc vật đang chạy hay đang đỗ.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người, kể cả SUV và xe wagon. | Từng xe con trên sàn `img_002`; `img_015`; xe đen thật ở `img_024`; xe diễu hành cỡ thật `img_019` | Tượng xe `img_027`; xe đồ chơi `img_038`; xe mô hình trên nóc xe `img_024` |
| `truck` | Xe có thùng hoặc sàn hàng **tách riêng khỏi cabin**, kể cả xe tải, xe chở xe và **xe bán tải** (R-12). | Xe chở xe `img_002`; `img_007`; `img_036`; bán tải `img_001`, `img_012`, `img_028`, `img_037` | `van` (khoang hàng liền cabin); `bus`; tượng bán tải `img_027` |
| `van` | Xe thùng kín cỡ nhỏ, **khoang hàng liền một khối với cabin**. | `img_005` | Bán tải có nắp thùng `img_037` là `truck` |
| `bus` | Xe cơ giới thiết kế để chở nhiều hành khách (xe buýt, xe khách). | `img_009`, `img_031`, `img_037`, `img_040` | `truck` chở hàng `img_007` |
| `tuk_tuk` | Xe ba bánh **có động cơ** chở khách / hàng (tuk-tuk, auto rickshaw). | `img_004`, `img_016`, `img_035` | `motorcycle` (hai bánh); `cyclo` (không động cơ) |
| `motorcycle` | Xe hai bánh có động cơ, kể cả xe có thùng bên (sidecar). | `img_005`, `img_006`, `img_022`, `img_026`, `img_029`, `img_030` | `bicycle` (không động cơ); `tuk_tuk` (ba bánh) |
| `bicycle` | Xe đạp hai bánh **không động cơ**. | `img_003`, `img_010` | `motorcycle`; `cyclo` |
| `cyclo` | Xích lô / xe đạp ba bánh **không động cơ** chở người. | `img_023`; xích lô ở `img_006`, `img_009`, `img_034`, `img_039` | `tuk_tuk` (có động cơ); `bicycle` (hai bánh) |
| `person` | Người thật hiện diện trong cảnh: đi bộ, đứng, hoặc ngồi / lái trên xe. | `img_003`, `img_005`, `img_010`, `img_014`, `img_026` | Tượng `img_017`; ma-nơ-canh `img_033`; chân dung trên biển quảng cáo `img_018`; nhân vật mô hình `img_011`; người trong gương |
| `ignore_region` | Vùng không gán từng vật: vật < 20 px, hoặc phần cụm đông không tách được. Bỏ sót trong vùng này không tính lỗi. | Cụm đông chồng lấp `img_021`; người đi bộ rất nhỏ `img_025` | Không dùng để che vật đủ lớn mà mình chưa chắc (khi đó tick `can_review`); không dùng để đánh dấu tượng / mô hình |

**Vật ngoài taxonomy** (xe đẩy tay, động vật, hàng hóa…): không gán, không tự ép vào lớp gần giống. Chưa chắc vật có thuộc lớp nào không: vẽ box theo lớp gần nhất, tick `can_review` và ghi lý do. Chưa vẽ được box thì ghi vấn đề cấp ảnh.

Phép thử lớp / thuộc tính: *"mô hình 2D có phải đoán ra điều này khi chạy không?"* `bus`, `tuk_tuk`, `bicycle`, `cyclo` khác hình dạng rõ với `car` / `motorcycle`, và mô hình cần phân biệt chúng, nên là **lớp**. `occluded_level`, `truncated`, `can_review` chỉ dùng để lọc mẫu và QC, nên là **thuộc tính**.

### Thuộc tính

| Thuộc tính | Mức | Nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che. |
| | `1` | Bị che **> 0% đến 30%** diện tích vật. |
| | `2` | Bị che **> 30% đến 70%**. |
| | `3` | Bị che **> 70%** nhưng vẫn nhận ra chắc chắn lớp. Không nhận ra được thì không gán (trong cụm đông: xem R-07). |
| `truncated` | `no` / `yes` | `yes` khi box chạm mép ảnh (cách mép ≤ 2 px) **và** vật rõ ràng còn kéo dài ra ngoài khung, bất kể mất bao nhiêu. Còn lại `no`. |
| `can_review` | `false` / `true` | Bật khi đã vẽ box nhưng còn nghi ngờ về lớp, ranh giới hoặc cách áp dụng quy tắc. Đây là cờ để reviewer kiểm lại, không phải lý do để bỏ qua ca. |

- Mẫu số để ước lượng % che là **bản thân vật** (thân người / thân xe), không tính hàng hóa (vì hàng không nằm trong box, R-05).
- Phần mất do **mép ảnh** chỉ tính vào `truncated`, không tính vào `occluded_level`.
- Dùng thuộc tính `occluded_level` trong panel Objects. **Không dùng** cờ `occluded` có sẵn của CVAT (phím `Q`), vì script bỏ qua cờ này.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| R-01 | **Phần nhìn thấy:** box chữ nhật nhỏ nhất ôm sát phần nhìn thấy của vật, không đoán phần bị che. Một vật có nhiều phần nhìn thấy rời nhau (bị cột, người che ngang) vẫn chỉ có **1 box** bao các phần đó. |
| R-02 | **Vật không thật:** không gán tượng, mô hình, ma-nơ-canh, đồ chơi, hình in (biển quảng cáo, poster), và người / xe trong phản chiếu (gương, kính, mặt nước). Vật thật khác trong cùng ảnh vẫn gán bình thường. Không tạo `ignore_region` cho vật bị loại theo quy tắc này. Ảnh chỉ có vật mô phỏng vẫn giữ trong Task, với 0 box. |
| R-03 | **Người trên xe** (xe máy, xe đạp, xích lô, tuk-tuk): mỗi người (lái và chở) **1 box `person`**, xe **1 box riêng** theo lớp xe. Hai box được phép chồng lấn. Box `person` bao cả mũ bảo hiểm và phần chân nhìn thấy. Box xe bao xe từ đầu tới đuôi; người ngồi trên xe **không tính** là vật che khi chấm `occluded_level` của xe. |
| R-04 | **Xe chở xe** (`img_002`): **1 box `truck`** bao đầu kéo và khung sàn chở, cộng **1 box `car` cho mỗi xe con** nhìn thấy trên sàn (không gộp nhiều xe con vào một box). Xe con bị khung sàn che thì chấm `occluded_level`. Xe con < 20 px áp dụng R-06. |
| R-05 | **Hàng hóa và đồ mang theo:** box xe chỉ bao **thân xe**, không bao phần hàng nhô ra ngoài thân xe (`img_035` tuk-tuk chất hàng trên nóc, `img_029` xe máy chở hàng, `img_010` xe đạp chở hàng). Hàng nằm gọn trong thùng xe (`img_012`) thì tự nằm trong box xe. Box `person` bao thân người và đồ đội / mặc / đeo sát người; **không** kéo ra để bao đòn gánh, thúng hàng, xe đẩy (`img_003`). Hàng không có box riêng. `truncated` chỉ xét thân vật, không xét hàng. |
| R-06 | **Vật nhỏ:** vật có chiều cao phần nhìn thấy **< 20 px** (`rules.min_box_height_px`) không vẽ box theo lớp. Khoanh vùng chứa vật bằng `ignore_region` ôm sát, không phủ sang vật khác còn đủ điều kiện gán. Vật cao **đúng 20 px** vẫn gán box bình thường. |
| R-07 | **Đám đông / cụm chồng lấp:** chỉ khoanh `ignore_region` cho **phần không tách được** từng vật. Vật nhìn rõ xung quanh vẫn gán riêng. Trong cụm, vật chỉ nhìn thấy < 20% (bị che > 80%) thì nằm trong vùng ignore, không vẽ box riêng. Không bỏ cả đám đông chỉ vì vài người bị che nhiều. |
| R-08 | **Xe chủ (ego):** không gán phần đầu xe / gương / thân của xe đang chụp ảnh. |
| R-09 | **Ảnh mờ do chuyển động hoặc mưa** (`img_014`, `img_015`): vẫn gán nếu nhận ra lớp. Box ôm phần thân nhận ra được, không bao vệt mờ kéo dài. Không xác định được ranh giới thì tick `can_review`. |
| R-10 | **Xe diễu hành / xe trưng bày cỡ thật** (`img_019`, xe đen ở `img_024`): là phương tiện thật nên gán theo lớp có hình dạng tương ứng (thường là `car`), dù không chạy trong giao thông thường. Xe mô hình / đồ chơi cỡ nhỏ thì áp dụng R-02. |
| R-11 | **Mô hình / vật trang trí gắn trên xe** (xe mô hình đặt trên nóc / capo ở `img_024`, vật trang trí trên bán tải `img_028`): không gán; chỉ gán chiếc xe thật mang nó. Box xe không bao vật trang trí nhô ra. |
| R-12 | **Bán tải (pickup):** gán `truck`, kể cả khi thùng có nắp cao (`img_037`, Master 75 Truck Cap). Lý do: thùng hàng tách khỏi cabin, đúng định nghĩa `truck`. Xe có khoang hàng liền một khối với cabin mới là `van`. |

## 4. Ca khó

### 4.1. Quyết định theo ảnh

| Ảnh | Tình huống | Rule | Quyết định |
|---|---|---|---|
| `img_002` | Đầu kéo chở nhiều ô tô trên 2 sàn | R-04 | 1 `truck` + 1 `car` cho mỗi xe con nhìn thấy |
| `img_003` | Người gánh hàng rong; xe đạp | R-03, R-05 | `person` chỉ bao thân người, không bao đòn gánh / thúng; xe đạp → `bicycle` |
| `img_004`, `img_016` | Tuk-tuk | lớp | `tuk_tuk` |
| `img_035` | Auto rickshaw chở quá tải, hàng chất trên nóc | R-05 | `tuk_tuk`, box chỉ bao thân xe, không bao khối hàng |
| `img_010` | Xe đạp chở hàng | R-03, R-05 | `bicycle` bao thân xe (không bao hàng nhô ra); người đạp → `person` riêng |
| `img_029` | Xe máy chở hàng cồng kềnh | R-03, R-05 | `motorcycle` bao thân xe; người lái → `person` riêng |
| `img_005`, `img_026` | Người đi xe máy | R-03 | Mỗi người 1 `person` + mỗi xe 1 `motorcycle` |
| `img_006`, `img_023` | Xích lô (tour Huế) | lớp, R-03 | Xích lô → `cyclo`; người ngồi / đạp → `person`; xe máy, ô tô thật trong cảnh vẫn gán |
| `img_037` | Bán tải gắn nắp thùng cao; có xe buýt | R-12 | Bán tải → `truck`; xe buýt → `bus` |
| `img_028` | Bán tải thật có vật trang trí | R-11, R-12 | Xe → `truck`; vật trang trí không gán |
| `img_024` | Xe wagon đen thật, trên nóc / capo có xe mô hình | R-10, R-11 | Xe đen → `car`; xe mô hình phía trên không gán |
| `img_019` | Xe diễu hành hình ô tô, cỡ thật | R-10 | `car` |
| `img_011` | Nhân vật Predator và xe đều là mô hình | R-02 | Không gán |
| `img_017` | Tượng người | R-02 | Không gán |
| `img_027` | Tượng xe bán tải | R-02 | Không gán |
| `img_033` | Ma-nơ-canh | R-02 | Không gán |
| `img_038` | Tủ kính đồ chơi, xe mô hình | R-02 | Không gán |
| `img_018` | Chân dung trên biển quảng cáo | R-02 | Không gán chân dung; người thật đứng gần (nếu có) vẫn gán |
| `img_020` | Xe trong gương chiếu hậu | R-02, R-08 | Không gán xe trong gương, không gán xe chủ |
| `img_021` | Cụm người / xe chồng lấp | R-07 | `ignore_region` cho phần không tách được; vật rõ vẫn gán riêng |
| `img_025` | Người đi bộ rất nhỏ ở xa | R-06 | Người < 20 px → `ignore_region` |
| `img_014`, `img_015` | Ảnh mờ / mưa | R-09 | Gán nếu nhận ra lớp; không rõ ranh giới → `can_review` |

### 4.2. Bất đồng giữa các bản cá nhân và cách giải quyết

| Vấn đề | 101 | 102 | 103 | Chốt v1.0 | Lý do |
|---|---|---|---|---|---|
| Hàng chở cồng kềnh | Không bao hàng | Bao cả hàng (mọi xe, cả người gánh) | Chỉ riêng `img_035` bao hàng | **Không bao hàng** (R-05) | Một quy tắc chung cho mọi xe và người, dễ áp dụng nhất quán hơn ngoại lệ theo từng ảnh. Box ôm thân vật ổn định hơn nên IoU giữa người gán cao hơn. Hàng không phải lớp mô hình cần phát hiện. Khớp quyết định mẫu trong tài liệu lab cho `img_035`. |
| Bán tải, `img_037` | `truck` | — | `van` | **`truck`** (R-12) | v0: `truck` = "xe có thùng hàng riêng". Thùng bán tải tách khỏi cabin; nắp thùng chỉ là phụ kiện. |
| Xe đạp, xích lô | Không gán xe | Thêm `bicycle`, `cyclo` | Thêm `bicycle`, `cyclo` | **Thêm 2 lớp** | Đa số 2/3. Tránh ép xe không động cơ vào `motorcycle`. Mô hình cần phân biệt chúng với xe máy / tuk-tuk. |
| Vật đơn lẻ < 20 px | `ignore_region` | `ignore_region` | Không gán gì | **`ignore_region`** (R-06) | Đa số 2/3, và theo `docs/QC_CHEATSHEET.md`: vật nhỏ → `ignore_region`. Tránh bị tính bỏ sót khi so IAA. |
| Ngưỡng `occluded_level` | 30 / 70% | 50 / 80% | 30 / 70% | **30 / 70%** | Đa số 2/3; khớp ví dụ trong tài liệu lab. Lấy cách ghi biên chặt của 102 (> 0, > 30, > 70) để không mơ hồ ở đúng 30% / 70%. |
| Lớp `bus` | Thêm | Không nêu | Thêm | **Thêm** | Đa số 2/3. Xe buýt khác `truck` rõ về hình dạng và mục đích. |
| `img_024` | Xe thật, vỏ xe trên nóc không gán | Xe mô hình, không gán | Xe thật → `car` | **Xe đen → `car`; xe mô hình trên nóc không gán** (R-10, R-11) | Đối chiếu trên ảnh: xe bên dưới là xe thật, xe phía trên là mô hình. |
| `img_011` | Có điều kiện (tượng hay người hóa trang) | Người và xe đều là mô hình | Không nêu | **Không gán** (R-02) | Đối chiếu trên ảnh: cả nhân vật và xe đều là mô hình. |
| `img_029` | `motorcycle` | — | `bicycle` | **`motorcycle`** | Đối chiếu trên ảnh: xe có động cơ (tên ảnh gốc: "load on a motorbike"). |
| `img_028` | Bán tải thật → `truck` | — | Xếp vào nhóm tượng | **Xe thật → `truck`** (R-11, R-12) | Đối chiếu trên ảnh: xe là xe thật; chỉ vật trang trí trên xe là không gán. |
| Xe chủ, hành khách, ảnh mờ, `truncated` | Có quy tắc | Để mở | Không nêu | **Theo 101** (R-03, R-08, R-09, thuộc tính) | Lấp các chỗ 102 ghi "chưa chốt"; không bản nào phản đối. |
| Nhiều phần rời của một vật | Không nêu | 1 box | Không nêu | **1 box** (R-01) | Tránh đếm một vật thành nhiều vật. |

## 5. QC và phiên bản

- Ngưỡng QC của lab (mục `rules` trong `taxonomy.json`, **không đổi**):
  - Ghép box 1–1, không phân biệt lớp, IoU ≥ 0.5.
  - Kiểm độ khớp box: IoU ≥ 0.85 với box cao ≥ 50 px; ≥ 0.7 với box nhỏ hơn.
  - IAA trước calibration: IoU trung bình ≥ 0.85 và cùng lớp ≥ 95%, chỉ tính trên các cặp ghép được; không có cặp thì ghi N/A.
  - Độ phủ phải đủ 100% lượt gán trước khi đo IAA.
  - QC bàn giao: duyệt toàn bộ 40 ảnh; tỉ lệ ảnh lỗi ≤ 2% (1/40 = 2.5% đã vượt); có lỗi lặp → Rework.
  - Vật nhỏ: cao < 20 px.
- Trong lượt gán độc lập (Pha 3): **không hỏi nhau**. Ca mơ hồ thì tick `can_review` trên box; chưa vẽ được box thì ghi `ảnh, vùng, lý do` để gom vào `submission/qc/image_issues.csv` sau khi khóa nhãn.
- Ca mơ hồ được nhóm chốt ở calibration (Pha 5). Reviewer cuối: **103**. Mỗi quyết định dẫn mã quy tắc R-xx.
- Mọi thay đổi làm đổi nhãn đều tăng phiên bản: đổi câu chữ → `editorial`; đổi cách chọn vật, lớp, box hoặc thuộc tính → `semantic`, phải kiểm lại cả lô.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên: 5 lớp, thuộc tính định tính | — |
| v1.0 | Hợp nhất 3 bản cá nhân. Thêm lớp `bus`, `tuk_tuk`, `bicycle`, `cyclo`; định nghĩa lớp có ví dụ / phản ví dụ; ngưỡng % cho `occluded_level`; định nghĩa đo được cho `truncated`; quy tắc R-01…R-12; giải quyết 12 bất đồng (mục 4.2) | `img_002`, `img_003`, `img_004`, `img_010`, `img_011`, `img_024`, `img_028`, `img_029`, `img_035`, `img_037`… |
