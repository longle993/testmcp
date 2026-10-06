# [SPEC_lookup-multiselect] — Field Lookup chọn nhiều (Multi-select)

> Dùng khi: field cần cho phép user **chọn nhiều giá trị** từ popup tra cứu.
> Áp dụng trong: Dir, Filter, Grid Detail.

---

## So sánh AutoComplete vs Lookup (chọn nhiều)

### Quy tắc nhận biết từ prompt

| Prompt ghi | Style sử dụng |
|---|---|
| "lookup **chọn nhiều** từ danh mục vật tư" | `style="Lookup"` |
| "tra cứu từ danh mục vật tư" (không ghi "chọn nhiều") | `style="AutoComplete"` |
| "chọn từ danh mục vật tư" (không ghi "chọn nhiều") | `style="AutoComplete"` |

> **Từ khóa quyết định:** có cụm **"chọn nhiều"** → Lookup. Không có → AutoComplete.

### So sánh khai báo

| Đặc điểm | `style="AutoComplete"` | `style="Lookup"` |
|---|---|---|
| Chọn | **1 giá trị** | **Nhiều giá trị** |
| Có `reference` (field tên) | ✅ Có — cần field tên đi kèm | ❌ Không có |
| Có `information` | ✅ Có | ❌ Không có |
| Có `clientScript` onChange | ✅ Thường có | ❌ Không cần |
| Field tên readOnly kèm theo | ✅ Cần khai báo | ❌ Không cần |
| Giá trị lưu DB | 1 mã đơn | Nhiều mã, phân tách bởi dấu `,` |

---

## Cấu trúc khai báo

### Lookup chọn nhiều (chuẩn)

```xml
<field name="tk_no" dataFormatString="@upperCaseFormat">
  <header v="Tài khoản nợ" e="Debit Account"></header>
  <items style="Lookup" controller="Account" key="status = '1'" check="1 = 1" />
</field>
```

### Đặc điểm khai báo

- `style="Lookup"` → thay vì `style="AutoComplete"`
- **Không có** `reference="..."` → không cần field tên đi kèm
- **Không có** `information="..."` → không cần thông tin hiển thị lại
- **Không có** `<clientScript>` onChange → không trigger response
- **Không cần** field tên readOnly external phía sau
- Vẫn dùng `controller`, `key`, `check` giống AutoComplete

---

## So sánh cụ thể: cùng 1 controller Account

### AutoComplete (chọn 1)

```xml
<field name="tk_no">
  <header v="Tài khoản nợ" e="Debit Account"></header>
  <items style="AutoComplete" controller="Account" reference="ten_tk%l" key="status = '1'" check="1 = 1" information="tk$dmtk.ten_tk%l" new="Default"/>
  <clientScript><![CDATA[onchange="onChange$MyController$Account(this);"]]></clientScript>
</field>
<field name="ten_tk%l" readOnly="true" external="true" defaultValue="''">
  <header v="" e=""></header>
</field>
```
→ 2 field (mã + tên), có reference, information, clientScript.

### Lookup chọn nhiều

```xml
<field name="tk_no" dataFormatString="@upperCaseFormat">
  <header v="Tài khoản nợ" e="Debit Account"></header>
  <items style="Lookup" controller="Account" key="status = '1'" check="1 = 1" />
</field>
```
→ 1 field duy nhất, không kèm tên.

---

## Ví dụ thêm

### Chọn nhiều khách hàng

```xml
<field name="ma_kh">
  <header v="Khách hàng" e="Customer"></header>
  <items style="Lookup" controller="Customer" key="status = '1'" check="1 = 1" />
</field>
```

### Chọn nhiều kho

```xml
<field name="ma_kho">
  <header v="Kho" e="Site"></header>
  <items style="Lookup" controller="Site" key="status = '1'" check="1 = 1" />
</field>
```

### Chọn nhiều bộ phận

```xml
<field name="ma_bp">
  <header v="Bộ phận" e="Department"></header>
  <items style="Lookup" controller="Department" key="status = '1'" check="1 = 1" />
</field>
```

### Chọn nhiều nhóm khách hàng (có lọc loai_nh)

```xml
<field name="nh_kh1">
  <header v="Nhóm KH 1" e="Customer Group 1"></header>
  <items style="Lookup" controller="CustomerGroup" key="loai_nh = 1 and status = '1'" check="1 = 1" />
</field>
```

---

## ✅ Checklist khi gặp field cần chọn nhiều

1. Xác định field cần multi-select
2. Đổi `style="AutoComplete"` → `style="Lookup"`
3. **Xóa** `reference="..."` và `information="..."`
4. **Xóa** `<clientScript>` onChange (nếu có)
5. **Xóa** field tên readOnly external đi kèm
6. Giữ nguyên `controller`, `key`, `check`
7. File Lookup XML (trong Controllers/Lookup/) **không thay đổi** — dùng chung

---

## Lưu ý SQL

Khi field Lookup chọn nhiều được dùng trong Filter, giá trị truyền vào SQL sẽ là chuỗi nhiều mã phân tách bởi dấu `,`. Xử lý trong SQL bằng cách split hoặc dùng `LIKE`/`CHARINDEX`:

```sql
where dbo.ff_inlist(ma_vt, @ma_vt) = 1
```
