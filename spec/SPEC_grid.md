# [SPEC_grid] — File XML Grid (Màn hình Browse)

> Dùng khi: tạo file Grid XML hiển thị danh sách dữ liệu.
> Thư mục: Controllers/Grid/{TênController}.xml

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE grid [
  <!ENTITY Controller "{TênController}">
]>

<grid table="{tên_bảng_hoặc_view}" code="{danh_sách_khóa}"
      order="{danh_sách_sắp_xếp}" xmlns="urn:schemas-fast-com:data-grid">
  <title />
  <subTitle />      <!-- tùy chọn -->
  <fields> ... </fields>
  <views> ... </views>
  <commands> ... </commands>
  <script> ... </script>
  <toolbar> ... </toolbar>  <!-- tùy chọn, mặc định có sẵn -->
</grid>
```

## Thuộc tính `<grid>`

| Thuộc tính | Bắt buộc | Mô tả |
|---|---|---|
| `table` | ✅ | Bảng hoặc **View** SQL. Dùng view (prefix `v`) khi cần join tên |
| `code` | ✅ | Danh sách cột khóa, phân cách bằng dấu phẩy |
| `order` | ✅ | Cột sắp xếp, phân cách bằng dấu phẩy |
| `filter` | | Điều kiện lọc mặc định (thường từ ENTITY) |
| `initialize` | | Query khởi tạo (thường từ ENTITY) |
| `xmlns` | ✅ | Cố định: `urn:schemas-fast-com:data-grid` |

## `<title>` và `<subTitle>`

> ⚠️ **BẮT BUỘC**: Grid danh mục (Browse) **luôn phải có cả `<title>` và `<subTitle>`**.
> Thiếu `<subTitle>` sẽ khiến giao diện thiếu mô tả chức năng.

### Cú pháp

```xml
<title v="Tên danh mục tiếng Việt" e="English Catalog Name"></title>
<subTitle v="Cập nhật {tên}: thêm, sửa, xóa..." e="Add, Edit, Delete {Name}..."></subTitle>
```

### Ví dụ danh mục thông thường (KHÔNG có placeholder)

```xml
<title v="Danh mục ngân hàng" e="Bank List"></title>
<subTitle v="Cập nhật ngân hàng: thêm, sửa, xóa..." e="Add, Edit, Delete Bank..."></subTitle>
```

```xml
<title v="Danh mục hàng hóa - vật tư" e="Item List"></title>
<subTitle v="Cập nhật hàng hóa, vật tư: thêm, sửa, xóa..." e="Add, Edit, Delete Item..."></subTitle>
```

### Ví dụ có placeholder (khi Grid có Filter truyền giá trị)

```xml
<title v="Cập nhật giá bán" e="Sales Price List"></title>
<subTitle v="Loại giá bán: %s1 - %s2" e="Sales Pricing Type: %s1 - %s2"></subTitle>
```

- `%s1`, `%s2`, `%d1`, `%d2`: placeholder, gán giá trị từ JS qua `g._alterTitle`

### Quy tắc đặt subTitle

| Loại Grid | subTitle mẫu |
|---|---|
| Danh mục thông thường | `"Cập nhật {tên}: thêm, sửa, xóa..."` |
| Danh mục có Filter | `"Loại {x}: %s1 - %s2"` hoặc `"Kỳ %s1, năm %s2..."` |
| Grid Detail (`type="Detail"`) | Để trống: `v="" e=""` |
| Grid Report (`type="Report"`) | Mô tả phạm vi: `"Từ ngày %d1 đến ngày %d2..."` |

## `<fields>` — Khai báo cột Grid

```xml
<field name="{tên_cột}" [isPrimaryKey="true"] width="{px}" [type="{kiểu}"] [dataFormatString="{format}"] [allowSorting="true"] [allowFilter="true"] [align="right"] [hidden="true"] [aggregate="Sum"][readOnly="true"] [external="true"] [hyperlinkFormatString="..."]>
  <header v="Nhãn VN" e="Label EN"></header>
