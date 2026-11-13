---
name: thiet-lap-sheet
description: Khởi tạo (hoặc kiểm tra/sửa) Google Sheet theo dõi chi tiêu & bữa ăn gồm 3 sheet ThongKe, ChiTieu, BuaAn qua Google Sheets MCP — tạo tiêu đề, công thức thống kê tổng và theo tháng, định dạng, rồi lưu spreadsheet_id vào config. Dùng khi người dùng nói "thiết lập sheet", "tạo file chi tiêu", "kết nối google sheet", "setup sheet", khi config chưa có spreadsheet_id, hoặc khi sheet bị lỗi công thức/thiếu cột.
---

# Thiết lập Google Sheet chi tiêu & bữa ăn

Đọc trước: `.claude/skills/_shared/schema.md` (cột, công thức) và `.claude/skills/_shared/conventions.md` (tool MCP, quy trình ghi).

## Bước 0 — Kiểm tra MCP
- Tìm các tool Google Sheets (vd `create_spreadsheet`, `update_cells`, `get_sheet_data`). Nếu tool bị ẩn, dùng `ToolSearch` với từ khóa `sheet`.
- Không có → dừng, hướng dẫn người dùng cấu hình theo `.claude/skills/README.md` (mục "Cài Google Sheets MCP").

## Bước 1 — Chọn spreadsheet
- Connector Google Sheets chính thức **không tạo được file mới**. Nếu không có Google Drive `create_file`: nhờ người dùng mở https://sheets.new và gửi link, hoặc mở sheets.new bằng Claude in Chrome rồi lấy ID từ URL.
- Người dùng đưa link/ID có sẵn → lấy ID (đoạn giữa `/d/` và `/edit`). Đọc `list_sheets` để xem đã có gì, **không xóa/ghi đè dữ liệu cũ** — chỉ bổ sung sheet/cột còn thiếu.
- Chưa có → `create_spreadsheet` với tiêu đề `config.spreadsheet_title`.
- Nếu tạo được bằng raw API: đặt `locale = en_US`, `timeZone = Asia/Ho_Chi_Minh` (xem "Lưu ý locale").

## Bước 2 — Tạo 3 sheet theo thứ tự
`ThongKe` (đổi tên từ "Sheet1" nếu có), `ChiTieu`, `BuaAn`.

## Bước 3 — Ghi tiêu đề + công thức

**ChiTieu!A1:Q1**
```
ID | Ngày | Giờ | Danh mục | Mô tả | Số lượng | Đơn vị | Đơn giá | Thành tiền | Phương thức | Nơi mua | Mã hóa đơn | Loại phân bổ | Đã dùng | Bữa ăn liên quan | Nguồn | Ghi chú
```
**ChiTieu!R1, S1, T1** — 3 công thức ARRAYFORMULA ở mục 1 của schema.md.

**BuaAn!A1:Q1**
```
Meal ID | Ngày | Bữa | Món ăn | Hình thức | Thành phần & khẩu phần | Số người ăn chung | Calo (kcal) | Protein (g) | Carb (g) | Fat (g) | Chất xơ (g) | Chi phí bữa | Chi tiết phân bổ | Độ tin cậy | Ảnh | Ghi chú
```
**BuaAn!R1** — công thức tháng ở mục 2 của schema.md.

**ThongKe** — ghi đúng các ô ở mục 3 của schema.md:
- `A1`, `A3`, nhãn `A4:A18`, công thức `B5:B12`, `B14:B18`.
- `B4` = `config.monthly_budget`, `B13` = `config.daily_calorie_target`.
- Tiêu đề khối + QUERY ở `D3/D4`, `H3/H4`, `O3/O4`, `AB3/AB4`.

Ghi bằng kiểu USER_ENTERED để công thức được tính.

## Bước 4 — Định dạng (tùy chọn, nếu MCP có `batch_update` raw request)
- Cố định hàng 1 ở `ChiTieu`, `BuaAn` (`updateSheetProperties` → `gridProperties.frozenRowCount = 1`); in đậm hàng tiêu đề.
- Định dạng số `#,##0` cho: `ChiTieu!H:I`, `ChiTieu!S:S`, `BuaAn!M:M`, `ThongKe!B4`, `ThongKe!B6:B11`, `ThongKe!B14:B18`.
- Định dạng ngày `yyyy-mm-dd` cho `ChiTieu!B:B`, `BuaAn!B:B`.
- Data validation dạng danh sách (`ONE_OF_LIST`, không strict):
  - `ChiTieu!D2:D` — 11 danh mục trong schema.md
  - `ChiTieu!J2:J` — Tiền mặt, Chuyển khoản, Thẻ, Ví điện tử
  - `ChiTieu!M2:M` — Trực tiếp, Kho, Không
  - `BuaAn!C2:C` — Sáng, Trưa, Tối, Phụ
  - `BuaAn!E2:E` — Ăn ngoài, Tự nấu, Đặt giao, Mang theo, Được mời
  - `BuaAn!O2:O` — Ước lượng, Nhãn dinh dưỡng, Cân đo

Không có tool định dạng → bỏ qua, báo người dùng có thể tự định dạng sau; dữ liệu và công thức vẫn chạy đúng.

## Bước 5 — Lưu config & kiểm tra
1. Ghi `spreadsheet_id`, `spreadsheet_url` vào `.claude/skills/_shared/config.json`.
2. Đọc lại `ThongKe!A1:F10` — công thức phải ra số (0) hoặc trống, **không** có `#ERROR!` / `#NAME?` / `#REF!`.
3. Báo người dùng: link sheet + nhắc chia sẻ quyền Editor cho email service account (nếu MCP dùng service account).

## Lưu ý locale
Spreadsheet locale Việt Nam (`vi_VN`) dùng dấu `;` ngăn cách tham số công thức. Nếu sau khi ghi thấy `#ERROR!` (lỗi phân tích công thức):
- Cách 1 (khuyên dùng): đổi locale sang `en_US` (File → Settings, hoặc `updateSpreadsheetProperties`) rồi ghi lại công thức.
- Cách 2: giữ locale, thay dấu `,` **ngăn cách tham số hàm** thành `;`. Không đổi dấu `,` nằm trong chuỗi (vd câu QUERY `"select T, sum(I) ..."`), và dấu `;` trong mảng `{"Tháng";...}` giữ nguyên (đó là ngăn cách hàng).

## Sửa sheet đã có
Khi người dùng báo lỗi thống kê: đọc hàng 1 của cả 3 sheet + các ô công thức, so với schema.md, chỉ ghi lại phần lệch. Không đụng vào dữ liệu từ hàng 2 trở đi.
