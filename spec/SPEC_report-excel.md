# [SPEC_report-excel] — Mẫu in Excel cho Báo cáo (Report Excel Template)

> Dùng khi: tạo file Excel template (`.xlsx`) cho mẫu in báo cáo.
> Thư mục: `Web/App_Data/Controllers/Templates/Excel/{templateFile}.xlsx`
> Đọc `SPEC_report-print.md` trước để hiểu cách khai báo form Excel trong Report XML.

---

## Tổng quan

Mỗi mẫu in Excel là file `.xlsx` chứa layout cố định với các **biến placeholder**. Khi user chọn in Excel, framework sẽ:

1. Đọc template `.xlsx`
2. Thay thế các biến bằng giá trị thực
3. Mở rộng vùng dữ liệu (insert rows) theo số dòng kết quả
4. Tính lại công thức
5. Trả file Excel cho user download

---

## Quy ước đặt tên Sheet

| Tên sheet | Xử lý |
|---|---|
| `Main`, `Main_01`, `Main_abc`, `MainA` | ✅ Framework xử lý đổ dữ liệu |
| `ABC`, `Note`, `Sheet1` | ❌ Bỏ qua — không đổ dữ liệu |

> **Bắt buộc**: Sheet chứa dữ liệu phải bắt đầu bằng `Main`. Nếu đặt tên khác thì framework không nhận, không đổ data.

---

## Các loại biến (Placeholder Variables)

### 1. Biến Entity — Thông tin công ty (`?Entity_...`)

| Biến | Mô tả | Nguồn |
|---|---|---|
| `?Entity_Line1` | Tên tập đoàn / công ty (dòng 1) | Entity.lic |
| `?Entity_Line2` | Tên công ty (dòng 2) | Entity.lic |
| `?Entity_Line3` | Địa chỉ (dòng 3) | Entity.lic |
| `?Entity_Line4` | Thông tin bổ sung (dòng 4) | Entity.lic |
| `?Entity_Line5` | Thông tin bổ sung (dòng 5) | Entity.lic |

> Mặc định đặt ở **cột A, dòng 1–4**, font **Times New Roman 10pt**, dòng 1 bold.

### 2. Biến Parameter — Header / Label (`?...`)

| Cú pháp | Mô tả | Ví dụ |
|---|---|---|
| `?field_name` | Lấy giá trị từ `<fields>` trong Report XML | `?title` → "BÁO CÁO NHẬP XUẤT TỒN" |
| `?h_xxx` | Header cột, khai báo trong Report XML | `?h_stt` → "Stt", `?h_ma_vt` → "Mã vật tư" |

**Ưu tiên lấy giá trị:**
1. `<fields>` trong `<form>` (form-level — ghi đè cho mẫu cụ thể)
2. `<fields>` trong `<report>` (report-level — chung cho tất cả mẫu)
3. File `Report.xml` trong thư mục Options (system-level)

### 3. Biến dữ liệu bảng — Từ kết quả Store (`!N.column`)

| Cú pháp | Mô tả | Ví dụ |
|---|---|---|
| `!1.column` | Lấy cột từ bảng 1 (table 1 — thường là parameter) | `!1.tu_ngay` → "01/01/2026" |
| `!2.column` | Lấy cột từ bảng 2 (table 2 — thường là data) | `!2.ma_vt` → "VT0001" |
| `!N.column%l` | Tự chọn ngôn ngữ hiện hành | `!2.ten_vt%l` → tên VN hoặc EN |
| `!N.column%c` | Tự format theo loại tiền (VNĐ/ngoại tệ) | `!2.du_dau%c` |

> **Đánh số bảng**: tính từ 1, theo thứ tự resultset mà Store procedure trả về. Thông thường: Table 1 = parameter (date_from, date_to, ...), Table 2 = data chính.

### 4. Biến nối chuỗi (`#...`)

| Cú pháp | Ví dụ | Kết quả |
|---|---|---|
| `#?var1 + + !1.col + + ?var2 + + !1.col2` | `#?h_tu_ngay + + !1.tu_ngay + + ?h_den_ngay + + !1.den_ngay` | "Từ ngày 01/01/2026 đến ngày 31/12/2026" |

