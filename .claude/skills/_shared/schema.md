# Schema Google Sheet — "Chi tiêu & Bữa ăn"

Một spreadsheet, 3 sheet chính. Tên sheet **không dấu** để tránh lỗi khi viết range A1.

| Sheet | Vai trò | Ai ghi |
|---|---|---|
| `ThongKe` | Tổng quan + thống kê theo tháng (toàn bộ là công thức) | Chỉ tạo 1 lần lúc setup |
| `ChiTieu` | Mỗi dòng = 1 khoản chi (hoặc 1 dòng hàng trong hóa đơn) | Skill `ghi-chi-tieu` |
| `BuaAn` | Mỗi dòng = 1 bữa ăn (món, dinh dưỡng, chi phí đã phân bổ) | Skill `ghi-bua-an` |

Cấu hình (spreadsheet_id, tên sheet, ngân sách...) nằm ở `.claude/skills/_shared/config.json`.

---

## 1. Sheet `ChiTieu`

Claude **chỉ ghi cột A → Q**. Cột R, S, T là ARRAYFORMULA ở hàng tiêu đề — **không bao giờ ghi đè**, nếu không sẽ lỗi `#REF!`.

| Cột | Tiêu đề | Kiểu | Ghi chú |
|---|---|---|---|
| A | ID | text | `CT-YYMMDD-NN` (vd `CT-261114-03`) |
| B | Ngày | date | `yyyy-mm-dd` |
| C | Giờ | text | `HH:mm`, để trống nếu không rõ |
| D | Danh mục | enum | xem danh sách bên dưới |
| E | Mô tả | text | tên món / tên hàng / nội dung chuyển khoản |
| F | Số lượng | number | mặc định 1 |
| G | Đơn vị | text | `kg`, `g`, `lít`, `hộp`, `bó`, `quả`, `phần`, `bữa`, `lần`... |
| H | Đơn giá | number | = I / F (làm tròn đồng) |
| I | Thành tiền | number | số tiền **thực trả** của dòng (đã trừ giảm giá) |
| J | Phương thức | enum | `Tiền mặt` / `Chuyển khoản` / `Thẻ` / `Ví điện tử` |
| K | Nơi mua | text | cửa hàng, quán, app |
| L | Mã hóa đơn | text | `HD-YYMMDD-NN` — gom các dòng cùng 1 hóa đơn; trống nếu chỉ 1 dòng |
| M | Loại phân bổ | enum | `Trực tiếp` / `Kho` / `Không` (xem mục 4) |
| N | Đã dùng | number | chỉ dùng cho `Kho`: tổng số lượng đã phân bổ vào bữa ăn (khởi tạo = 0) |
| O | Bữa ăn liên quan | text | các Meal ID, cách nhau bởi `, ` |
| P | Nguồn | text | `text` / `ảnh: <đường dẫn>` / `sao kê` |
| Q | Ghi chú | text | |
| R | Còn lại | **formula** | `={"Còn lại";ARRAYFORMULA(IF(M2:M="Kho",F2:F-N2:N,))}` |
| S | Giá trị còn lại | **formula** | `={"Giá trị còn lại";ARRAYFORMULA(IF(M2:M="Kho",ROUND((F2:F-N2:N)*H2:H),))}` |
| T | Tháng | **formula** | `={"Tháng";ARRAYFORMULA(IF(B2:B="",,TEXT(B2:B,"yyyy-mm")))}` |

### Danh mục (cột D)

| Danh mục | Dùng cho |
|---|---|
| `Ăn uống` | ăn ngoài, đồ uống, đặt đồ ăn, đồ ăn chín mua về |
| `Đi chợ` | thực phẩm/nguyên liệu nấu ăn, gia vị, đồ khô |
| `Di chuyển` | xăng, gửi xe, Grab/Be, vé xe, sửa xe |
| `Nhà ở` | tiền nhà, sửa chữa, đồ gia dụng lớn |
| `Hóa đơn` | điện, nước, internet, điện thoại, phí dịch vụ |
| `Mua sắm` | quần áo, đồ dùng cá nhân, đồ điện tử |
| `Sức khỏe` | thuốc, khám bệnh, gym, thực phẩm chức năng |
| `Giải trí` | phim, du lịch, game, subscription giải trí |
| `Học tập` | sách, khóa học, tài liệu |
| `Gia đình & Quà` | gửi gia đình, quà tặng, hiếu hỉ |
| `Khác` | không thuộc nhóm nào |

