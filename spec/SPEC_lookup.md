# [SPEC_lookup] — File XML Lookup (AutoComplete / Lookup chọn nhiều / Tra cứu)

> Dùng khi: tạo hoặc sử dụng file lookup cho field AutoComplete hoặc Lookup chọn nhiều trên Dir/Filter/Grid Detail.
> Thư mục: Controllers/Lookup/{TênLookup}.xml
> File lookup XML **dùng chung** cho cả `style="AutoComplete"` và `style="Lookup"` — không cần tạo riêng.

---

## Cấu trúc XML

```xml
<?xml version="1.0" encoding="utf-8"?>

<lookup table="{bảng}" code="{cột_mã}" name="{cột_tên}%l" order="{cột_sắp_xếp}" xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Tiêu đề VN" e="Title EN"></header>
  <fields>
    <field name="{cột_mã}" allowSorting="true" allowFilter="true">
      <header v="Mã" e="Code"></header>
    </field>
    <field name="{cột_tên}%l" allowSorting="true" allowFilter="true">
      <header v="Tên" e="Name"></header>
    </field>
    <!-- Có thể thêm cột hiển thị khác -->
  </fields>
</lookup>
```

## Thuộc tính `<lookup>`

| Thuộc tính | Mô tả |
|---|---|
| `table` | Bảng hoặc view chứa dữ liệu |
| `code` | Cột mã (giá trị trả về khi chọn) |
| `name` | Cột tên hiển thị. Thường có `%l` để song ngữ |
| `order` | Sắp xếp |

---

## ⭐ Quy trình tra cứu Lookup (Lookup Resolution)

Khi cần lookup cho một field AutoComplete **hoặc** Lookup chọn nhiều, Claude thực hiện **theo thứ tự**:

### Bước 1 — Kiểm tra danh sách Lookup thường dùng (bên dưới)

File này chứa các lookup **thường xuyên dùng** (Customer, Item, Site, Currency, Unit, Job, Account, Department...).

→ Nếu tìm thấy controller phù hợp → **dùng ngay**, không cần tạo mới.

### Bước 2 — Kiểm tra `SPEC_lookup-other.md`

File chứa các lookup **ít dùng hơn** (nhóm danh mục, nhân viên bán hàng, ngân hàng, thuế, chiết khấu...). Do lượng lookup khá nhiều nên tách riêng file.

→ Nếu tìm thấy → **dùng ngay**.

### Bước 3 — Lookup đặc thù dự án (Custom Lookup)

Một số dự án có lookup riêng không nằm trong danh sách chung. Khi đó prompt sẽ cung cấp đủ thông tin:

```
Lookup: {TênController}
  - Bảng: {tên_bảng}
  - Mã: {cột_mã}
  - Tên: {cột_tên}
  - [Cột thêm: {cột_khác} nếu có]
```

→ Claude tạo file `lookup_{TênController}.xml` theo cấu trúc chuẩn ở trên.

**Ví dụ prompt:**
```
Field ma_dvcs dùng lookup:
  Controller: CompanyUnit
  Bảng: dmdvcs
  Mã: ma_dvcs
  Tên: ten_dvcs
```

→ Claude tạo:
```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmdvcs" code="ma_dvcs" name="ten_dvcs%l" order="ma_dvcs" xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục đơn vị cơ sở" e="Company Unit List"></header>
  <fields>
    <field name="ma_dvcs" allowSorting="true" allowFilter="true">
      <header v="Mã ĐVCS" e="Unit ID"></header>
    </field>
    <field name="ten_dvcs%l" allowSorting="true" allowFilter="true">
      <header v="Tên ĐVCS" e="Unit Name"></header>
    </field>
  </fields>
</lookup>
```

### Bước 4 — Không có thông tin

Nếu prompt **không cung cấp** thông tin lookup cho field FK lạ → Claude **hỏi lại**:

> Field `{tên_field}` cần AutoComplete nhưng chưa có lookup. Vui lòng cung cấp: Controller name, bảng, cột mã, cột tên.

---

## Lookup có cột bổ sung (nhiều hơn mã + tên)

Một số lookup cần hiển thị thêm cột (ví dụ: ĐVT, nhóm, địa chỉ...):