> Dùng ` + + ` (có khoảng trắng) làm separator giữa các phần. Kết quả nối sẽ tự chèn khoảng trắng giữa các phần.

### 5. Biến chữ ký — Hệ thống (`?signature...`)

| Biến | Mô tả | Vị trí điển hình |
|---|---|---|
| `?reportDate` | Dòng "Ngày ... tháng ... năm ..." | Giữa phải, trên chữ ký |
| `?chiefAccountant` | Chức danh "Kế toán trưởng" | Cột trái |
| `?director` | Chức danh "Giám đốc" | Cột phải |
| `?signatureFullname` | Dòng "(Ký, họ tên)" | Dưới chức danh |
| `?chiefAccountantName` | Tên kế toán trưởng | Cuối cùng bên trái |
| `?directorName` | Tên giám đốc | Cuối cùng bên phải |

> Các biến chữ ký lấy từ file Options hệ thống, không cần khai báo trong Report XML.

### 6. Conditional binding — Ẩn/hiện theo điều kiện (`{b:condition}`)

| Cú pháp | Mô tả | Ví dụ |
|---|---|---|
| `!2.column{b:field=value}` | Chỉ hiện giá trị khi điều kiện đúng | `!2.ma_vt{b:systotal=0}` → chỉ hiện mã VT khi systotal=0 (dòng chi tiết) |

> Dòng tổng nhóm (systotal=0) thường không hiện mã, tên, đvt → dùng `{b:systotal=0}` để ẩn.

---

## Layout chuẩn — Cấu trúc dòng

Thứ tự dòng trong template (áp dụng cho hầu hết báo cáo):

| Dòng | Nội dung | Ghi chú |
|---|---|---|
| 1–4 | Entity (tên công ty, địa chỉ) | Cột A, left-align |
| 1–4 (cột phải) | Logo hoặc trống | Merge cuối bảng, right-align |
| 5 | Trống | |
| 6 | **Tiêu đề báo cáo** (`?title`) | Merge toàn bộ, center, **bold 16pt** |
| 7 | Dòng ngày từ/đến | Merge toàn bộ, center, 10pt |
| 8 | Trống | |
| 9(–10) | **Header cột** | Fill xanh nhạt, bold, border, center |
| N | **Vùng dữ liệu** (1 dòng mẫu) | Framework insert thêm dòng |
| N+2 | Trống (spacer) | |
| N+3 | **Dòng tổng** | Bold, border-top thin, công thức SUMIF |
| +2 | Trống | |
| +3 | `?reportDate` | Center, cột phải |
| +4 | Chức danh chữ ký | Bold 9pt, center |
| +5 | "(Ký, họ tên)" | Normal 9pt, center |
| +6 | Trống (chừa chỗ ký) | Row height ~57pt |
| +7 | Tên người ký | Bold 9pt, center |

### Header 1 dòng vs 2 dòng

- **1 dòng header** (VD: rptStockSummary_01): Row 9 chứa tất cả header → Row 10 là data
- **2 dòng header** (VD: rptStockSummary_02): Row 9 = header nhóm (Tồn đầu, Nhập, Xuất, Tồn cuối), Row 10 = header con (Số lượng, Giá trị) → Row 11 là data

---

## Định dạng (Formatting)

### Màu nền header

**RGB: 237, 245, 255** (hex `#EDF5FF`)

Trong Excel lưu dạng GradientFill hoặc PatternFill:
- GradientFill stops: `FFEDF5FF` → `FFEDF5FF` (đồng nhất)
- Hoặc PatternFill solid: `FFEDF5FF`

### Font chuẩn

| Vùng | Font | Size | Style |
|---|---|---|---|
| Entity dòng 1 | Times New Roman | 10pt | **Bold** |
| Entity dòng 2–4 | Times New Roman | 10pt | Normal |
| Tiêu đề (`?title`) | Times New Roman | **16pt** | **Bold** |
| Dòng ngày | Times New Roman | 10pt | Normal |
| Header cột | Times New Roman | 10pt | **Bold** |
| Dữ liệu | Times New Roman | 10pt | Normal |
| Dòng tổng | Times New Roman | 10pt | **Bold** |
| Chức danh chữ ký | Times New Roman | 9pt | **Bold** |
| "(Ký, họ tên)" | Times New Roman | 9pt | Normal |
| Tên người ký | Times New Roman | 9pt | **Bold** |

