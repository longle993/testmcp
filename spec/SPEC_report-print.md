# [SPEC_report-print] — Report XML cho Mẫu in Báo cáo

> Dùng khi: tạo file Report XML khai báo mẫu in cho báo cáo.
> Thư mục: `Web/App_Data/Controllers/Report/{Controller}.xml`
> File template `.rpt` (nếu có): `Web/App_Data/Controllers/Templates/Rpt/{templateFile}.rpt`
> Đọc `SPEC_report-overview.md` trước.

---

## Khác biệt so với Report chứng từ

| Đặc điểm | Report chứng từ | **Report báo cáo** |
|---|---|---|
| `<query>` | Có — query lại DB khi in | **Không có** — dùng dataset từ Filter Processing |
| Dữ liệu | Lấy mới từ DB (stt_rec) | **Dùng dataset đã lọc ở Grid** |
| Entity | Controller, Profile, External, VisibleField | **Đơn giản hơn — chỉ b, f, s, p, e** |
| Mẫu mặc định | 4 form (Rpt VND, Rpt Song ngữ, Excel VND, Excel Song ngữ) | **4 form (Rpt VND, Rpt Ngoại tệ, Excel VND, Excel Ngoại tệ)** |

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE report [
  <!-- Entity chuẩn -->
  <!ENTITY b SYSTEM ".\Include\BaseCurrency.xml">
  <!ENTITY f SYSTEM ".\Include\ForeignCurrency.xml">
  <!ENTITY s SYSTEM ".\Include\Separate.xml">
  <!ENTITY p "../images/pdf.gif">
  <!ENTITY e "../images/excel.gif">
]>

<report xmlns="urn:schemas-fast-com:data-report">
  <forms> ... </forms>
  <fields> ... </fields>
</report>
```

> **Không có `<query>`** — Report báo cáo lấy dữ liệu từ dataset sau khi lọc (Filter Processing), không query lại DB.

---

## Entity — Giải thích

| Entity | Giá trị | Mô tả |
|---|---|---|
| `b` | `BaseCurrency.xml` | Mẫu in VNĐ — ẩn cột ngoại tệ |
| `f` | `ForeignCurrency.xml` | Mẫu in ngoại tệ — hiện cột ngoại tệ |
| `s` | `Separate.xml` | Dòng phân cách giữa nhóm PDF và nhóm Excel |
| `p` | `../images/pdf.gif` | Icon PDF |
| `e` | `../images/excel.gif` | Icon Excel |

---

## `<forms>` — Khai báo mẫu in

### Thuộc tính `<form>`

| Thuộc tính | Mô tả |
|---|---|
| `id` | Mã mẫu: `010`/`020` cho Rpt (PDF), `110`/`120` cho Excel |
| `reportFile` | Tên file RPT (không extension). Dùng cho PDF |
| `templateFile` | Tên file Excel template (không extension). Dùng cho Excel |
| `commandArgument` | `"Pdf"` hoặc `"Excel"` |
| `urlImage` | Icon: `&p;` (PDF) hoặc `&e;` (Excel). Form thứ 2 trong nhóm thường không có (ẩn icon) |
| `rowFilter` | Điều kiện lọc dòng trước khi đổ vào Excel. VD: `"[2$sysprint = 1]"` |

### Quy ước id

| id | Loại | Mô tả |
|---|---|---|
| `010` | PDF | Mẫu VNĐ |
| `020` | PDF | Mẫu ngoại tệ |
| `110` | Excel | Mẫu VNĐ |
| `120` | Excel | Mẫu ngoại tệ |

### Khai báo 4 form mặc định

```xml
<forms>
  <!-- === Nhóm PDF === -->
  <form id="010" reportFile="{Controller}_01" commandArgument="Pdf" urlImage="&p;">
    <header v="{Tên báo cáo}" e="{Report Name}"/>
    <download>
      <header v="{Tên báo cáo}" e="{Report Name}"/>
    </download>&b;
  </form>

  <form id="020" reportFile="{Controller}_01" commandArgument="Pdf">
    <header v="{Tên báo cáo} tiền ngoại tệ" e="{Report Name} in Foreign Currency"/>
    <download>
      <header v="{Tên báo cáo} tiền ngoại tệ" e="{Report Name} in Foreign Currency"/>
    </download>&f;
  </form>

  <!-- Dòng phân cách -->
  &s;

  <!-- === Nhóm Excel === -->
  <form id="110" templateFile="{Controller}_01" commandArgument="Excel"
        urlImage="&e;" rowFilter="[2$sysprint = 1]">
    <header v="{Tên báo cáo}" e="{Report Name}"/>
    <download>
      <header v="{Tên báo cáo}" e="{Report Name}"/>
    </download>
    <fields>
      <field name="isFC" type="Boolean">
        <header v="False" e="False"/>
      </field>
      <field name="cLan" type="String">
        <header v="v" e="e"/>
      </field>
    </fields>
  </form>

  <form id="120" templateFile="{Controller}_01" commandArgument="Excel"
        rowFilter="[2$sysprint = 1]">
    <header v="{Tên báo cáo} tiền ngoại tệ" e="{Report Name} in Foreign Currency"/>
    <download>
      <header v="{Tên báo cáo} tiền ngoại tệ" e="{Report Name} in Foreign Currency"/>
    </download>
    <fields>
      <field name="isFC" type="Boolean">
        <header v="True" e="True"/>
      </field>
      <field name="cLan" type="String">
        <header v="v" e="e"/>
      </field>
    </fields>
  </form>
