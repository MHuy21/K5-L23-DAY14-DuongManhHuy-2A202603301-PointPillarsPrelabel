# Báo cáo Day 14 — Dương Mạnh Huy — 2A202603301

- Mã học viên dùng với `--student`: `2A202603301`
- `seed_tag` trong `frames_A.txt`: `82ac3100fd`

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

- Tham số tôi nghĩ giảm công sửa nhiều nhất: **tăng `score_thr`** từ 0.1 lên khoảng **0.4**. `min_points` (5 hoặc 10) chỉ giảm được rất ít.
- Vì sao (nhìn ảnh BEV tập A: box thừa nhiều hay box sót nhiều?):
  - **Box thừa nhiều hơn hẳn box sót.** Cấu hình gốc trên tập A: xóa 40, thêm 4, sửa 3 → chi phí 17.5. Phần xóa chiếm 40 × 0.3 = 12.0 điểm, tức khoảng 70% chi phí. Trên ảnh `png_A/`, box thừa chủ yếu là hộp trùm lên bụi cây và cọc rào ven đường (Car và Cyclist), hộp trùng nhau, và hộp lơ lửng.
  - **Box thừa có score thấp.** Khi đối chiếu pre-label với nhãn tôi đã sửa, nhóm box bị xóa có score trung vị khoảng 0.38, còn nhóm box giữ lại là 0.78. Khoảng 24 box bị xóa có score < 0.4, trong khi chỉ khoảng 4 box giữ lại có score < 0.4. Tăng `score_thr` lên 0.4 có thể bớt khoảng 24 × 0.3 ≈ 7 điểm xóa, nhưng làm mất khoảng 4 box thật (mỗi box +1.0). Ước lượng chi phí giảm khoảng 3 điểm.
  - **Không nên tăng quá cao.** Ở mức 0.5, khoảng 13 box thật bị lọc mất (+13), lớn hơn phần xóa được (khoảng 31 × 0.3 ≈ 9.3). Vì thêm một box tốn 1.0, gấp hơn 3 lần xóa một box (0.3), ngưỡng quá cao sẽ làm chi phí tăng lại. Cyclist dễ bị mất nhất vì model KITTI cho Cyclist score thấp.
  - **`min_points` ít tác dụng.** Chỉ khoảng 2 box bị xóa có dưới 10 điểm LiDAR. Phần lớn box thừa nằm trên bụi cây nên vẫn có vài chục điểm. Ngược lại, một số xe thật ở xa (khoảng 38–45 m) cũng chỉ có 14–60 điểm.
  - Đây là ước lượng: script ghép box theo BEV IoU ≥ 0.3 và có ngưỡng riêng cho "sửa", nên số thật sau sweep có thể khác.

## 2. Tập A: sửa tay và tune

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | | | | |
| `best_config.json` | | | | |

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_2A202603301/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_2A202603301/frames_B.txt --config d14_2A202603301/best_config.json --gt d14_2A202603301/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- Nêu ít nhất 2 giới hạn (ví dụ: chỉ 3 frame tune; model KITTI không có Truck/Bus; chi phí là công thức, chưa phải thời gian sửa thật): ...