---

## 2. Sheet `BuaAn`

Claude **chỉ ghi cột A → Q**. Cột R là ARRAYFORMULA.

| Cột | Tiêu đề | Kiểu | Ghi chú |
|---|---|---|---|
| A | Meal ID | text | `BA-YYMMDD-NN` |
| B | Ngày | date | `yyyy-mm-dd` |
| C | Bữa | enum | `Sáng` / `Trưa` / `Tối` / `Phụ` |
| D | Món ăn | text | các món, cách nhau bởi `, ` |
| E | Hình thức | enum | `Ăn ngoài` / `Tự nấu` / `Đặt giao` / `Mang theo` / `Được mời` |
| F | Thành phần & khẩu phần | text | vd `cơm 200g; cá basa 150g; canh chua 1 chén` |
| G | Số người ăn chung | number | mặc định 1 |
| H | Calo (kcal) | number | phần của mình |
| I | Protein (g) | number | |
| J | Carb (g) | number | |
| K | Fat (g) | number | |
| L | Chất xơ (g) | number | để trống nếu không ước được |
| M | Chi phí bữa | number | **phần của mình**, sau khi phân bổ |
| N | Chi tiết phân bổ | text | vd `CT-261112-04: 0.15kg × 120.000 = 18.000; CT-261112-07: 1 bữa × 25.000 = 25.000` |
| O | Độ tin cậy | enum | `Ước lượng` / `Nhãn dinh dưỡng` / `Cân đo` |
| P | Ảnh | text | đường dẫn ảnh món ăn (nếu có) |
| Q | Ghi chú | text | |
| R | Tháng | **formula** | `={"Tháng";ARRAYFORMULA(IF(B2:B="",,TEXT(B2:B,"yyyy-mm")))}` |

---

## 3. Sheet `ThongKe`

Toàn bộ là công thức, tự cập nhật. Bố cục theo khối cột để các QUERY tự giãn không đè lên nhau.

### Khối A:B — Tổng quan (ô B4, B13 là ô nhập tay)

| Ô | Nhãn (cột A) | Công thức (cột B) |
|---|---|---|
| A1 | `THỐNG KÊ CHI TIÊU & BỮA ĂN` | |
| A3 | `TỔNG QUAN` | |
| 4 | Ngân sách tháng | *(số, lấy từ config `monthly_budget`)* |
| 5 | Tháng hiện tại | `=TEXT(TODAY(),"yyyy-mm")` |
| 6 | Đã chi tháng này | `=SUMIFS(ChiTieu!I:I,ChiTieu!T:T,B5)` |
| 7 | Còn lại ngân sách | `=IF(B4>0,B4-B6,"")` |
| 8 | Chi ăn uống + đi chợ tháng này | `=SUMIFS(ChiTieu!I:I,ChiTieu!T:T,B5,ChiTieu!D:D,"Ăn uống")+SUMIFS(ChiTieu!I:I,ChiTieu!T:T,B5,ChiTieu!D:D,"Đi chợ")` |
| 9 | Chi phí bữa ăn tháng này (đã phân bổ) | `=SUMIFS(BuaAn!M:M,BuaAn!R:R,B5)` |
| 10 | Số bữa tháng này | `=COUNTIFS(BuaAn!R:R,B5)` |
| 11 | Trung bình / bữa (tháng này) | `=IFERROR(B9/B10,0)` |
| 12 | Calo TB / ngày (tháng này) | `=IFERROR(SUMIFS(BuaAn!H:H,BuaAn!R:R,B5)/COUNTUNIQUE(FILTER(BuaAn!B2:B,BuaAn!R2:R=B5)),0)` |
| 13 | Mục tiêu calo / ngày | *(số, lấy từ config `daily_calorie_target`)* |
| 14 | Tổng chi tiêu (toàn bộ) | `=SUM(ChiTieu!I2:I)` |
| 15 | Tổng chi phí bữa ăn (toàn bộ) | `=SUM(BuaAn!M2:M)` |
| 16 | Tổng số bữa | `=COUNTA(BuaAn!A2:A)` |
| 17 | Trung bình / bữa (toàn bộ) | `=IFERROR(B15/B16,0)` |
| 18 | Giá trị nguyên liệu còn trong kho | `=SUM(ChiTieu!S2:S)` |