</forms>
```

### Giải thích

- **Nhóm PDF (010, 020)**: dùng `reportFile` trỏ tới file `.rpt`. Form `010` có `urlImage` (hiện icon), form `020` không có (ẩn icon — nhóm cùng icon trên). `&b;` / `&f;` là entity điều khiển VNĐ / ngoại tệ
- **`&s;`**: phân cách giữa nhóm PDF và Excel trên menu in
- **Nhóm Excel (110, 120)**: dùng `templateFile` trỏ tới file `.xlsx`. Có `rowFilter` để lọc dòng. Có `<fields>` riêng truyền parameter (`isFC`, `cLan`) vào template

### Về `rowFilter`

`rowFilter="[2$sysprint = 1]"` → với bảng thứ 2 (tính từ 1), chỉ lấy dòng có `sysprint = 1` khi đổ dữ liệu vào Excel.

Nếu nhiều bảng cần lọc: `rowFilter="[1$key],[2$key],[3$key]"`

### Về parameter fields trong form

```xml
<fields>
  <field name="isFC" type="Boolean">
    <header v="False" e="False"/>   <!-- Giá trị default -->
  </field>
  <field name="cLan" type="String">
    <header v="v" e="e"/>            <!-- v=Vietnamese, e=English -->
  </field>
</fields>
```

- `isFC`: flag ngoại tệ — `False` cho VNĐ, `True` cho ngoại tệ
- `cLan`: ngôn ngữ hiện hành — `v` hoặc `e`
- Giá trị nằm trong `<header v="..." e="..."/>` — `v` cho tiếng Việt, `e` cho tiếng Anh

---

## `<fields>` — Parameter text cố định cho mẫu in

Các field text song ngữ (VN + EN) dùng cho header/label trong mẫu in.

```xml
<fields>
  <!-- Tiêu đề báo cáo -->
  <field name="title" type="String">
    <header v="{TIÊU ĐỀ TIẾNG VIỆT}" e="{ENGLISH TITLE}"/>
  </field>

  <!-- Header cột / label -->
  <field name="h_ngay" type="String">
    <header v="Ngày" e="Date"/>
  </field>
  <field name="h_so" type="String">
    <header v="Số" e="Number"/>
  </field>
  <field name="h_dien_giai" type="String">
    <header v="Diễn giải" e="Description"/>
  </field>
  <field name="h_ps_no" type="String">
    <header v="Phát sinh nợ" e="Debit"/>
  </field>
  <field name="h_ps_co" type="String">
    <header v="Phát sinh có" e="Credit"/>
  </field>
  <field name="h_tk" type="String">
    <header v="Tài khoản" e="Account"/>
  </field>
  <field name="h_tk_doi_ung" type="String">
    <header v="Tk đối ứng" e="Reference Account"/>
  </field>
  <field name="h_ten_kh" type="String">
    <header v="Tên khách" e="Customer Name"/>
  </field>

  <!-- Label ngày từ/đến (dùng cho dòng header mẫu in) -->
  <field name="h_tu_ngay" type="String">
    <header v="Từ ngày" e="Date from"/>
  </field>
  <field name="h_den_ngay" type="String">
    <header v="đến ngày" e="to"/>
  </field>

  <!-- Thêm field theo layout mẫu in cụ thể -->