### Border

| Vùng | Left/Right | Top | Bottom |
|---|---|---|---|
| Header cột | thin | thin | thin |
| Dữ liệu | thin | — | hair (nét mảnh) |
| Dòng tổng | — | thin | — |

### Canh lề (Alignment)

| Loại cột | Horizontal | Ví dụ |
|---|---|---|
| STT, mã số | center | `!2.stt` |
| Tên, diễn giải | left | `!2.ten_vt%l` |
| Đơn vị tính | left hoặc center | `!2.dvt` |
| Số lượng, tiền | right | `!2.ton_dau`, `!2.du_dau` |
| Header cột | center | `?h_stt`, `?h_ma_vt` |
| Tiêu đề | center | `?title` |

### Format số chuẩn FBO

| Loại | Excel Number Format | Ghi chú |
|---|---|---|
| Số lượng (3 lẻ) | `_(* #,##0.000_);_(* \(#,##0.000\);_(* ""_);_(@_)` | =0 hiện trắng |
| Tiền VNĐ (nguyên) | `_(* #,##0_);_(* \(#,##0\);_(* ""_);_(@_)` | =0 hiện trắng |
| Tiền ngoại tệ | `_(* #,##0.00_);_(* \(#,##0.00\);_(* ""_);_(@_)` | 2 chữ số lẻ |
| Tổng SL (không ẩn 0) | `_(* #,##0.000_);_(* \(#,##0.000\);_(@_)` | =0 vẫn hiện |
| Tổng tiền (không ẩn 0) | `_(* #,##0_);_(* \(#,##0\);_(@_)` | =0 vẫn hiện |

> **Lưu ý**: Dòng data dùng format ẩn giá trị 0 (`""_`), dòng tổng dùng format hiện giá trị 0 (không có phần `""_`).

### Margin (Canh lề trang in)

- Top = Left = Right = Bottom = **0.5 inch**

---

## Merge Cell

### Nguyên tắc

- Tiêu đề (`?title`): merge toàn bộ chiều ngang (A6:T6 hoặc A6:AC6)
- Dòng ngày: merge toàn bộ chiều ngang
- Header cột: merge theo nhóm cột logic
- Dữ liệu: merge theo cột (nếu cột rộng cần nhiều ô)
- Chữ ký: merge theo 2 nhóm (trái + phải)

### Ví dụ merge cột dữ liệu

Khi một cột logic chiếm nhiều ô Excel (VD: "Tên vật tư" chiếm D–F):
- Header: merge `D9:F9`
- Dữ liệu: merge `D10:F10` (framework tự merge khi insert)

---

## Conditional Format (Comment-based)

Dùng **Excel Comment** trên cell cần format theo điều kiện:

```
#kiểu_format:điều_kiện
```

| Ký hiệu | Ý nghĩa |
|---|---|
| `b` | Bold |
| `i` | Italic |
| `u` | Underline |
| `bi` | Bold + Italic |
| `biu` | Bold + Italic + Underline |

**Ví dụ**: Comment `#b:systotal = 0` trên cell `!2.ten_vt%l` → in đậm tên vật tư ở dòng nhóm (systotal=0).

---

## Công thức Excel (Formulas)

### Dòng tổng — SUMIF

```excel
=SUMIF($A$10:A10, ">0", $I$10:I10)
```

- Cột A chứa `stt` — dùng làm tiêu chí: chỉ cộng dòng có stt > 0 (bỏ dòng header nhóm)
- `$A$10:A10` — cố định đầu, thả cuối → framework mở rộng khi insert dòng
- `$I$10:I10` — tương tự

### Dòng tổng — có điều kiện hiện/ẩn

```excel
=IF("!1.in_sl" = "1", SUMIF($A$10:A10, ">0", $I$10:I10), "")
```