```xml
<lookup table="dmvt" code="ma_vt" name="ten_vt%l" order="ma_vt" xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục vật tư" e="Item List"></header>
  <fields>
    <field name="ma_vt" allowSorting="true" allowFilter="true">
      <header v="Mã vật tư" e="Item ID"></header>
    </field>
    <field name="ten_vt%l" allowSorting="true" allowFilter="true">
      <header v="Tên vật tư" e="Item Name"></header>
    </field>
    <field name="dvt" allowSorting="true" allowFilter="true">
      <header v="ĐVT" e="UOM"></header>
    </field>
  </fields>
</lookup>
```

## Lookup có điều kiện lọc (key trên items)

Khi field AutoComplete trên Dir/Filter cần lọc dữ liệu lookup:

```xml
<items style="AutoComplete" controller="Customer" reference="ten_kh%l"  key="status = '1'" check="1 = 1"  information="ma_kh$dmkh.ten_kh%l" new="Default"/>
```

- `key="status = '1'"` → chỉ hiện bản ghi active
- `check="1 = 1"` → không giới hạn thêm
- `information` → cột mã + bảng.cột tên (để hiển thị khi load lại form)

## Lookup nhóm danh mục (có loai_nh)

Nhiều bảng nhóm dùng chung cấu trúc `(loai_nh, ma_nh, ten_nh)`. Mỗi `loai_nh` là một loại nhóm riêng. Lọc bằng `key` trên field AutoComplete:

```xml
<!-- Nhóm VV loại 1 -->
<items style="AutoComplete" controller="JobGroup" reference="ten_nh%l"  key="loai_nh = 1 and status = '1'" check="1 = 1" information="ma_nh$dmnhvv.ten_nh%l" new="Default"/>
```

→ Cùng 1 file lookup `JobGroup.xml`, field khác nhau dùng `key` khác nhau để lọc `loai_nh`.

---

## ✅ Checklist khi gặp field tra cứu (AutoComplete hoặc Lookup chọn nhiều)

### Bước 0 — Xác định style từ prompt

| Prompt ghi | Style | Pattern |
|---|---|---|
| "lookup chọn nhiều từ ..." | `style="Lookup"` | Không `reference`, không field tên kèm |
| Không ghi "chọn nhiều" | `style="AutoComplete"` | Có `reference`, `information`, field tên kèm |

> Chi tiết khai báo Lookup chọn nhiều: xem `SPEC_lookup-multiselect.md`

### Bước 1–4 — Tra cứu controller (giống nhau cho cả 2 style)

1. Xác định controller name từ `<items ... controller="...">`
2. Kiểm tra bảng **Lookup thường dùng** bên dưới → có thì dùng ngay
3. Không có → tra `SPEC_lookup-other.md`
4. Vẫn không có → kiểm tra prompt có cung cấp thông tin custom lookup không
5. Không có thông tin → hỏi lại user
6. Khi tạo mới: file đặt tên `lookup_{Controller}.xml`

> **File lookup XML dùng chung** — cùng 1 file `Account.xml` phục vụ cả AutoComplete lẫn Lookup chọn nhiều.

---

## ⭐ Danh sách Lookup thường dùng

> Các lookup xuất hiện trong hầu hết các module.
> Nếu controller cần dùng có trong bảng dưới → **KHÔNG tạo mới**.

| Controller | Bảng | Mã (`code`) | Tên (`name`) | Ghi chú |
|---|---|---|---|---|
| `Customer` | dmkh | ma_kh | ten_kh%l | Khách hàng |
| `Item` | dmvt | ma_vt | ten_vt%l | Vật tư / Hàng hóa |
| `Site` | dmkho | ma_kho | ten_kho%l | Kho |
| `Currency` | dmnt | ma_nt | ten_nt%l | Loại tiền |
| `UOM` | dmdvt | dvt | ten_dvt%l | Đơn vị tính |
| `Job` | dmvv | ma_vv | ten_vv%l | Vụ việc / Dự án |
| `Account` | dmtk | tk | ten_tk%l | Tài khoản kế toán |
| `Department` | dmbp | ma_bp | ten_bp%l | Bộ phận / Phòng ban |
| `Unit` | dmdvcs | ma_dvcs | ten_dvcs%l | Đơn vị |

---

## XML mẫu từng Lookup thường dùng

### Customer

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmkh" code="ma_kh" name="ten_kh%l" order="ma_kh"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục khách hàng" e="Customer List"></header>
  <fields>
    <field name="ma_kh" allowSorting="true" allowFilter="true">
      <header v="Mã khách" e="Customer ID"></header>
    </field>
    <field name="ten_kh%l" allowSorting="true" allowFilter="true">
      <header v="Tên khách hàng" e="Customer Name"></header>
    </field>
  </fields>
