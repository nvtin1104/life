# Skills: Chi tiêu & Bữa ăn (Google Sheets)

Bộ skill cho Claude Code để ghi chi tiêu hằng ngày, tính chi phí + dinh dưỡng từng bữa ăn, lưu vào Google Sheets qua MCP.

```
.claude/skills/
├── README.md                      ← file này
├── _shared/                       ← dùng chung (không phải skill)
│   ├── config.json                  spreadsheet_id, tên sheet, ngân sách, mục tiêu calo
│   ├── schema.md                    cột của 3 sheet + công thức thống kê + logic phân bổ
│   └── conventions.md               tool MCP, quy trình ghi, ID, đọc ngày/số tiền, ảnh
├── thiet-lap-sheet/               ← tạo spreadsheet 3 sheet + công thức (chạy 1 lần)
├── ghi-chi-tieu/                  ← ghi chi tiêu từ text / ảnh hóa đơn / chuyển khoản
│   └── references/doc-anh-chi-tieu.md
├── ghi-bua-an/                    ← ghi bữa ăn, chia chi phí, ước lượng dinh dưỡng
│   └── references/
│       ├── phan-bo-chi-phi.md
│       └── dinh-duong-vn.md
└── thong-ke-chi-tieu/             ← báo cáo tổng / theo tháng / ăn uống / kho
```

## 3 sheet

| Sheet | Nội dung |
|---|---|
| **ThongKe** | Tổng quan (tháng này, ngân sách, TB/bữa, kcal/ngày, tồn kho) + bảng theo tháng, theo danh mục × tháng, bữa ăn theo tháng — toàn công thức, tự cập nhật |
| **ChiTieu** | Mỗi khoản chi / mỗi dòng hàng trong hóa đơn. Cột `Loại phân bổ`: `Trực tiếp` (= 1 bữa), `Kho` (nguyên liệu dùng dần), `Không` |
| **BuaAn** | Mỗi bữa: món, hình thức, kcal/P/C/F, **chi phí phần mình** + cách tính |

Cách chia chi phí bữa: ăn ngoài → chia theo số người; tự nấu → trừ lượng nguyên liệu trong Kho × đơn giá, chia theo số phần của mẻ nấu. Chi tiết ở `_shared/schema.md` mục 4.

## Cài Google Sheets MCP

Skill không phụ thuộc server cụ thể — chỉ cần một MCP có thể tạo/đọc/ghi Google Sheets. Gợi ý dùng [`mcp-google-sheets`](https://github.com/xing5/mcp-google-sheets) (Python, chạy bằng `uvx`):

1. Google Cloud Console → tạo project → bật **Google Sheets API** và **Google Drive API**.
2. Tạo **Service Account** → tải key JSON (vd lưu ở `C:\Users\<you>\.secrets\gsheets-sa.json`, **không** để trong repo).
3. Tạo một thư mục trên Google Drive, chia sẻ quyền **Editor** cho email của service account, lấy ID thư mục (phần cuối URL).
4. Thêm server vào Claude Code (scope project tạo file `.mcp.json`, scope user dùng cho mọi project):

```bash
claude mcp add google-sheets --scope user -e SERVICE_ACCOUNT_PATH=C:/Users/<you>/.secrets/gsheets-sa.json -e DRIVE_FOLDER_ID=<folder_id> -- uvx mcp-google-sheets@latest
```

5. Khởi động lại phiên Claude Code, kiểm tra server đã kết nối.

> Dùng OAuth thay service account: xem README của server (biến `CREDENTIALS_PATH`, `TOKEN_PATH`). Server khác tên tool → các skill tự tìm tool tương đương (xem `_shared/conventions.md`).

## Bắt đầu
1. Nói: **"thiết lập sheet chi tiêu"** → Claude tạo spreadsheet, lưu ID vào `_shared/config.json`.
2. (Tùy chọn) sửa `monthly_budget`, `daily_calorie_target` trong `config.json`.
3. Dùng hằng ngày:
   - `sáng phở 45k, đổ xăng 60k`
   - *(gửi ảnh hóa đơn siêu thị)* `ghi giúp mình`
   - `tối nấu cơm thịt kho trứng cho 2 người, dùng 300g ba chỉ, 4 quả trứng`
   - *(gửi ảnh món ăn)* `bữa trưa nay, 55k`
   - `tháng này tiêu bao nhiêu rồi?` · `trung bình 1 bữa hết bao nhiêu?` · `trong kho còn gì?`

## Ghi chú
- Ảnh hóa đơn có thể lưu vào `data/receipts/YYYY-MM/` (cấu hình `receipts_dir`). Cân nhắc thêm `data/` vào `.gitignore` nếu không muốn commit ảnh.
- Spreadsheet nên để locale `en_US` để công thức dùng dấu `,` (xem `thiet-lap-sheet/SKILL.md` → "Lưu ý locale").
- Dinh dưỡng là **ước lượng** cho theo dõi cá nhân.