- `"!1.in_sl"` — framework thay bằng giá trị thực của cột `in_sl` từ table 1
- Nếu `in_sl = 1` → tính tổng; ngược lại → để trống

### Dòng tổng — không điều kiện (luôn hiện)

```excel
=SUMIF($A$11:A11, ">0", $L$11:L11)
```

> Dùng cho các cột tiền luôn hiện tổng (không phụ thuộc flag).

### Hàm được hỗ trợ

SUM, COUNT, MIN, MAX, AVERAGE, SUMIF, COUNTIF, AVERAGEIF, VLOOKUP, HLOOKUP, IF

---

## Ví dụ hoàn chỉnh — Mẫu đơn giản (chỉ số lượng)

Lấy từ `rptStockSummary_01.xlsx`:

### Kết quả Store trả về

**Table 1** (parameter — 1 dòng):

| date_from | date_to | in_sl | tu_ngay | den_ngay |
|---|---|---|---|---|
| 01/01/2026 | 31/12/2026 | 1 | 01/01/2026 | 31/12/2026 |

**Table 2** (data — nhiều dòng):

| sysorder | sysprint | systotal | stt | ma_vt | ten_vt | dvt | ton_dau | sl_nhap | sl_xuat | ton_cuoi | ... |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | null | null | Tổng cộng | Total | 0.0000 | 1.0000 | ... | ... | ... |
| 1 | 0 | 0 | null | | | | null | null | ... | ... | ... |
| 5 | 1 | 1 | 1 | VT0001 | null | null | 0.0000 | 1.0000 | ... | ... | ... |

### Cấu trúc template

```
Row 1:  A1 = ?Entity_Line1                          (bold)
Row 2:  A2 = ?Entity_Line2
Row 3:  A3 = ?Entity_Line3
Row 4:  A4 = ?Entity_Line4
Row 5:  (trống)
Row 6:  A6:T6 = ?title                              (merge, bold 16pt, center)
Row 7:  A7:T7 = #?h_tu_ngay + + !1.tu_ngay + + ?h_den_ngay + + !1.den_ngay
                                                     (merge, center)
Row 8:  (trống)
Row 9:  [HEADER - fill #EDF5FF, bold, border thin all]
        A9     = ?h_stt
        B9:C9  = ?h_ma_vt                            (merge)
        D9:F9  = ?h_ten_vt                           (merge)
        G9:H9  = ?h_dvt                              (merge)
        I9:K9  = ?h_ton_dau                          (merge)
        L9:N9  = ?h_nhap_u                           (merge)
        O9:Q9  = ?h_xuat_u                           (merge)
        R9:T9  = ?h_ton_cuoi                         (merge)
Row 10: [DATA - border thin L/R, hair bottom]
        A10    = !2.stt                               (center)
        B10:C10= !2.ma_vt{b:systotal=0}              (left)
        D10:F10= !2.ten_vt%l{b:systotal=0}           (left)
        G10:H10= !2.dvt{b:systotal=0}                (left)
        I10:K10= !2.ton_dau{b:systotal=0}            (right, #,##0.000)
        L10:N10= !2.sl_nhap{b:systotal=0}            (right, #,##0.000)
        O10:Q10= !2.sl_xuat{b:systotal=0}            (right, #,##0.000)
        R10:T10= !2.ton_cuoi{b:systotal=0}           (right, #,##0.000)
Row 11: (spacer, height ~20pt)
Row 12: [TOTAL - bold, border-top thin]
        G12:H12= ?total                              (center)
        I12:K12= =IF("!1.in_sl" = "1", SUMIF($A$10:A10, ">0", $I$10:I10), "")
        L12:N12= =IF("!1.in_sl" = "1", SUMIF($A$10:A10, ">0", $L$10:L10), "")
        O12:Q12= =IF("!1.in_sl" = "1", SUMIF($A$10:A10, ">0", $O$10:O10), "")
        R12:T12= =IF("!1.in_sl" = "1", SUMIF($A$10:A10, ">0", $R$10:R10), "")
Row 13: (trống)
Row 14: O14:T14= ?reportDate                         (center)
Row 15: A15:F15= ?chiefAccountant                    (bold 9pt, center)
        O15:T15= ?director                           (bold 9pt, center)
Row 16: A16:F16= ?signatureFullname                  (9pt, center)
        O16:T16= ?signatureFullname                  (9pt, center)
Row 17: (trống, height ~57pt — chừa chỗ ký)
Row 18: A18:F18= ?chiefAccountantName                (bold 9pt, center)
        O18:T18= ?directorName                       (bold 9pt, center)
```

