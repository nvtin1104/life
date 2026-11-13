---
name: thong-ke-chi-tieu
description: Thống kê và phân tích chi tiêu & bữa ăn từ Google Sheet (ThongKe, ChiTieu, BuaAn) — tổng quan, theo ngày/tuần/tháng, theo danh mục, so sánh tháng trước, chi phí trung bình mỗi bữa (ăn ngoài vs tự nấu), dinh dưỡng trung bình/ngày, tồn kho nguyên liệu và hao hụt. Dùng khi người dùng hỏi "tháng này tiêu bao nhiêu", "thống kê", "báo cáo tháng", "tổng kết tuần", "ăn uống hết bao nhiêu", "trung bình 1 bữa", "còn bao nhiêu ngân sách", "trong kho còn gì".
---

# Thống kê chi tiêu & bữa ăn

Đọc trước: `.claude/skills/_shared/conventions.md`, `.claude/skills/_shared/schema.md`.
Skill này **chỉ đọc**, không ghi dữ liệu (trừ khi người dùng yêu cầu cập nhật ngân sách → sửa `ThongKe!B4` và `config.monthly_budget`).

## 1. Lấy dữ liệu
- Câu hỏi nhanh (tháng này đã chi bao nhiêu, còn bao nhiêu ngân sách) → đọc `ThongKe!A3:B18` là đủ.
- Phân tích sâu → đọc `ChiTieu!A1:T` và `BuaAn!A1:R` rồi tự tính (lọc theo cột `Tháng` hoặc khoảng ngày).
- Khoảng thời gian mặc định: tháng hiện tại. "tuần này" = thứ 2 → hôm nay. "tháng trước", "quý", "năm 2026" theo nghĩa thông thường.

## 2. Các chỉ số
**Chi tiêu**
- Tổng chi, số giao dịch, TB/ngày (chia cho số ngày đã qua trong kỳ).
- Theo danh mục: số tiền, % tổng, chênh lệch so với kỳ trước (▲/▼ %).
- Top 5 khoản lớn nhất.
- Ngân sách: đã dùng %, dự báo cuối tháng = `TB/ngày × số ngày của tháng`; cảnh báo nếu dự báo vượt ngân sách.

**Ăn uống**
- Số bữa đã ghi, tổng chi phí bữa (BuaAn!M), TB/bữa; tách theo `Hình thức` (ăn ngoài / tự nấu / đặt giao).
- So sánh: TB/bữa ăn ngoài vs tự nấu → tiết kiệm được bao nhiêu nếu đổi N bữa.
- `Chi ăn uống thực (ChiTieu: Ăn uống + Đi chợ)` vs `Chi phí bữa đã phân bổ` → chênh lệch = tồn kho + hao hụt + bữa chưa ghi. Tách: tồn kho = Σ `ChiTieu!S` (giá trị còn lại); phần còn lại là hao hụt/chưa ghi.
- Dinh dưỡng: kcal TB/ngày (chỉ tính các ngày có ghi bữa), so với `config.daily_calorie_target`; tỷ lệ năng lượng từ P/C/F (`4P : 4C : 9F`); số ngày thiếu dữ liệu.

**Kho nguyên liệu**
- Danh sách dòng `Kho` còn lại > 0: tên, còn lại, giá trị, ngày mua; đánh dấu đồ tươi mua > 5 ngày.

## 3. Trình bày
Ngắn, bảng gọn, số định dạng `1.234.000đ`. Mẫu báo cáo tháng:
```
📊 Tháng 11/2026 (1–14/11)
Tổng chi: 4.120.000đ · 63 giao dịch · TB 294.000đ/ngày
Ngân sách 8.000.000đ → đã dùng 51% · dự báo cuối tháng 8.830.000đ ⚠ vượt 830.000đ

| Danh mục   | Số tiền    | %   | so T10 |
| Ăn uống    | 1.250.000  | 30% | ▲ 12%  |
| Đi chợ     |   890.000  | 22% | ▼ 5%   |
| Di chuyển  |   560.000  | 14% | =      |
...

🍚 Bữa ăn: 38 bữa · chi phí 1.640.000đ · TB 43.200đ/bữa
  Ăn ngoài 22 bữa · TB 56.000đ   |   Tự nấu 14 bữa · TB 29.000đ   |   Được mời 2
  Kho còn ~210.000đ nguyên liệu · hao hụt/chưa ghi ~290.000đ
  Dinh dưỡng TB: 1.870 kcal/ngày (mục tiêu 2.000) · P 18% · C 55% · F 27%

💡 Gợi ý: đổi 4 bữa trưa ăn ngoài/tuần sang mang cơm ≈ tiết kiệm 460.000đ/tháng.
```
Chỉ đưa 1–3 gợi ý cụ thể, dựa trên số liệu thật; không lên lớp.

## 4. Biểu đồ (khi người dùng muốn xem trực quan)
Dữ liệu đã tính sẵn ở sheet `ThongKe` (khối theo tháng / danh mục) — có thể gợi ý người dùng chèn chart trong Google Sheets từ các khối đó, hoặc dùng skill `dataviz` để vẽ biểu đồ trong chat từ số liệu vừa đọc.