</fields>
```

### Quy ước đặt tên field parameter

| Prefix | Mô tả | Ví dụ |
|---|---|---|
| `title` | Tiêu đề báo cáo | `title` |
| `h_` | Header cột / label | `h_ngay`, `h_so_ct`, `h_dien_giai` |
| `h_line1..4` | Dòng thông tin phụ trên header | `h_line1` = "Mẫu số 01 - TT" |
| `h_tu_ngay` / `h_den_ngay` | Label ngày từ/đến | |

---

## Mẫu in Excel — Cấu trúc file template

### Quy tắc ký hiệu trong Excel

| Ký hiệu | Cú pháp | Mô tả |
|---|---|---|
| **Parameter text** | `?field_name` | Lấy giá trị từ `<fields>` (Report XML hoặc Form XML). VD: `?title`, `?h_ngay` |
| **Entity hệ thống** | `?Entity_Line1..5` | Tên tập đoàn/công ty/địa chỉ từ file Entity.lic |
| **Dữ liệu bảng** | `!{N}.column_name` | Lấy từ bảng N (tính từ 1). VD: `!1.date_from`, `!2.ma_vt` |
| **Dữ liệu song ngữ** | `!{N}.column%l` | Tự chọn ngôn ngữ. VD: `!2.ten_kh%l` |
| **Nối chuỗi** | `#?text + + !1.field` | Nối parameter + dữ liệu. VD: `#?h_tu_ngay + + !1.date_from` |
| **Dữ liệu số theo loại tiền** | `!{N}.column%c` | Tự format theo VNĐ/ngoại tệ |

### Ưu tiên lấy parameter text

1. `<fields>` trong `<form>` (mẫu cụ thể)
2. `<fields>` trong `<report>` (chung cho tất cả mẫu)
3. File `Report.xml` trong thư mục Options (hệ thống)

### Sheet naming

- Sheet bắt đầu bằng `"Main"` → được xử lý đổ dữ liệu: `Main`, `Main_01`, `MainA`
- Sheet không bắt đầu bằng Main → bỏ qua: `ABC`, `Note`

### Format cell

- Format cell trong template giữ nguyên khi xuất
- Merge cell giữ nguyên vị trí
- Bảng nhiều dòng: insert dòng mới cùng format với dòng đầu
- Format trường số lấy theo thiết kế Excel (không lấy từ options)

### Format số chuẩn FBO

| Loại | Excel Format |
|---|---|
| Số (=0 thì trắng) | `_(* #,##0_);_(* (#,##0);_(* ""_);_(@_)` |
| Số lượng 3 số lẻ | `_(* #,##0.000_);_(* (#,##0.000);_(* ""_);_(@_)` |
| Tiền VNĐ | `_(* #,##0_);_(* (#,##0);_(* ""_);_(@_)` |
| Tiền ngoại tệ | `_(* #,##0.00_);_(* (#,##0.00);_(* ""_);_(@_)` |
| Tỷ giá | `_(* #,##0.00_);_(* (#,##0.00);_(* ""_);_(@_)` |

### Canh lề chuẩn

- Top = Left = Right = Bottom = **0.5**
- Màu nền tiêu đề: **237 - 245 - 255** (RGB)
- Focus con trỏ: tại cell tiêu đề báo cáo

### Conditional format (Bold/Italic/Underline theo điều kiện)

Thêm **Comment** vào cell cần format điều kiện:

```
#kiểu_format:điều_kiện
```

- `kiểu_format`: `b` (bold), `i` (italic), `u` (underline), hoặc kết hợp: `bi`, `biu`
- `điều_kiện`: biểu thức trên dòng. VD: `systotal = 0`

Ví dụ: `#b:systotal = 0` → in đậm các dòng nhóm (systotal = 0).

### Công thức Excel

- Công thức dòng tổng: cố định 1 cell, thả lỏng cell kia để mở rộng
- VD: `=SUMIF($A$10:A10, ">0", $E$10:E10)`
- Hỗ trợ: Sum, Count, Min, Max, Average, SumIf, CountIf, AverageIf, Vlookup, Hlookup

