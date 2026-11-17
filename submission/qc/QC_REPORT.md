# Báo cáo QC — Group79

Lô 40 ảnh đường phố, nhãn bounding box 2D, bàn giao theo vòng QC 3.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC nhãn 2D (phát hiện người và phương tiện thật) trong Day 07; tập 40 ảnh tuyển chọn để luyện ca biên, chưa đánh giá độ đại diện cho dữ liệu vận hành, không dùng để huấn luyện mô hình sản xuất.
- Nguồn và phép sử dụng dữ liệu: 40 ảnh Wikimedia Commons theo `data/ATTRIBUTION.csv` (CC BY, CC BY-SA, CC0, public domain); giữ ghi công và điều kiện chia sẻ tương tự của CC BY-SA khi phân phối lại.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `submission/qc/labels_calibrated.json` SHA-256 `1b267315aea11f23ebe0a220212065aa3adf8908ee5af034fc5dd335825f38ce` (40 ảnh, 390 annotation gồm 6 vùng `ignore_region`).
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0 (tag `v1.0-locked`); bàn giao theo `guideline_v1.1_calibrated.md` (thay đổi `editorial`, xem `guideline_change.json`), taxonomy giữ nguyên v1.0 (10 lớp, mục `rules` không đổi).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 (3 người 101, 102, 103; K = 2; 26 / 27 / 27 ảnh).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 340 cặp box ghép 1–1 (IoU ghép ≥ 0.5); IoU trung bình 0.989 và cùng lớp 99.4% trên 340 cặp → ĐẠT ngưỡng 0.85 / 95%; cùng `occluded_level` 45.6%, cùng `truncated` 97.6% trên 340 cặp. IAA bị đẩy cao do pre-label: 271/283 box của 101, 194/210 của 102, 204/248 của 103 giữ nguyên box của mô hình.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 23 + 33 box chưa ghép (48 ca `bo_sot_thua`); 0 box loại do ignore; 13 ca `bat_dong_ignore` (chỉ 103 dùng `ignore_region`, có vùng phủ box).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_020` — cả 101 và 103 vẽ `car` cho xe chỉ thấy trong gương chiếu hậu (R-02), IAA không phát hiện; đã xóa ở calibration. Vòng QC 1 tìm thêm các lỗi cùng giữ box của pre-label: `img_001` (truck lấn sang người), `img_002` (truck chưa bao xe con, R-04), `img_029` (chưa bao hàng, R-05), `img_015` (`occluded_level` do mép ảnh).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 494 dòng trong `calibration_decisions.csv` = 483 ca IAA (C001–C483) + 11 ca từ QC (Q01–Q11); mỗi dòng có mã quy tắc R-xx hoặc thuộc tính, người chốt, phiếu sửa (101 sửa) và kết quả kiểm lại (101, 103 cho ca IAA; 103 qua vòng QC 2–3 cho ca Q).
- Tác động v1.1 lên nhãn và ảnh: `editorial`, không đổi cách chọn vật, lớp, box hay thuộc tính; phạm vi sửa 36 ảnh bị cờ (4 ảnh còn lại không có box); không cần mapping vì taxonomy giữ v1.0.
- Tham chiếu bằng chứng khi Rework: Vòng 1 (`reviews/round-01`, REWORK 9/40) → Q01–Q10 → `labels_calibrated_4a166e524360c217.json` → vòng 2 (`reviews/round-02`, REWORK 1/40) → Q11 → `labels_calibrated_811a13f287fbd8cd.json` → vòng 3 (`reviews/round-03`, RELEASE); giữ nguyên thư mục cả 3 vòng.

## 4. Kết quả duyệt

- Vòng QC: 3
- Người review và ngày: 103, 2026-11-17
- Đã duyệt / cả lô: 40/40
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0% (vòng 1: 9/40 = 22.5%; vòng 2: 1/40 = 2.5%)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/384 = 0.0% (vòng 1: 6/382 = 1.6%; vòng 2: 1/384 = 0.3%)
- Vật bỏ sót, ảnh và vùng: Vòng 3: 0. Vòng 1 có 4 ca bỏ sót đã sửa: `img_011` cụm khách ở xa (Q04, ignore_region), `img_021` dòng xe ở xa (Q07, ignore_region), `img_030` xe máy mép trái (Q09), `img_040` xe máy của người đội mũ (Q10).
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Lỗi lặp ở vòng 1 là box giữ nguyên từ pre-label chưa áp quy tắc nhóm (R-04 `img_002`, R-05 `img_029`, ranh giới `img_001`); `occluded_level` vẫn là thuộc tính khó ước lượng nhất (45.6% đồng thuận trước calibration, lỗi duy nhất của vòng 2 là `img_005`).
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy: `qc_report.py` tự xếp REWORK khi 1/40 ảnh lỗi (2.5% > 2%), và `build_final.py` từ chối COCO sau sửa khi tên phiên bản (hash) không khớp bảng quyết định mới. Người: xe trong gương ở `img_020` — hai người vẽ giống nhau nên IAA không bắt được, phải đọc ảnh mới thấy là phản chiếu.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Vòng 3 duyệt toàn lô 40/40, 0 ảnh lỗi, 0 annotation lỗi, không lỗi lặp, đủ bằng chứng 3 vòng; người nhận là coach lớp K5-L23; chỉ dùng cho mục đích thực hành Day 07.
- Nếu còn việc: Không còn việc bắt buộc; nếu dùng tiếp, nên rà lại `occluded_level` toàn lô vì đây là thuộc tính có đồng thuận thấp nhất.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca Q08 (`calibration_decisions.csv`) → `img_029.jpg` → R-05 (bao cả hàng đang chở) → box `motorcycle` [60, 118, 845, 256] trong `labels_calibrated.json` bao xe máy và ống kim loại; lỗi gốc ở `reviews/round-01/qc_review.csv` (#259); kết quả: khớp.
