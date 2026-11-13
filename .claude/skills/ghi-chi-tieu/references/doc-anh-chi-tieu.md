# Đọc ảnh để ghi chi tiêu

Xem ảnh trực tiếp (Read tool với file ảnh, hoặc ảnh đính kèm trong chat). Ảnh khó đọc → thử skill `ai-multimodal` để OCR; vẫn không đọc được → hỏi người dùng con số, đừng đoán.

## 1. Nhận dạng loại ảnh
| Loại | Dấu hiệu | Trích xuất |
|---|---|---|
| Hóa đơn siêu thị / cửa hàng | bảng tên hàng – SL – đơn giá – thành tiền, có "Tổng cộng", "Tiền khách đưa" | từng dòng hàng + giảm giá + tổng |
| Bill quán ăn / nhà hàng | tên món, số bàn, phí phục vụ, VAT | tổng tiền, các món (để ước dinh dưỡng) |
| Đơn app giao đồ ăn (GrabFood, ShopeeFood, Be) | tên quán, món, phí giao, khuyến mãi | tổng thực trả, món |
| Chuyển khoản ngân hàng / ví | "Chuyển tiền thành công", số tiền, người nhận, nội dung, thời gian | 1 khoản |
| Sao kê / lịch sử giao dịch | danh sách nhiều giao dịch | chỉ các giao dịch **chi** (dấu `-`), bỏ tiền vào |

## 2. Hóa đơn nhiều dòng
1. Ghi lại: nơi mua, ngày giờ trên hóa đơn (không có → hỏi hoặc dùng hôm nay), phương thức thanh toán nếu in trên bill.
2. Mỗi dòng hàng: tên (viết lại dễ hiểu, vd `CẢI NGỌT VS 500G` → `Cải ngọt 500g`), số lượng, đơn vị, thành tiền.
   - Hàng cân ký: `0.452 kg × 89.000 = 40.228` → `Số lượng 0.452`, `Đơn vị kg`.
   - Hàng đóng gói có trọng lượng trong tên (`Thịt xay 300g`) → ưu tiên đơn vị gram/kg để phân bổ chính xác: `Số lượng 300`, `Đơn vị g`.
3. **Giảm giá / voucher tổng bill**: chia theo tỷ lệ thành tiền của từng dòng, sao cho tổng các dòng = số tiền thực trả. Làm tròn đến đồng, dồn phần lệch làm tròn vào dòng lớn nhất.
4. **Phí giao hàng, phí phục vụ, VAT** (nếu tách riêng): cũng chia theo tỷ lệ vào các dòng (với đồ ăn) — hoặc ghi 1 dòng riêng nếu hóa đơn không phải đồ ăn.
5. **Đối chiếu**: tổng các dòng phải bằng "Tổng thanh toán" trên bill (lệch ≤ 1.000đ do làm tròn là chấp nhận). Lệch lớn hơn → báo người dùng dòng nào có thể đọc sai.
6. Phân loại từng dòng theo bảng ở SKILL.md (một hóa đơn siêu thị có thể vừa `Đi chợ/Kho` vừa `Mua sắm/Không`, vd nước giặt).

## 3. Chuyển khoản / ví điện tử
- Số tiền, thời gian, người nhận, nội dung chuyển khoản → `Mô tả` = nội dung (rút gọn), `Nơi mua` = người nhận, `Phương thức` = `Chuyển khoản` hoặc `Ví điện tử`.
- Đoán danh mục từ nội dung/người nhận (vd "tien nha T11" → `Nhà ở`, "EVN" → `Hóa đơn`, tên quán → `Ăn uống`). Không đoán được → hỏi.
- Không ghi số tài khoản, số dư, mã giao dịch vào sheet.

## 4. Xem trước trước khi ghi
Luôn trình bày bảng xem trước + tổng, và chỉ rõ những chỗ không chắc (đánh dấu `?`):
```
Hóa đơn Bách Hóa Xanh — 14/11/2026 18:42 — tổng thực trả 187.500đ (đã trừ KM 12.500đ)
| # | Mô tả            | SL    | ĐV | Thành tiền | Danh mục | Phân bổ |
| 1 | Thịt ba chỉ      | 0.452 | kg | 38.100     | Đi chợ   | Kho     |
| 2 | Cải ngọt         | 500   | g  | 14.900     | Đi chợ   | Kho     |
| 3 | Trứng gà (10)    | 10    | quả| 32.600     | Đi chợ   | Kho     |
| 4 | Nước giặt 2L ?   | 1     | chai| 101.900   | Mua sắm  | Không   |
Ghi vào sheet? (có thể sửa dòng bất kỳ)
```

## 5. Lưu ảnh (tùy chọn)
Ảnh là file trên máy và người dùng muốn lưu → copy vào `data/receipts/YYYY-MM/<Mã hóa đơn hoặc ID>.<ext>`, ghi `ảnh: data/receipts/...` vào cột `Nguồn`.