### Khối D:F — Chi tiêu theo tháng
- `D3`: `CHI TIÊU THEO THÁNG`
- `D4`:
  ```
  =QUERY(ChiTieu!A1:T,"select T, sum(I), count(A) where T is not null group by T order by T desc label T 'Tháng', sum(I) 'Tổng chi', count(A) 'Số dòng'",1)
  ```

### Khối H:M — Bữa ăn theo tháng
- `H3`: `BỮA ĂN THEO THÁNG`
- `H4`:
  ```
  =QUERY(BuaAn!A1:R,"select R, count(A), sum(M), avg(M), sum(H), avg(H) where R is not null group by R order by R desc label R 'Tháng', count(A) 'Số bữa', sum(M) 'Chi phí ăn', avg(M) 'TB/bữa', sum(H) 'Tổng kcal', avg(H) 'Kcal TB/bữa'",1)
  ```

### Khối O:Z — Danh mục × tháng (pivot)
- `O3`: `CHI TIÊU THEO DANH MỤC × THÁNG`
- `O4`:
  ```
  =QUERY(ChiTieu!A1:T,"select T, sum(I) where T is not null group by T pivot D order by T desc",1)
  ```

### Khối AB:AH — Chi phí bữa ăn theo hình thức × tháng
- `AB3`: `CHI PHÍ BỮA ĂN THEO HÌNH THỨC × THÁNG`
- `AB4`:
  ```
  =QUERY(BuaAn!A1:R,"select R, sum(M) where R is not null group by R pivot E order by R desc",1)
  ```

---

> **Bản thực tế đã ghi vào sheet:** cả 4 QUERY ở trên được bọc `=IFERROR(QUERY(...),"Chưa có dữ liệu")`, vì QUERY trên sheet trống trả `#VALUE!`. Khi sửa hoặc tạo lại sheet, giữ lớp bọc này.

---

## 4. Loại phân bổ (cột `ChiTieu!M`) — cốt lõi của tính chi phí mỗi bữa

| Loại | Khi nào | Cách tính chi phí bữa |
|---|---|---|
| `Trực tiếp` | Khoản chi = đúng 1 bữa (ăn phở 45k, đặt cơm 60k) | `chi phí bữa = Thành tiền / Số người ăn chung` (nếu mình trả cho cả nhóm) hoặc = Thành tiền (nếu chỉ trả phần mình). Ghi Meal ID vào cột O. |
| `Kho` | Nguyên liệu mua về dùng cho nhiều bữa (đi chợ, siêu thị) | Mỗi bữa lấy ra một lượng: `chi phí = lượng dùng × Đơn giá`. Cộng lượng dùng vào cột N (`Đã dùng`). Cột R tự tính phần còn lại. |
| `Không` | Không liên quan bữa ăn (xăng, điện...) | Không phân bổ |

**Mẹo "Kho" khi không có chi tiết hóa đơn:** "đi chợ 200k nấu cho 4 bữa" → ghi 1 dòng `Số lượng = 4`, `Đơn vị = bữa`, `Đơn giá = 50.000`. Mỗi bữa tự nấu lấy `1 bữa`. Cùng một cơ chế, không cần ngoại lệ.

**Gia vị / đồ dùng lâu (dầu ăn, nước mắm, gạo bao lớn):** ghi `Kho` với đơn vị thật (`lít`, `kg`) và trừ dần theo lượng ước tính, hoặc quy đổi sang `lần` (vd chai nước mắm 40k ≈ 40 lần → 1.000đ/lần).
