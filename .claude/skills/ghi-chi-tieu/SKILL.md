---
name: ghi-chi-tieu
description: Ghi chi tiêu hằng ngày vào Google Sheet (sheet ChiTieu) từ tin nhắn text, ảnh hóa đơn/bill siêu thị, ảnh chụp màn hình chuyển khoản ngân hàng hoặc ví MoMo/ZaloPay. Tự phân danh mục, tách từng dòng hàng trong hóa đơn, đánh dấu nguyên liệu đi chợ vào "Kho" để phân bổ chi phí cho các bữa ăn sau. Dùng khi người dùng nói "ghi chi tiêu", "vừa mua", "hết 50k", "đi chợ", "đổ xăng", "trả tiền nhà", gửi ảnh hóa đơn/bill, hoặc liệt kê các khoản đã tiêu trong ngày; cũng dùng để sửa/xóa một khoản đã ghi.
---

# Ghi chi tiêu theo ngày

Đọc trước: `.claude/skills/_shared/conventions.md` và `.claude/skills/_shared/schema.md` (mục 1 và 4).

## Quy trình

### 1. Thu thập
Xác định nguồn:
- **Text** — có thể nhiều khoản trong 1 tin: "sáng phở 45k, đổ xăng 60k, tối đi chợ 230k". Tách từng khoản.
- **Ảnh** (hóa đơn, bill, chuyển khoản, sao kê) — làm theo `references/doc-anh-chi-tieu.md`.

Với mỗi khoản cần: ngày, số tiền, mô tả. Còn lại suy luận được thì tự điền (danh mục, phương thức mặc định `config.default_payment_method`, bữa ăn theo giờ).

### 2. Phân loại từng dòng
| Nội dung | Danh mục | Loại phân bổ |
|---|---|---|
| Ăn ở quán, mua đồ ăn chín, đặt GrabFood/ShopeeFood, cà phê, trà sữa | `Ăn uống` | `Trực tiếp` |
| Thực phẩm sống, rau củ, thịt cá, gạo, trứng, sữa, gia vị, mì gói mua trữ | `Đi chợ` | `Kho` |
| Mọi thứ khác | theo bảng danh mục trong schema.md | `Không` |

Mua đồ ăn cho cả nhà / nhóm: vẫn ghi đủ số tiền đã chi; phần chia đầu người xử lý ở bữa ăn (cột "Số người ăn chung").

### 3. Điền các cột theo loại
- **Kho**: tách theo từng mặt hàng nếu có thể. `Số lượng` + `Đơn vị` theo đơn vị đo thật (`0.5 kg`, `10 quả`, `1 bó`, `2 lít`), `Đơn giá = Thành tiền / Số lượng`, `Đã dùng = 0`. Không rõ số lượng mà người dùng nói "cho N bữa" → `Số lượng = N`, `Đơn vị = bữa`.
- **Trực tiếp**: `Số lượng = 1`, `Đơn vị = phần`, `Đơn giá = Thành tiền`. Ghi Meal ID tương ứng vào cột `Bữa ăn liên quan` (bước 5).
- **Không**: `Số lượng = 1`, `Đơn vị = lần`, `Đã dùng` để trống.
- Nhiều dòng từ cùng một hóa đơn → chung `Mã hóa đơn` `HD-YYMMDD-NN`.

### 4. Kiểm tra trùng & ghi
1. Đọc `ChiTieu!A:Q` của ngày đó → nếu có dòng trùng (cùng số tiền + nơi mua/mô tả) thì hỏi trước.
2. Xác nhận nếu dữ liệu đến từ ảnh hoặc chưa rõ (xem conventions mục 8) — hiển thị bảng xem trước:
   ```
   | ID | Ngày | Danh mục | Mô tả | SL | ĐV | Thành tiền | Phân bổ |
   ```
3. Ghi vào `A{n}:Q{n}` theo quy trình ở conventions mục 3.

### 5. Liên kết bữa ăn (khoản `Trực tiếp`)
Khoản ăn uống `Trực tiếp` chính là một bữa → tạo luôn dòng `BuaAn` theo skill `ghi-bua-an` (ước lượng dinh dưỡng từ tên món, chi phí = số tiền / số người ăn chung), rồi ghi Meal ID vào cột O của dòng chi tiêu. Đồ uống lẻ (cà phê, trà sữa) → bữa `Phụ`.
Người dùng nói rõ "không cần ghi bữa" → bỏ qua bước này.

### 6. Báo kết quả (ngắn gọn)
```
Đã ghi 3 khoản ngày 14/11 — tổng 335.000đ
• CT-261114-01 Ăn uống  Phở bò             45.000đ  → BA-261114-01 (Sáng, ~450 kcal)
• CT-261114-02 Di chuyển Đổ xăng           60.000đ
• HD-261114-01 Đi chợ   6 món vào Kho     230.000đ
Hôm nay đã chi: 335.000đ · Tháng 11: 4.120.000đ / ngân sách 8.000.000đ
```
Số "hôm nay" và "tháng này" lấy bằng cách đọc lại `ChiTieu` (hoặc `ThongKe!B6`).

## Sửa / xóa
- "sửa khoản phở thành 50k" → tìm theo ngày + mô tả (hoặc ID), cập nhật `Thành tiền` và `Đơn giá`; nếu khoản có bữa ăn liên quan thì cập nhật luôn `BuaAn!M` của bữa đó.
- Xóa: hỏi xác nhận, rồi xóa nội dung `A:Q` của dòng (hoặc xóa hẳn dòng nếu MCP hỗ trợ `deleteDimension`). Nếu là dòng `Kho` đã có `Đã dùng > 0` → cảnh báo các bữa đang tham chiếu nó.

## Ví dụ nhanh
- "trưa nay cơm tấm 40k với đồng nghiệp, mình trả luôn cho 2 người 80k" → 1 dòng `Ăn uống` 80.000, bữa Trưa với `Số người ăn chung = 2`, chi phí bữa 40.000.
- "đi chợ 150k: 300g thịt ba chỉ 45k, 1kg cá basa 70k, rau 20k, 10 quả trứng 35k" → tổng các món 170k ≠ 150k → hỏi lại số nào đúng trước khi ghi.
- "tiền điện tháng 10 hết 520k chuyển khoản" → `Hóa đơn`, `Chuyển khoản`, `Không`.