</lookup>
```

### Item

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmvt" code="ma_vt" name="ten_vt%l" order="ma_vt"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục vật tư" e="Item List"></header>
  <fields>
    <field name="ma_vt" allowSorting="true" allowFilter="true">
      <header v="Mã vật tư" e="Item ID"></header>
    </field>
    <field name="ten_vt%l" allowSorting="true" allowFilter="true">
      <header v="Tên vật tư" e="Item Name"></header>
    </field>
  </fields>
</lookup>
```

### Site

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmkho" code="ma_kho" name="ten_kho%l" order="ma_kho"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục kho" e="Site List"></header>
  <fields>
    <field name="ma_kho" allowSorting="true" allowFilter="true">
      <header v="Mã kho" e="Site ID"></header>
    </field>
    <field name="ten_kho%l" allowSorting="true" allowFilter="true">
      <header v="Tên kho" e="Site Name"></header>
    </field>
  </fields>
</lookup>
```

### Currency

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmnt" code="ma_nt" name="ten_nt%l" order="ma_nt"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục ngoại tệ" e="Currency List"></header>
  <fields>
    <field name="ma_nt" allowSorting="true" allowFilter="true">
      <header v="Mã ngoại tệ" e="Currency ID"></header>
    </field>
    <field name="ten_nt%l" allowSorting="true" allowFilter="true">
      <header v="Tên ngoại tệ" e="Currency Name"></header>
    </field>
  </fields>
</lookup>
```

### UOM

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmdvt" code="dvt" name="ten_dvt%l" order="dvt"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục đơn vị tính" e="Unit of Measure List"></header>
  <fields>
    <field name="dvt" allowSorting="true" allowFilter="true">
      <header v="Đơn vị tính" e="UOM"></header>
    </field>
    <field name="ten_dvt%l" allowSorting="true" allowFilter="true">
      <header v="Tên đơn vị tính" e="UOM Name"></header>
    </field>
  </fields>
</lookup>
```

### Job

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmvv" code="ma_vv" name="ten_vv%l" order="ma_vv"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục vụ việc" e="Job List"></header>
  <fields>
    <field name="ma_vv" allowSorting="true" allowFilter="true">
      <header v="Mã vụ việc" e="Job ID"></header>
    </field>
    <field name="ten_vv%l" allowSorting="true" allowFilter="true">
      <header v="Tên vụ việc" e="Job Name"></header>
    </field>
  </fields>
</lookup>
```

### Account

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmtk" code="tk" name="ten_tk%l" order="tk"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục tài khoản" e="Account List"></header>
  <fields>
    <field name="tk" allowSorting="true" allowFilter="true">
      <header v="Tài khoản" e="Account"></header>
    </field>
    <field name="ten_tk%l" allowSorting="true" allowFilter="true">
      <header v="Tên tài khoản" e="Account Name"></header>
    </field>
  </fields>
</lookup>
```

### Department

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmbp" code="ma_bp" name="ten_bp%l" order="ma_bp"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục bộ phận" e="Department List"></header>
  <fields>
    <field name="ma_bp" allowSorting="true" allowFilter="true">
      <header v="Mã bộ phận" e="Dept. ID"></header>
    </field>
    <field name="ten_bp%l" allowSorting="true" allowFilter="true">
      <header v="Tên bộ phận" e="Department Name"></header>
    </field>
  </fields>
</lookup>
```

---

## Cách dùng trong Dir/Filter (tham khảo nhanh)

```xml
<!-- Field AutoComplete dùng lookup Customer -->
<field name="ma_kh">
  <header v="Mã khách" e="Customer"></header>
  <items style="AutoComplete" controller="Customer" reference="ten_kh%l"
    key="status = '1'" check="1 = 1"
    information="ma_kh$dmkh.ten_kh%l" new="Default"/>
  <clientScript><![CDATA[onchange="onChange${Controller}$Customer(this);"]]></clientScript>
</field>
<field name="ten_kh%l" readOnly="true" external="true" defaultValue="''">
  <header v="" e=""></header>
</field>
```

> Thay `${Controller}` bằng tên Controller thực tế (VD: `Bank`, `Job`, `SalesPrice`).