---

## Ví dụ hoàn chỉnh — Mẫu phức tạp (số lượng + giá trị)

Lấy từ `rptStockSummary_02.xlsx` — có 2 dòng header:

### Header 2 dòng

```
Row 9:  [HEADER dòng 1 - nhóm lớn]
        A9     = ?h_stt                               (merge A9:A10 = rowspan 2)
        B9:C10 = ?h_ma_vt                             (merge = rowspan 2)
        D9:G10 = ?h_ten_vt                            (merge = rowspan 2)
        H9:I10 = ?h_dvt                               (merge = rowspan 2)
        J9:N9  = ?h_ton_dau                           (merge = colspan 5)
        O9:S9  = ?h_nhap_u                            (merge = colspan 5)
        T9:X9  = ?h_xuat_u                            (merge = colspan 5)
        Y9:AC9 = ?h_ton_cuoi                          (merge = colspan 5)

Row 10: [HEADER dòng 2 - chi tiết]
        J10:K10= ?h_sl                                (Số lượng)
        L10:N10= ?h_gia_tri                           (Giá trị)
        O10:P10= ?h_sl
        Q10:S10= ?h_gia_tri
        T10:U10= ?h_sl
        V10:X10= ?h_gia_tri
        Y10:Z10= ?h_sl
        AA10:AC10= ?h_gia_tri

Row 11: [DATA]
        A11    = !2.stt
        B11:C11= !2.ma_vt{b:systotal=0}
        D11:G11= !2.ten_vt%l{b:systotal=0}
        H11:I11= !2.dvt{b:systotal=0}
        J11:K11= !2.ton_dau{b:systotal=0}             (#,##0.000)
        L11:N11= !2.du_dau{b:systotal=0}              (#,##0)
        O11:P11= !2.sl_nhap{b:systotal=0}             (#,##0.000)
        Q11:S11= !2.tien_nhap{b:systotal=0}           (#,##0)
        T11:U11= !2.sl_xuat{b:systotal=0}             (#,##0.000)
        V11:X11= !2.tien_xuat{b:systotal=0}           (#,##0)
        Y11:Z11= !2.ton_cuoi{b:systotal=0}            (#,##0.000)
        AA11:AC11= !2.du_cuoi{b:systotal=0}           (#,##0)
```

### Dòng tổng — hỗn hợp

```
Row 13:
  H13:I13 = ?total                                    (center, bold)
  J13:K13 = =IF("!1.in_sl" = "1", SUMIF(...), "")    (có điều kiện — cột SL)
  L13:N13 = =SUMIF($A$11:A11, ">0", $L$11:L11)      (luôn hiện — cột tiền)
  O13:P13 = =IF("!1.in_sl" = "1", SUMIF(...), "")
  Q13:S13 = =SUMIF(...)
  ... (tương tự cho xuất, tồn cuối)
```

> **Quy tắc**: Cột số lượng → dùng `IF("!1.in_sl"...)` có điều kiện. Cột tiền → SUMIF trực tiếp, luôn hiện.

---

## Tạo nhiều mẫu Excel từ cùng layout

Khi báo cáo có nhiều biến thể (VD: chỉ SL, SL+giá trị, SL+giá trị ngoại tệ):

| Mẫu | File template | Khác biệt |
|---|---|---|
| Chỉ số lượng | `rptXxx_01.xlsx` | Ít cột hơn, chỉ có cột SL |
| SL + giá trị | `rptXxx_02.xlsx` | Thêm cột giá trị VNĐ |
| SL + giá trị NT | `rptXxx_02FC.xlsx` | Copy từ _02, đổi `?h_gia_tri` thành `?h_gia_tri` (NT), cột dữ liệu dùng `du_dau_nt`, `tien_nt_n`, ... |

