# Thuật toán chia chi phí cho từng bữa

Nguyên tắc: **tiền đi ra (ChiTieu) ghi đúng số đã chi; chi phí bữa (BuaAn!M) là phần thực sự mình ăn.** Cột `Chi tiết phân bổ` (BuaAn!N) luôn ghi cách tính để kiểm tra lại được.

## A. Ăn ngoài / Đặt giao (khoản `Trực tiếp`)
1. Tìm trong `ChiTieu` khoản cùng ngày, `Danh mục = Ăn uống`, `Loại phân bổ = Trực tiếp`, cột O trống, khớp mô tả/nơi mua/số tiền. Không có → tạo khoản mới (theo skill `ghi-chi-tieu`).
2. Chi phí bữa:
   - Mình trả cho cả nhóm `k` người: `M = Thành tiền / k` → ghi chú "trả hộ k-1 người".
   - Chia tiền, mình chỉ trả phần mình: khoản chi đã là phần mình → `M = Thành tiền`.
   - Một khoản gồm nhiều bữa (đặt combo trưa + tối): chia theo giá từng món nếu biết, không thì chia đều.
3. `N` = `CT-261114-01: 80.000 ÷ 2 người = 40.000`.

## B. Tự nấu / Mang theo (trừ `Kho`)
1. Liệt kê nguyên liệu và lượng dùng **cho cả mẻ nấu**. Người dùng không nói lượng → ước theo khẩu phần thường (bảng cuối file) và cho người dùng xem.
2. Đọc `ChiTieu!A:T`, lọc `Loại phân bổ = Kho` và `Còn lại (R) > 0`. Với mỗi nguyên liệu:
   - Khớp theo tên (gần đúng: "thịt" ↔ "thịt ba chỉ"/"thịt nạc"; hỏi nếu nhiều lựa chọn khác nhau đáng kể về giá).
   - Nhiều dòng cùng loại → lấy **dòng mua cũ nhất trước** (FIFO); thiếu thì lấy tiếp dòng sau.
   - Đổi đơn vị về đơn vị của dòng kho: `1 kg = 1000 g`, `1 lít = 1000 ml`, `1 chén cơm chín ≈ 150 g cơm ≈ 55 g gạo`, `1 muỗng canh dầu ≈ 14 ml`, `1 quả trứng ≈ 50 g`.
   - `chi phí = lượng dùng × Đơn giá (H)`.
   - Lượng dùng > Còn lại → dùng hết phần còn lại, báo người dùng (có thể mua chưa ghi).
3. Nguyên liệu **không có trong Kho**: hỏi người dùng giá mua (và đề nghị ghi luôn khoản chi đó vào ChiTieu), hoặc dùng giá ước tính thị trường, ghi `(ước tính)` trong N.
4. `Tổng mẻ = Σ chi phí nguyên liệu`.
5. Chia theo phần:
   - `Số phần của mẻ = số người ăn chung × số bữa định ăn từ mẻ này` (hỏi nếu nấu nhiều: "nồi này ăn mấy bữa?").
   - `M = Tổng mẻ × (số phần mình ăn bữa này / Số phần của mẻ)`.
6. Cập nhật `Đã dùng (N)` cho từng dòng kho = cũ + lượng dùng **cả mẻ**.

### Mẻ nấu ăn nhiều bữa
Bữa đầu: trừ kho toàn bộ mẻ một lần, `N` ghi `MẺ BA-261114-02 (3 phần, 56.770đ) — phần 1/3: 18.920`. Ghi chú: `còn 2 phần`.
Bữa sau "ăn lại thịt kho": tìm dòng BuaAn có `MẺ` chưa hết phần → `M = Tổng mẻ / số phần`, `N` = `MẺ BA-261114-02 — phần 2/3: 18.920`, **không trừ kho lần nữa**. Cập nhật ghi chú của dòng mẻ gốc (`còn 1 phần`).

### Kho dạng "bữa" (đi chợ không có chi tiết)
Dòng kho `Số lượng = 4`, `Đơn vị = bữa`, `Đơn giá = 50.000` → mỗi bữa tự nấu lấy `1 bữa × 50.000`. Ăn chung 2 người cùng 1 "bữa" đã dự tính → vẫn trừ 1 bữa, chia 2.

## C. Được mời / Ăn miễn phí
`M = 0`, `N = "được mời"`, ghi người mời vào Ghi chú.

## D. Hao hụt & dọn kho
Khi người dùng nói "rau hỏng bỏ rồi", "hết trứng": cập nhật `Đã dùng = Số lượng` cho dòng đó, ghi chú `hao hụt <số lượng>`. Phần hao hụt **không** tính vào bữa nào (vẫn nằm trong tổng chi Đi chợ) — báo cáo tháng sẽ hiển thị chênh lệch "Chi đi chợ − Chi phí nguyên liệu đã ăn" như hao hụt / tồn kho.

## Khẩu phần ước lượng khi không có số liệu (1 người / 1 bữa)
| Nguyên liệu | Lượng |
|---|---|
| Gạo (sống) | 80–100 g (≈ 1.5–2 chén cơm) |
| Thịt / cá | 120–150 g |
| Trứng | 1–2 quả |
| Rau | 150–200 g |
| Đậu hũ | 1 bìa ~ 150 g |
| Dầu ăn (xào/chiên) | 10–15 ml |
| Gia vị | 1 "lần" |