---

## Ví dụ hoàn chỉnh

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE report [
  <!ENTITY b SYSTEM ".\Include\BaseCurrency.xml">
  <!ENTITY f SYSTEM ".\Include\ForeignCurrency.xml">
  <!ENTITY s SYSTEM ".\Include\Separate.xml">
  <!ENTITY p "../images/pdf.gif">
  <!ENTITY e "../images/excel.gif">
]>

<report xmlns="urn:schemas-fast-com:data-report">
  <forms>
    <form id="010" reportFile="rptMyReport_01" commandArgument="Pdf" urlImage="&p;">
      <header v="Báo cáo tổng hợp" e="Summary Report"/>
      <download>
        <header v="Báo cáo tổng hợp" e="Summary Report"/>
      </download>&b;
    </form>

    <form id="020" reportFile="rptMyReport_01" commandArgument="Pdf">
      <header v="Báo cáo tổng hợp tiền ngoại tệ" e="Summary Report in Foreign Currency"/>
      <download>
        <header v="Báo cáo tổng hợp tiền ngoại tệ" e="Summary Report in Foreign Currency"/>
      </download>&f;
    </form>

    &s;

    <form id="110" templateFile="rptMyReport_01" commandArgument="Excel"
          urlImage="&e;" rowFilter="[2$sysprint = 1]">
      <header v="Báo cáo tổng hợp" e="Summary Report"/>
      <download>
        <header v="Báo cáo tổng hợp" e="Summary Report"/>
      </download>
      <fields>
        <field name="isFC" type="Boolean">
          <header v="False" e="False"/>
        </field>
        <field name="cLan" type="String">
          <header v="v" e="e"/>
        </field>
      </fields>
    </form>

    <form id="120" templateFile="rptMyReport_01" commandArgument="Excel"
          rowFilter="[2$sysprint = 1]">
      <header v="Báo cáo tổng hợp tiền ngoại tệ" e="Summary Report in Foreign Currency"/>
      <download>
        <header v="Báo cáo tổng hợp tiền ngoại tệ" e="Summary Report in Foreign Currency"/>
      </download>
      <fields>
        <field name="isFC" type="Boolean">
          <header v="True" e="True"/>
        </field>
        <field name="cLan" type="String">
          <header v="v" e="e"/>
        </field>
      </fields>
    </form>
  </forms>

  <fields>
    <field name="title" type="String">
      <header v="BÁO CÁO TỔNG HỢP" e="SUMMARY REPORT"/>
    </field>
    <field name="h_ngay" type="String">
      <header v="Ngày" e="Date"/>
    </field>
    <field name="h_so" type="String">
      <header v="Số" e="Number"/>
    </field>
    <field name="h_dien_giai" type="String">
      <header v="Diễn giải" e="Description"/>
    </field>
    <field name="h_tu_ngay" type="String">
      <header v="Từ ngày" e="Date from"/>
    </field>
    <field name="h_den_ngay" type="String">
      <header v="đến ngày" e="to"/>
    </field>
  </fields>
</report>
```

---

## Lưu ý quan trọng

1. **Không có `<query>`** — Report báo cáo dùng dataset từ Filter Processing, không query lại DB. Trường hợp có query riêng là đặc thù, sẽ xử lý riêng
2. **Mặc định 4 form**: 2 Rpt (VNĐ + ngoại tệ) + 2 Excel (VNĐ + ngoại tệ)
3. **`&s;` phân cách** giữa nhóm Rpt và nhóm Excel
4. **`rowFilter`** chỉ có tác dụng cho Excel, không áp dụng cho PDF
5. **Form PDF thứ 2 (020)** không có `urlImage` — ẩn icon, nhóm chung với form 010
6. **Form Excel** cần `<fields>` riêng cho parameter `isFC` và `cLan`
7. **File template Excel** đặt tại `Web/App_Data/Controllers/Templates/Excel/{templateFile}.xlsx`
8. **File RPT** đặt tại thư mục Report tương ứng
9. **Chi tiết cách tạo file Excel template** → xem `SPEC_report-excel.md` (layout, biến, format, công thức)
