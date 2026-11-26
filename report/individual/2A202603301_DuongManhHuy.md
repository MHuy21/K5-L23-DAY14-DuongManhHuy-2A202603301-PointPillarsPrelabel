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
| Gốc | 42 | 3 | 2 | 16.6 |
| `best_config.json` | 17 | 6 | 2 | 12.1 |

> **Lịch sử sửa `ref_A.json`** (ghi lại cho minh bạch, xem lịch sử commit):
> - **Lần 1** (commit `ed459d1`): tôi **giữ** box Car nằm ngay tâm LiDAR, tức chính xe thu dữ liệu, ở cả 3 frame. Sweep lần 1 (commit `78ec4cf`) cũng chọn `score_thr = 0.4`: gốc 17.5 → 14.3 (−18%). Nhưng leave-one-out không ổn định (lượt bỏ s05f22 chọn 0.2/10), và s02f12 tệ hơn gốc (3.1 → 4.2).
> - Nguyên nhân: box xe thu dữ liệu có score 0.33–0.43, quanh ngưỡng lọc. Nó lúc bị lọc (bị tính "thêm" +1.0), lúc không, nên làm nhiễu sweep. Repo Day 14 không quy định về box này. Tôi quyết định **không gán nhãn xe thu dữ liệu**, vì guideline định nghĩa 5 lớp là vật thể xung quanh xe. Quy ước này áp dụng **cho cả tập A và tập B**.
> - **Lần 2** (commit này): xóa 3 box đó trên CVAT, xuất lại `ref_A.json` (18 / 13 / 15 box), chạy lại sweep và compare. `best_config.json` **không đổi** (`score_thr = 0.4`). **Mục 1 không sửa.** Các số trong mục 2 và 3 dưới đây là của lần 2.

- Tham số đã đổi trong `best_config.json`: chỉ đổi `score_thr` từ 0.1 lên **0.4**. `min_points` giữ 0, vì ở mức `score_thr` 0.4 cả ba mức 0, 5, 10 cho cùng chi phí 12.1. Sweep thử 15 cấu hình: `score_thr` ∈ {0.1, 0.2, 0.3, 0.4, 0.5} × `min_points` ∈ {0, 5, 10}.
- Chi phí từng frame của một số cấu hình (trích từ `sweep.csv`):

  | `score_thr` / `min_points` | s02f12 | s04f58 | s05f22 | Tổng |
  |---|---:|---:|---:|---:|
  | 0.1 / 0 (gốc) | 3.4 | 3.0 | 10.2 | 16.6 |
  | 0.2 / 10 | 3.4 | **1.8** | 9.9 | 15.1 |
  | 0.3 / 10 | **2.8** | 2.5 | 9.3 | 14.6 |
  | **0.4 / 0 (chọn)** | 3.2 | 1.9 | **7.0** | **12.1** |
  | 0.5 / 0 | 5.8 | 2.3 | 7.8 | 15.9 |

- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra):

  ```
  s02f12: cấu hình chọn {'score_thr': '0.4', 'min_points': '0'} → 3.2 (tốt nhất có thể 2.8)
  s04f58: cấu hình chọn {'score_thr': '0.4', 'min_points': '0'} → 1.9 (tốt nhất có thể 1.8)
  s05f22: cấu hình chọn {'score_thr': '0.4', 'min_points': '0'} → 7.0 (tốt nhất có thể 7.0)
  ```

- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó:
  - **Khá ổn định trong 3 frame.** Cả 3 lượt leave-one-out đều chọn lại `score_thr = 0.4`. Chi phí trên frame bỏ ra chỉ cao hơn mức tốt nhất có thể 0.4 (s02f12), 0.1 (s04f58) và 0 (s05f22). Tổng của 3 lượt là 3.2 + 1.9 + 7.0 = **12.1**, bằng đúng kết quả khi chọn trên cả 3 frame.
  - **Nhưng mỗi frame vẫn có ngưỡng tốt nhất riêng:** s02f12 tốt nhất ở 0.3, s04f58 ở 0.2, s05f22 ở 0.4. Ngưỡng 0.4 thắng trên tổng phần lớn nhờ **s05f22**, frame có nhiều box thừa nhất (24 box phải xóa ở cấu hình gốc, chiếm hơn nửa tổng số box phải xóa).
  - **Tôi chọn `score_thr = 0.4`** vì nó có chi phí tổng thấp nhất (12.1), và tổng phần chênh so với mức tốt nhất của từng frame nhỏ nhất: 0.4 + 0.1 + 0 = 0.5, so với 3.0 của mức 0.3 / 10 và 3.5 của mức 0.2 / 10. Tuy vậy, leave-one-out chỉ có 3 lượt trên 3 frame nên chưa đủ để khẳng định cấu hình sẽ tổng quát. Tập B (12 frame) là phép kiểm tra thật.

## 3. Trước/sau (bước 8)

![so sánh](../../d14_2A202603301/compare/s05f22.jpg)

*Frame s05f22: bên trái là cấu hình gốc (xóa 24, thêm 3, sửa 0 → 10.2), bên phải là `score_thr = 0.4` (xóa 10, thêm 4, sửa 0 → 7.0). Xanh lá là box đúng, vàng cần sửa, đỏ thừa (phải xóa), xanh dương bị sót (phải thêm).*

- `scripts/pp_compare.py` báo **giảm 27%** công sửa trên tập A (16.6 → 12.1). Chỉ đổi một tham số: `score_thr: 0.1 -> 0.4`.

  | Frame | Chi phí gốc | `score_thr` 0.4 |
  |---|---:|---:|
  | s02f12 | 3.4 | 3.2 |
  | s04f58 | 3.0 | 1.9 |
  | s05f22 | 10.2 | 7.0 |
  | **Tổng** | **16.6** | **12.1** |

- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa:
  - **Dự đoán đúng.** `score_thr` là tham số có tác dụng, và mức tốt nhất đúng là 0.4. Tăng lên 0.5 thì chi phí tăng lại lên 15.9 (phải thêm 13), đúng như lo ngại ở mục 1. `min_points` ít tác dụng: ở ngưỡng 0.1 nó chỉ giảm từ 16.6 xuống 16.0, ở ngưỡng 0.4 thì không đổi gì.
  - **Giải thích bằng số:** xóa giảm từ 42 xuống 17 (bớt 25 × 0.3 = −7.5), thêm tăng từ 3 lên 6 (mất 3 box thật × 1.0 = +3.0), sửa giữ nguyên 2. Tổng cộng −4.5. Kết quả **gần với ước lượng ở mục 1** (bớt khoảng 24 box xóa, mất khoảng 4 box thật, giảm khoảng 3 điểm). Mức giảm thật lớn hơn ước lượng vì ở lần sửa 2 tôi bỏ box xe thu dữ liệu khỏi `ref_A`: box này không còn bị tính "mất box thật" khi bị lọc.
  - **Ảnh s05f22** cho thấy rõ cơ chế này: phần lớn hộp đỏ (hộp trùm bụi cây và cọc rào hai bên đường) biến mất ở ảnh bên phải. Đổi lại, một xe thật ở xa phía trước (khoảng 38 m, score 0.35) chuyển thành xanh dương, tức bị sót. Box xe thu dữ liệu ở giữa có score 0.43 nên vẫn còn và vẫn phải xóa tay.
  - **Mức lợi không đều giữa các frame.** s02f12 chỉ giảm từ 3.4 xuống 3.2: frame này ít hộp thừa, ngưỡng 0.4 bớt được 4 hộp xóa (−1.2) nhưng mất một Cyclist thật phía sau (score 0.39, +1.0). Phần lớn mức giảm đến từ s05f22. Với chỉ 3 frame tune, con số 27% chưa đủ cơ sở để khẳng định cho dữ liệu khác.

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_2A202603301/frames_B.txt --config d14_2A202603301/best_config.json --gt d14_2A202603301/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- Nêu ít nhất 2 giới hạn (ví dụ: chỉ 3 frame tune; model KITTI không có Truck/Bus; chi phí là công thức, chưa phải thời gian sửa thật): ...