### Cách khai báo form cho biến thể ngoại tệ

Trong Report XML, form ngoại tệ thêm field ghi đè:

```xml
<form id="130" templateFile="rptXxx_02FC" commandArgument="Excel" rowFilter="[2$sysprint = 1]">
  <header v="... tiền ngoại tệ" e="... in Foreign Currency"/>
  <download>
    <header v="... tiền ngoại tệ" e="... in Foreign Currency"/>
  </download>
  <fields>
    <field name="isFC" type="Boolean">
      <header v="True" e="True"/>
    </field>
    <field name="h_gia_tri" type="String">
      <header v="Giá trị nt" e="FC Amount"/>       <!-- ghi đè report-level -->
    </field>
  </fields>
</form>
```

---

## Khai báo trong Report XML — Liên kết form Excel

Tham khảo `SPEC_report-print.md` cho chi tiết. Tóm tắt:

```xml
<form id="110" reportFile="" templateFile="rptXxx_01" commandArgument="Excel"
      urlImage="&e;" rowFilter="[2$sysprint = 1]">
  <header v="..." e="..."/>
  <download><header v="..." e="..."/></download>
  <!-- fields nếu cần parameter riêng -->
</form>
```

| Thuộc tính | Giá trị | Ghi chú |
|---|---|---|
| `reportFile` | `""` (rỗng) | Form Excel không dùng RPT |
| `templateFile` | Tên file xlsx (không extension) | Framework tìm trong thư mục `Controllers/Templates/Excel/` |
| `commandArgument` | `"Excel"` | Bắt buộc |
| `rowFilter` | `"[2$sysprint = 1]"` | Lọc dòng table 2 trước khi đổ |

---

## Checklist tạo file Excel template

1. ☐ Tạo sheet tên bắt đầu bằng `Main`
2. ☐ Dòng 1–4: Entity (`?Entity_Line1..4`)
3. ☐ Dòng tiêu đề: `?title` — merge full, bold 16pt, center
4. ☐ Dòng ngày: nối chuỗi `#?h_tu_ngay + + !1.tu_ngay + + ?h_den_ngay + + !1.den_ngay`
5. ☐ Header cột: fill `#EDF5FF`, bold, thin border all, center
6. ☐ Dữ liệu: `!2.column` hoặc `!2.column{b:systotal=0}`, format số đúng, border thin L/R + hair bottom
7. ☐ Dòng tổng: `?total` + công thức SUMIF, bold, border-top thin
8. ☐ Chữ ký: `?reportDate`, `?chiefAccountant`, `?director`, `?signatureFullname`, `?chiefAccountantName`, `?directorName`
9. ☐ Row height dòng ký ~57pt (chừa chỗ ký tay)
10. ☐ Margin: 0.5 inch all sides
11. ☐ Khai báo form trong Report XML (`templateFile`, `commandArgument="Excel"`, `rowFilter`)
12. ☐ Khai báo `<fields>` parameter nếu cần (`isFC`, `cLan`, ghi đè header)

---

## Lưu ý quan trọng

1. **Sheet phải tên Main** — Nếu tên khác (Sheet1, Data) thì framework không nhận
2. **Biến phân biệt hoa/thường** — `?Entity_Line1` ≠ `?entity_line1`
3. **Chỉ 1 dòng data mẫu** — Framework tự insert thêm dòng mới copy format từ dòng đầu
4. **Merge cell giữ nguyên** — Framework tự xử lý merge khi insert dòng
5. **Công thức SUMIF** — Cố định 1 đầu (`$A$10`), thả 1 đầu (`A10`) để mở rộng đúng
6. **Format số** — Lấy theo thiết kế Excel, không lấy từ Options hệ thống
7. **rowFilter** — `[2$sysprint = 1]` chỉ lọc cho Excel, RPT không áp dụng
8. **File đặt tại** `Web/App_Data/Controllers/Templates/Excel/{templateFile}.xlsx`
