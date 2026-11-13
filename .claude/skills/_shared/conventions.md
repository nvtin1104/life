# Quy ước chung cho các skill chi tiêu / bữa ăn

## 1. Đọc config trước tiên
Luôn đọc `.claude/skills/_shared/config.json`.
- `spreadsheet_id` trống → dừng lại, chạy skill `thiet-lap-sheet` trước.
- Tên sheet lấy từ `sheets.*` (mặc định `ThongKe`, `ChiTieu`, `BuaAn`).

## 2. Google Sheets MCP — thao tác cần dùng
Các skill viết theo **thao tác**, không phụ thuộc một server cụ thể. Bảng ánh xạ với server phổ biến `mcp-google-sheets` (xem README):

| Thao tác | Tool thường gặp | Ghi chú |
|---|---|---|
| Tạo spreadsheet | `create_spreadsheet` | |
| Liệt kê / tạo sheet | `list_sheets`, `create_sheet`, `rename_sheet` | Sheet mặc định "Sheet1" → đổi tên thành `ThongKe` |
| Đọc dữ liệu | `get_sheet_data` (range), `get_multiple_sheet_data` | |
| Ghi ô / dòng | `update_cells` (range + 2D array), `batch_update_cells` | Ghi theo kiểu USER_ENTERED để ngày/công thức được hiểu |
| Thêm dòng | `add_rows` hoặc `update_cells` vào dòng trống kế tiếp | |
| Định dạng / validation | `batch_update` (raw Sheets API request) | Tùy chọn, nếu server hỗ trợ |

**Connector chính thức Google Sheets** (`sheetsmcp.googleapis.com`, kết nối qua claude.ai) — ưu tiên dùng nếu có:

| Thao tác | Tool |
|---|---|
| Tạo spreadsheet | Không có trong Sheets connector → dùng Google Drive connector `create_file` (mimeType `application/vnd.google-apps.spreadsheet`), hoặc nhờ người dùng tạo file trống ở sheets.new rồi gửi link |
| Xem cấu trúc (danh sách sheet, locale) | `get_spreadsheet` |
| Tạo / đổi tên sheet, định dạng, validation, locale | `update_spreadsheet` (batchUpdate: `addSheet`, `updateSheetProperties`, `repeatCell`, `setDataValidation`, `updateSpreadsheetProperties`) |
| Đọc dữ liệu | `get_values` |
| Ghi giá trị | `update_values` |
| Ghi công thức | `update_formulas` |
| Chèn dòng/cột | `insert_dimension` |

Nếu tên tool khác, tìm tool có chức năng tương đương (dùng `ToolSearch` với từ khóa `sheet`). Nếu không có tool Google Sheets nào → báo người dùng cấu hình MCP theo README, không giả vờ đã ghi.

## 3. Quy trình ghi dòng an toàn
1. Đọc cột A của sheet đích (`ChiTieu!A:A` hoặc `BuaAn!A:A`) → tìm dòng trống kế tiếp `n` và ID lớn nhất trong ngày.
2. Ghi đúng range `A{n}:Q{n}` (nhiều dòng: `A{n}:Q{n+k-1}`). **Không ghi vào cột formula** (ChiTieu R:T, BuaAn R).
3. Ô không có dữ liệu → chuỗi rỗng `""` (chỉ trong vùng A:Q).
4. Sau khi ghi, đọc lại range vừa ghi để xác nhận, rồi báo kết quả ngắn gọn cho người dùng (ID + số tiền + danh mục).
5. Khi sửa một dòng cũ: tìm theo ID ở cột A, chỉ cập nhật đúng ô cần sửa.

## 4. ID
- Chi tiêu: `CT-YYMMDD-NN`, bữa ăn: `BA-YYMMDD-NN`, hóa đơn: `HD-YYMMDD-NN`.
- `NN` = số thứ tự trong ngày, 2 chữ số, tăng dần từ ID lớn nhất đã có của ngày đó.

## 5. Ngày giờ (múi giờ `Asia/Ho_Chi_Minh`)
- Không nói ngày → hôm nay. "hôm qua", "tối qua", "sáng nay", "thứ 2 tuần này", "15/11" (dd/mm, năm hiện tại) → quy về `yyyy-mm-dd`.
- Bữa theo giờ nếu không nói rõ: trước 10:00 `Sáng`; 10:00–14:00 `Trưa`; 17:00–21:30 `Tối`; còn lại `Phụ`.

## 6. Đọc số tiền kiểu Việt Nam
| Người dùng viết | Giá trị |
|---|---|
| `45k`, `45 nghìn`, `45 ngàn`, `45 cành` | 45.000 |
| `1tr2`, `1.2tr`, `1,2 triệu`, `1 củ 2` | 1.200.000 |
| `2tr`, `2 củ`, `2 triệu` | 2.000.000 |
| `150.000`, `150,000`, `150000đ`, `150K VND` | 150.000 |
| `3 xị`, `1 lít` (tiếng lóng = 100k) | 300.000 / 100.000 — **hỏi lại** nếu ngữ cảnh có thể là thể tích |

Lưu số nguyên (không dấu phân cách) khi ghi vào sheet. Khi hiển thị cho người dùng: `45.000đ`.

## 7. Ảnh
- Ảnh người dùng đính kèm hoặc đường dẫn file → xem trực tiếp bằng khả năng đọc ảnh (Read tool với file ảnh). Ảnh mờ / nhiều chữ nhỏ → có thể dùng skill `ai-multimodal` để OCR.
- Nếu ảnh là file trên máy, có thể copy vào `data/receipts/YYYY-MM/` (theo `receipts_dir` trong config) và ghi đường dẫn vào cột Nguồn / Ảnh.
- Thông tin đọc từ ảnh **luôn hiển thị bảng xem trước** để người dùng xác nhận trước khi ghi (OCR có thể sai số).

## 8. Khi nào hỏi lại, khi nào ghi luôn
- Ghi luôn: thông tin dạng text rõ ràng (có số tiền + biết là gì).
- Hỏi lại (một câu gọn, gộp mọi điểm chưa rõ): thiếu số tiền, không rõ danh mục giữa 2 lựa chọn, dữ liệu từ ảnh, khoản lớn bất thường (> 5 triệu), có khả năng trùng (cùng ngày + cùng số tiền + cùng nơi đã có trong sheet).
