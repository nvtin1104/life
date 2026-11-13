---
name: ghi-bua-an
description: Ghi bữa ăn vào Google Sheet (sheet BuaAn) — tính chi phí thực của từng bữa bằng cách chia nhỏ các khoản chi (ăn ngoài chia theo số người, nguyên liệu đi chợ trong "Kho" trừ dần theo lượng dùng, mẻ nấu ăn nhiều bữa) và ước lượng dinh dưỡng (kcal, protein, carb, fat, chất xơ) từ tên món, nguyên liệu hoặc ảnh chụp món ăn. Dùng khi người dùng nói "ghi bữa ăn", "trưa nay ăn...", "tối nấu...", "bữa này hết bao nhiêu", "bao nhiêu calo", gửi ảnh món ăn, hoặc hỏi dinh dưỡng trong ngày.
---

# Ghi bữa ăn — chi phí & dinh dưỡng

Đọc trước: `.claude/skills/_shared/conventions.md`, `.claude/skills/_shared/schema.md` (mục 2 và 4).
Tham khảo: `references/phan-bo-chi-phi.md` (thuật toán chia chi phí), `references/dinh-duong-vn.md` (bảng dinh dưỡng).

## Quy trình

### 1. Xác định bữa
- Ngày, bữa (`Sáng/Trưa/Tối/Phụ`), món ăn, hình thức (`Ăn ngoài / Tự nấu / Đặt giao / Mang theo / Được mời`), số người ăn chung.
- **Ảnh món ăn**: nhận diện từng món, ước khẩu phần dựa vào kích thước tô/đĩa/chén/đũa trong ảnh, nêu rõ giả định ("tô phở cỡ vừa ~ 500g có nước"). Không chắc món gì → hỏi.

### 2. Tính chi phí bữa — theo `references/phan-bo-chi-phi.md`
| Hình thức | Nguồn chi phí |
|---|---|
| Ăn ngoài / Đặt giao | Khoản `Trực tiếp` trong `ChiTieu` (tìm khoản cùng ngày chưa gắn bữa; chưa có thì tạo qua skill `ghi-chi-tieu`) ÷ số người nếu mình trả cho cả nhóm |
| Tự nấu / Mang theo | Trừ nguyên liệu trong `Kho`: `lượng dùng × Đơn giá` cho từng nguyên liệu, chia theo số phần của mẻ nấu |
| Được mời | 0đ (ghi chú người mời) |

### 3. Ước lượng dinh dưỡng
- Món ăn ngoài phổ biến → bảng "Món ăn" trong `dinh-duong-vn.md`, điều chỉnh theo khẩu phần (tô lớn ×1.3, thêm trứng/thêm thịt cộng thêm).
- Tự nấu → cộng từ bảng "Nguyên liệu / 100g" theo lượng thực dùng (phần của mình), cộng dầu mỡ nấu ăn (chiên/xào: ~1 muỗng canh dầu/phần).
- Đồ đóng gói có nhãn → dùng số trên nhãn, `Độ tin cậy = Nhãn dinh dưỡng`.
- Làm tròn: kcal đến 10, macro đến 1g. Mặc định `Độ tin cậy = Ước lượng`.

### 4. Ghi vào sheet
1. Hiển thị xem trước (bắt buộc nếu có dùng ảnh, trừ kho, hoặc ước tính giá):
   ```
   BA-261114-02 · Tối 14/11 · Tự nấu · 2 người
   Món: Cơm, thịt kho trứng, canh cải
   Nguyên liệu cả mẻ (trừ kho):
     CT-261112-03 Thịt ba chỉ 0.3 kg  × 84.300 = 25.290
     CT-261112-05 Trứng gà    4 quả   × 3.260  = 13.040
     CT-261112-06 Cải ngọt    300 g   × 29,8   =  8.940
     CT-261110-01 Gạo         0.25 kg × 22.000 =  5.500
     CT-261101-02 Gia vị      4 lần   × 1.000  =  4.000
   Tổng mẻ 56.770đ ÷ 2 người → chi phí phần mình: 28.390đ
   Dinh dưỡng (ước lượng): 780 kcal · P 32g · C 85g · F 34g · xơ 3g
   ```
2. Ghi dòng `BuaAn!A{n}:Q{n}`.
3. Cập nhật `ChiTieu!N` (Đã dùng) của từng dòng Kho đã trừ = giá trị cũ + lượng dùng (của **cả mẻ**, không chỉ phần mình).
4. Thêm Meal ID vào `ChiTieu!O` của các dòng liên quan (nối thêm, không ghi đè ID cũ).

### 5. Báo kết quả
- Chi phí bữa + dinh dưỡng.
- Tổng trong ngày so với mục tiêu: `Hôm nay: 1.650 / 2.000 kcal · chi ăn 96.000đ (3 bữa)`.
- Nguyên liệu sắp hết hoặc đã mua > 5 ngày còn tồn (rau, thịt tươi) → nhắc ngắn.
- Phần còn lại của mẻ nấu (nếu có) → nhắc "còn 1 phần thịt kho, lần sau nói 'ăn lại thịt kho' để ghi".

## Câu hỏi thường gặp
- "Hôm nay ăn bao nhiêu calo / tốn bao nhiêu?" → đọc `BuaAn` theo ngày, cộng H:M, không ghi gì.
- "Bữa này hết bao nhiêu?" mà chưa ghi → làm bước 2 và hỏi có muốn ghi không.
- Không đủ thông tin nguyên liệu → đề xuất một ước tính hợp lý (dựa trên các bữa tự nấu gần đây hoặc dòng Kho đơn vị `bữa`) và hỏi người dùng đồng ý, ghi `Ghi chú = "chi phí ước tính"`.
