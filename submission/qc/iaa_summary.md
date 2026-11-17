# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 340 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 340 | 0.989 | 99.4% | 45.6% | 97.6% | 23/33 | 0 | ĐẠT |
| 101|102 | 13 | 125 | 0.992 | 100.0% | 48.0% | 96.0% | 10/5 | 0 | ĐẠT |
| 101|103 | 13 | 140 | 0.984 | 99.3% | 35.7% | 97.9% | 8/10 | 0 | ĐẠT |
| 102|103 | 14 | 75 | 0.993 | 98.7% | 60.0% | 100.0% | 5/18 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 483 ({'bo_sot_thua': 48, 'co_can_review': 218, 'sai_thuoc_tinh': 191, 'bat_dong_ignore': 13, 'iou_thap': 11, 'sai_lop': 2}), trên 36 ảnh cần calibration.