</field>
```

### Thuộc tính field trong Grid

| Thuộc tính | Mô tả |
|---|---|
| `name` | Tên cột trong bảng/view |
| `isPrimaryKey` | Là khóa chính |
| `width` | Chiều rộng cột (px). `width="0"` = ẩn |
| `type` | Kiểu: `DateTime`, `Decimal`, `Int16`, `Boolean` |
| `dataFormatString` | `@datetimeFormat`, `@upperCaseFormat`, `@foreignCurrencyPriceViewFormat`, `@baseCurrencyAmountViewFormat`, `@quantityViewFormat`, `X` (viết hoa) |
| `allowSorting` | Cho phép sắp xếp |
| `allowFilter` | Cho phép lọc |
| `align` | `left`, `right`, `center` |
| `hidden` | Ẩn hoàn toàn |
| `aggregate` | `Sum`, `Count`, `Average`, `Max`, `Min` |
| `readOnly` | Chỉ đọc |
| `external` | Trường giả (không có trong bảng DB, chỉ hiển thị) |
| `hyperlinkFormatString` | Tạo link drill-down |

### ⚠️ Hướng dẫn `width` theo loại dữ liệu

> **Quy tắc quan trọng**: Cột tiền, giá, số lượng phải có width **nhỏ hơn** cột mã/tên. Không được để width bằng nhau cho tất cả.

| Loại cột | width khuyến nghị | Ví dụ |
|---|---|---|
| **Mã (code)** | `100` – `120` | `ma_kh`, `ma_vt`, `ma_nh` |
| **Tên (name)** | `250` – `350` | `ten_kh%l`, `ten_vt%l` |
| **Mô tả / Ghi chú** | `250` – `350` | `dien_giai`, `ghi_chu` |
| **Số lượng** | `80` – `100` | `so_luong` |
| **Đơn giá** | `90` – `110` | `gia_nt`, `gia_nt2` |
| **Thành tiền / Tiền** | `110` – `130` | `tien_nt`, `tien`, `ps_no`, `ps_co` |
| **Tỷ giá** | `80` – `100` | `ty_gia` |
| **Hạn mức** | `110` – `130` | `han_muc`, `han_muc_lc` |
| **Ngày** | `90` – `110` | `ngay_ct`, `ngay_vv` |
| **Trạng thái / Flag** | `60` – `80` | `status`, `phan_loai` |
| **ĐVT** | `50` – `70` | `dvt` |
| **Số chứng từ** | `80` – `100` | `so_ct` |

**Ví dụ ĐÚNG:**
```xml
<field name="ma_vt" width="100" .../>     <!-- mã: 100px -->
<field name="ten_vt%l" width="300" .../>  <!-- tên: 300px -->
<field name="so_luong" width="80" .../>   <!-- số lượng: 80px -->
<field name="gia_nt" width="90" .../>     <!-- đơn giá: 90px -->
<field name="tien_nt" width="120" .../>   <!-- thành tiền: 120px -->
<field name="han_muc" width="120" .../>   <!-- hạn mức: 120px -->
```

**Ví dụ SAI (tất cả cùng width):**
```xml
<!-- ❌ KHÔNG LÀM THẾ NÀY — tiền/giá/số lượng width bằng mã -->
<field name="ma_vt" width="200" .../>
<field name="so_luong" width="200" .../>
<field name="gia_nt" width="200" .../>
<field name="tien_nt" width="200" .../>
```

### Pattern field tên hiển thị (từ View SQL)

```xml
<!-- Cột mã -->
<field name="ma_kh" isPrimaryKey="true" width="100" allowSorting="true" allowFilter="true">
  <header v="Mã khách" e="Customer ID"></header>
</field>
<!-- Cột tên (từ view join) -->
<field name="ten_kh%l" width="300" allowSorting="true" allowFilter="true">
  <header v="Tên khách" e="Customer Name"></header>
</field>
```

## `<views>` — Cột hiển thị trên Browse

```xml
<views>
  <view id="Grid">
    <field name="ma_kh"/>
    <field name="ten_kh%l"/>
    <field name="ngay_ban"/>
    <field name="gia_nt2"/>
    <field name="ma_nt"/>
  </view>
</views>
```

- `id` luôn = `"Grid"`
- Thứ tự `<field>` trong view = thứ tự cột hiển thị

## `<commands>` — Events

### Danh mục đơn giản (Loading + Closing)

```xml
<commands>
  <command event="Loading">
    <text><![CDATA[select 'load$Grid(this);' as message
return]]></text>
  </command>
  <command event="Closing">
    <text><![CDATA[select 'dispose$Grid(this);' as message
return]]></text>
  </command>
</commands>
```

### Danh mục có Filter (Loading truyền thêm biến)

```xml
<command event="Loading">
  <text><![CDATA[
select 'load$Grid(this);' as message
return
]]></text>
</command>
```

## `<toolbar>` — Thanh công cụ

Nếu không khai báo, framework dùng mặc định. Khai báo tùy chỉnh:

```xml
<toolbar>
  <button command="New"><title v="Toolbar.New" e="Toolbar.New"/></button>
  <button command="Edit"><title v="Toolbar.Edit" e="Toolbar.Edit"/></button>
  <button command="Delete"><title v="Toolbar.Delete" e="Toolbar.Delete"/></button>
  <button command="Search"><title v="Toolbar.Search" e="Toolbar.Search"/></button>
  <button command="View"><title v="Toolbar.View" e="Toolbar.View"/></button>
  <button command="Export"><title v="Toolbar.Export" e="Toolbar.Export"/></button>
  <button command="Separate"><title v="-" e="-"/></button>
  <button command="Freeze"><title v="Toolbar.Freeze" e="Toolbar.Freeze"/></button>
</toolbar>
```

- `Toolbar.{Command}` → framework tự dịch
- `command="-"` hoặc `command="Separate"` → dấu phân cách
- Có thể thêm button tùy chỉnh: `ImportData`, `Download`, ...

### Toolbar có tính năng Import

Khi danh mục hỗ trợ import Excel, thêm 2 nút custom command. Xử lý click qua `ExecuteCommand` pattern trong `<script>`:

```xml
<toolbar>
  <!-- ... các nút chuẩn ... -->
  <button command="-"><title v="-" e="-"/></button>
  <!-- Custom command — tên command = CSS class cho icon -->
  <button command="ImportData">
    <title v="Lấy dữ liệu từ tệp..." e="Import Data from File..."/>
  </button>
  <button command="Download">
    <title v="Tải tệp mẫu..." e="Download Template File..."/>
  </button>
</toolbar>
```

> `command="ImportData"` / `command="Download"` là custom — framework không tự xử lý, mà gọi `ExecuteCommand` handler trong JS. Cần kèm CSS, script, response, DOCTYPE. Xem chi tiết tại `SPEC_import.md` — Phần 1.

## SQL View (vdm...) để Grid lấy tên hiển thị

Khi Grid cần hiển thị tên (ví dụ tên khách, tên vật tư) từ bảng khác:

```sql
-- Tạo view vdmgia2
CREATE VIEW vdmgia2 AS
SELECT a.*,
  b.ten_vt, b.ten_vt2,   -- từ bảng dmvt
  c.ten_kh, c.ten_kh2,   -- từ bảng dmkh
  d.ten_kho, d.ten_kho2  -- từ bảng dmkho
FROM dmgia2 a
LEFT JOIN dmvt b ON a.ma_vt = b.ma_vt
LEFT JOIN dmkh c ON a.ma_kh = c.ma_kh
LEFT JOIN dmkho d ON a.ma_kho = d.ma_kho
```

- Grid khai báo `table="vdmgia2"` thay vì `table="dmgia2"`
- Tên view = `v` + tên bảng gốc

---

## ✅ Checklist trước khi xuất file Grid

- [ ] `<title>` có nội dung song ngữ (`v` và `e`)
- [ ] **`<subTitle>` có nội dung** — danh mục thường: `"Cập nhật {tên}: thêm, sửa, xóa..."` (KHÔNG được để trống hoặc bỏ sót)
- [ ] **Width cột phù hợp loại dữ liệu**: mã `100-120`, tên `250-350`, tiền/hạn mức `110-130`, số lượng `80-100`, giá `90-110`, ngày `90-110`, ĐVT `50-70`, status `60-80`
- [ ] Cột tiền/giá/số lượng **KHÔNG** để width bằng cột mã/tên
- [ ] Field PK có `isPrimaryKey="true"`
- [ ] Field có `allowSorting="true"` và `allowFilter="true"` khi cần
- [ ] `<views>` liệt kê đúng thứ tự cột hiển thị
- [ ] `<commands>` có đủ Loading + Closing
