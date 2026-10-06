# [SPEC_dir] — File XML Dir (Form nhập liệu)

> Dùng khi: tạo form thêm / sửa / xóa dữ liệu.
> File gốc: `Controllers/Dir/{Controller}.xml`
> Khi share/download: `dir_{Controller}.xml`

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE dir [
  <!ENTITY ScriptIrregular SYSTEM "..\Include\Javascript\Irregular.txt">
  <!-- Nếu có Suggestion (gợi ý mã tự động) thì thêm: -->
  <!ENTITY XMLSuggestion SYSTEM "..\Include\XML\Suggestion.xml">
  <!ENTITY ScriptSuggestion SYSTEM "..\Include\Javascript\Suggestion.txt">
]>

<dir table="{tên_bảng}" code="{cột_mã}" order="{cột_sắp_xếp}" xmlns="urn:schemas-fast-com:data-dir">
  <title v="tên tiếng Việt" e="English Name"></title>
  <fields> ... </fields>
  <views> ... </views>
  <commands> ... </commands>
  <script> ... </script>
  <response> ... </response>  <!-- nếu có request từ JS -->
  <css> ... </css>            <!-- nếu cần style tùy chỉnh -->
</dir>
```

### Thuộc tính `<dir>`

| Thuộc tính | Bắt buộc | Mô tả |
|---|---|---|
| `table` | ✅ | Tên bảng DB |
| `code` | ✅ | Cột mã (các PK của bảng, nhiều PK thì để dang PK1, PK2, PK3) |
| `order` | ✅ | Cột sắp xếp mặc định |
| `xmlns` | ✅ | Cố định `urn:schemas-fast-com:data-dir` |

---

## `<fields>` — Khai báo field trên form

### Thuộc tính field

| Thuộc tính | Mô tả |
|---|---|
| `name` | Tên cột DB |
| `isPrimaryKey` | Khóa chính (chỉ field PK) |
| `allowNulls="false"` | Bắt buộc nhập (hiện dấu * đỏ) |
| `dataFormatString` | Có 2 dạng: (1) tên format đặc biệt: `@upperCaseFormat`, `@datetimeFormat`, `@foreignCurrencyAmountInputFormat`, `@baseCurrencyAmountInputFormat`; (2) **danh sách giá trị hợp lệ** cách nhau bằng dấu phẩy: `0, 1` (flag 2 giá trị), `1, 2, 3` (enum 3 giá trị), v.v. — framework tự validate, **không cần kiểm tra trong SQL** |
| `type` | `DateTime`, `Decimal`, `Boolean`, `Int16` |
| `readOnly` | Chỉ đọc |
| `external` | Trường giả (không có trong DB, dùng cho tên hiển thị) |
| `hidden` | Ẩn trên form |
| `defaultValue` | Giá trị mặc định JS — `"''"`, `"0"`, `"new Date()"`, `"cast(0 as bit)"` |
| `clientDefault` | `"Default"` = lấy giá trị từ dòng đang chọn / `"1"` cho status |
| `align` | `"left"`, `"right"`, `"center"` |
| `inactivate` | Bỏ qua TabStop khi nhấn Tab (đặt cho `status` cuối form) |
| `rows` | Số dòng cho textarea — `rows="2"` |
| `categoryIndex` | Số tab chứa field (1, 2, 3...) — **chỉ khi dùng tab** |

### Thẻ con `<header>`, `<footer>`, `<items>`, `<clientScript>`

```xml
<field name="ma_xxx" ...>
  <header v="Nhãn VN" e="Label EN"></header>
  <footer v="Ghi chú VN" e="Note EN"></footer>      <!-- tùy chọn -->
  <items style="Mask"/>                              <!-- tùy chọn -->
  <clientScript><![CDATA[onchange="..."]]></clientScript>  <!-- tùy chọn -->
</field>
```

### Kiểu control `<items style="...">`

| style | Dùng cho |
|---|---|
| `Mask` | Field mã, flag 0/1, enum nhiều giá trị cố định, các text có format cố định |
| `AutoComplete` | Tra cứu từ Lookup — chọn **1 giá trị**, kèm field tên hiển thị |
| `Numeric` | Trường số |
| `Lookup` | Tra cứu từ Lookup — **chọn nhiều giá trị**, không kèm field tên |

> **Phân biệt trong prompt:**
> - Prompt ghi "lookup chọn nhiều từ ..." → dùng `style="Lookup"` (pattern #10)
> - Prompt không ghi "chọn nhiều" → dùng `style="AutoComplete"` (pattern #4)
>
> Chi tiết: xem `SPEC_lookup-multiselect.md`

---

## 🔹 Pattern field chuẩn (10 mẫu thường dùng)

### 1. Field mã (Primary Key)
```xml
<field name="ma_xxx" isPrimaryKey="true" dataFormatString="@upperCaseFormat" allowNulls="false">
  <header v="Mã xxx" e="xxx ID"></header>
  <items style="Mask"/>
</field>
```

### 2. Field tên bắt buộc
```xml
<field name="ten_xxx" allowNulls="false">
  <header v="Tên xxx" e="xxx Name"></header>
</field>
```

### 3. Field tên khác (không bắt buộc)
```xml
<field name="ten_xxx2">
  <header v="Tên khác" e="Other Name"></header>
</field>
```

### 4. Field AutoComplete (FK đến bảng khác) — kèm tên hiển thị

> ⚠️ **Quy tắc quan trọng**: Framework AutoComplete **tự động** lấy và hiển thị tên qua `reference` + `information`. **KHÔNG cần** viết `clientScript` onChange, `<response>`, hay JS request chỉ để gán tên.
>
> Chỉ thêm `clientScript` onChange + `<response>` khi prompt yêu cầu **lấy thông tin phụ khác ngoài tên** (ví dụ: nhập mã khách → lấy địa chỉ, MST, ĐT gán vào các field khác).

#### 4a. AutoComplete đơn giản — chỉ hiển thị tên (MẶC ĐỊNH)

```xml
<field name="ma_bp">
  <header v="Mã bộ phận" e="Department"></header>
  <items style="AutoComplete" controller="Department" reference="ten_bp%l" key="status = '1'" check="1 = 1" information="ma_bp$dmbp.ten_bp%l" new="Default"/>
</field>
<field name="ten_bp%l" readOnly="true" external="true" defaultValue="''">
  <header v="" e=""></header>
</field>
```

**Không có** `clientScript`, không có `<response>`, không có JS onChange. Framework tự xử lý.

#### 4b. AutoComplete có lấy thông tin phụ — cần onChange + response

Chỉ dùng khi prompt mô tả rõ: *"nhập mã khách → lấy địa chỉ, MST, ĐT gán vào field tương ứng"*.

```xml
<field name="ma_kh">
  <header v="Mã khách" e="Customer"></header>
  <items style="AutoComplete" controller="Customer" reference="ten_kh%l" key="status = '1'" check="1 = 1" information="ma_kh$dmkh.ten_kh%l" new="Default"/>
  <clientScript><![CDATA[onchange="onChange$MyController$Customer(this);"]]></clientScript>
</field>
<field name="ten_kh%l" readOnly="true" external="true" defaultValue="''">
  <header v="" e=""></header>
</field>
<!-- Các field phụ được gán từ response -->
<field name="dia_chi"><header v="Địa chỉ" e="Address"></header></field>
<field name="ma_so_thue"><header v="Mã số thuế" e="Tax Code"></header></field>
```

Kèm JS + response tương ứng (xem phần `<script>` và `<response>` bên dưới).

### 5. Field ngày
```xml
<field name="ngay_xx" type="DateTime" dataFormatString="@datetimeFormat">
  <header v="Ngày xx" e="Date"></header>
</field>
```

### 6. Field số tiền, số lượng
```xml
<!-- Ngoại tệ -->
<field name="tien_nt" type="Decimal" dataFormatString="@foreignCurrencyAmountInputFormat" defaultValue="0">
  <header v="Tiền ngoại tệ" e="FC Amount"></header>
  <items style="Numeric"/>
</field>
<!-- Bản tệ -->
<field name="tien" type="Decimal" dataFormatString="@baseCurrencyAmountInputFormat" defaultValue="0">
  <header v="Tiền hạch toán" e="Base Currency Amount"></header>
  <items style="Numeric"/>
</field>
<!-- Số lượng -->
<field name="so_luong" type="Decimal" dataFormatString="@quantityViewFormat" defaultValue="0">
  <header v="Số lượng" e="Quantity"></header>
  <items style="Numeric"/>
</field>
```

### 7. Field flag 0/1
```xml
<field name="phan_loai" dataFormatString="0, 1" clientDefault="0" align="right">
  <header v="Theo dõi số dư" e="Balance Tracking"></header>
  <footer v="1 - Có, 0 - Không" e="1 - Yes, 0 - No"></footer>
  <items style="Mask"/>
</field>
```

### 8. Field enum — nhiều giá trị cố định

> Dùng khi field chỉ nhận một tập giá trị nguyên cố định (không phải chỉ 0/1).
> `dataFormatString` liệt kê các giá trị hợp lệ — framework tự validate,
> **không cần `if @field not in (...)` trong SQL**.
> Dữ liệu lưu DB có thể là chuỗi hoặc số nguyên, đều được.

```xml
<field name="nhom" type="Int16" dataFormatString="1, 2, 3" allowNulls="false" defaultValue="1" align="right">
  <header v="Nhóm" e="Group"/>
  <footer v="1 - Cơ bản, 2 - Nâng câo, 3 - Cao cấp" e="1 - Cơ bản, 2 - Nâng câo, 3 - Cao cấp"/>
  <items style="Mask"/>
</field>
```

**So sánh flag vs enum:**

| | Flag 0/1 | Enum nhiều giá trị |
|---|---|---|
| `dataFormatString` | `"0, 1"` | `"1, 2, 3"` hoặc bất kỳ tập nào |
| `type` | *(bỏ qua)* | `Int16` nếu lưu số nguyên |
| `defaultValue` | `"0"` hoặc `"1"` | Giá trị đầu tiên trong tập |
| `clientDefault` | Dùng cho status | **Không dùng** |
| SQL validate | Không cần | **Không cần** |
| Layout view | `1110` | `1110` |

### 9. Field status (luôn đặt cuối, có `inactivate`)
```xml
<field name="status" dataFormatString="0, 1" clientDefault="1" align="right" inactivate="true">
  <header v="Trạng thái" e="Status"></header>
  <footer v="1 - Còn sử dụng, 0 - Không còn sử dụng" e="1 - Active, 0 - Inactive"></footer>
  <items style="Mask"/>
</field>
```

### 10. Field Lookup chọn nhiều (multi-select)

> Dùng khi prompt ghi **"lookup chọn nhiều từ ..."**.
> Khác AutoComplete: không có `reference`, `information`, `clientScript`, không cần field tên đi kèm.

```xml
<field name="tk_no" dataFormatString="@upperCaseFormat">
  <header v="Tài khoản nợ" e="Debit Account"></header>
  <items style="Lookup" controller="Account" key="status = '1'" check="1 = 1" />
</field>
```

**So sánh nhanh với AutoComplete (pattern #4):**

| | AutoComplete (chọn 1) | Lookup (chọn nhiều) |
|---|---|---|
| `reference` | ✅ Có | ❌ Không |
| `information` | ✅ Có | ❌ Không |
| `clientScript` onChange | ❌ Không (trừ khi lấy thông tin phụ — pattern 4b) | ❌ Không |
| Field tên readOnly kèm theo | ✅ Cần | ❌ Không cần |
| Số field khai báo | 2 (mã + tên) | 1 (chỉ mã) |

> Chi tiết và ví dụ thêm: xem `SPEC_lookup-multiselect.md`

---

## `<views>` — Layout form (KHÔNG TAB)

```xml
<views>
  <view id="Dir">
    <item value="120, 30, 70, 330"/>          <!-- Dòng 1: độ rộng các cột (4 cột) -->
    <item value="110: [ma_xxx].Label, [ma_xxx]"/>
    <item value="1100: [ten_xxx].Label, [ten_xxx]"/>
    <!-- ... -->
  </view>
</views>
```

### Dòng đầu — độ rộng cột (px)

Layout 4 cột chuẩn cho danh mục đơn giản:
```xml
<item value="120, 30, 70, 330"/>
<!-- col1=120: Label
     col2=30:  Cột phụ (cho field flag)
     col3=70:  Cột flag (input)
     col4=330: Cột input chính / cột tên -->
```

### Các dòng field — Cú pháp `"layout: phần_tử_1, phần_tử_2, ..."`

**Quy tắc layout (mỗi ký tự = 1 cột):**
- `1` = phần tử có mặt tại cột này (lấy phần tử kế tiếp ở vế phải)
- `0` = merge với cột trước nó (phần tử trước rộng ra)
- `-` = ô trống tại cột này

**Số ký tự ở vế trái = số cột đã khai báo ở dòng đầu.**
**Số chữ `1` ở vế trái = số phần tử ở vế phải.**

### Cheat sheet layout (cho 4 cột)

| Mã layout | Hiển thị | Dùng cho |
|---|---|---|
| `110:` | Label + Input (col 3-4 trống) | Field mã (`ma_xxx`) |
| `110-:` | Label + Input + ô trống cuối | Field ngắn (phone, mã thuế) |
| `1100:` | Label + Input (merge col 3-4 vào input) | Field text dài |
| `1101:` | Label + Input + Tên (col 3 merge vào col 2) | FK với tên hiển thị |
| `1110:` | Label + Input + Description | Field flag, enum, status |
| `1111:` | Label + Input + Tên + (cột 4) | Field FK có thêm cột phụ |

### Các phần tử ở vế phải

```
[field_name].Label       → nhãn (từ <header>)
[field_name]             → control input
[field_name].Description → ghi chú (từ <footer>)
```

---

## `<views>` — Layout form CÓ TAB

Khi danh mục có nhiều field (> 10-15), tách thành tab:

```xml
<views>
  <view id="Dir" height="232">
    <!-- Header (NGOÀI tab) — các field cơ bản, KHÔNG có categoryIndex -->
    <item value="120, 100, 330"/>
    <item value="11: [ma_xxx].Label, [ma_xxx]"/>
    <item value="110: [ten_xxx].Label, [ten_xxx]"/>
    <item value="110: [ten_xxx2].Label, [ten_xxx2]"/>

    <!-- Các field thuộc tab (CÓ categoryIndex trong <fields>) -->
    <item value="110-11: [ngay_vv].Label, [ngay_vv], [so_vv].Label, [so_vv]"/>
    <item value="110100: [ma_nt].Label, [ma_nt], [ten_nt%l]"/>
    <!-- ... -->

    <!-- Định nghĩa các tab -->
    <categories>
      <category index="1" columns="120, 30, 70, 110, 120, 100">
        <header v="Thông tin chính" e="General"/>
      </category>
      <category index="2" columns="120, 30, 70, 330">
        <header v="Khác" e="Other"/>
      </category>
    </categories>
  </view>
</views>
```

### Quy tắc tab

1. **Thuộc tính `height`** trên `<view>`: cố định chiều cao form khi có tab
2. **Field header** (mã, tên, tên khác): KHÔNG có `categoryIndex` → hiển thị **trên** thanh tab
3. **Field trong tab**: thêm `categoryIndex="1"`, `categoryIndex="2"`... trong `<fields>`
4. **Mỗi tab** có `columns` riêng (số cột có thể khác nhau)
5. **Số ký tự layout** của mỗi `<item>` phải khớp số cột của tab chứa field đó

### Ví dụ layout 6 cột (Job.xml, tab 1)

```xml
<category index="1" columns="120, 30, 70, 110, 120, 100">
```

| Mã layout | Hiển thị (6 cột) | Dùng cho |
|---|---|---|
| `110-11` | Label₁ + Input₁ (col1-3) + trống + Label₂ + Input₂ | 2 field trên 1 dòng |
| `111000` | Label + Input + Description (merge col3-6) | Field flag span full |
| `110100` | Label + Input (col2-3) + Tên (col4-6) | FK + tên hiển thị |
| `1101` | Label + Input + Tên | Layout 4 cột |

---

## `<commands>` — SQL Events (xem `SPEC_sql-events.md`)

> ⚠️ **BẮT BUỘC — KHÔNG ĐƯỢC BỎ SÓT**: Mọi danh mục (Dir) **luôn phải có event `Declare`** để khai báo biến thông báo lỗi (`@$exists`, `@$recordHasBeenChanged`).
> Các event `Inserting`, `Updating` sử dụng các biến này. Nếu thiếu `Declare`, SQL sẽ lỗi runtime do biến chưa khai báo.
>
> **Danh sách event BẮT BUỘC cho mọi danh mục:**
> 1. `Loading` — gọi `active$Form{Controller}(this);`
> 2. `Closing` — gọi `close$Form{Controller}(this);`
> 3. **`Declare`** — khai báo `@$exists` và `@$recordHasBeenChanged`
> 4. `Inserting` — validate + gán audit fields
> 5. `Updating` — validate khi sửa
> 6. `Updated` — cập nhật audit fields sau khi sửa
> 7. _(tùy chọn)_ `Deleting` — chặn xóa nếu đã phát sinh

### Template chuẩn cho danh mục có code hierarchical (Bank/Job pattern)

```xml
<commands>
  <command event="Loading">
    <text><![CDATA[
select 'active$Form{Controller}(this);' as message
return
]]></text>
  </command>

  <command event="Closing">
    <text><![CDATA[
select 'close$Form{Controller}(this);' as message
return
]]></text>
  </command>

  <command event="Declare">
    <text><![CDATA[
declare @$exists nvarchar(512), @$recordHasBeenChanged nvarchar(512)
select @$exists = case when @@language = 'v'
  then N'Mã {tên_VN} <span class="Highlight">%s</span> đã có hoặc lồng nhau trong danh mục {tên_VN}.'
  else N'The {name_EN} <span class="Highlight">%s</span> is invalid or already exists.' end
select @$recordHasBeenChanged = case when @@language = 'v'
  then N'Mã {tên_VN} <span class="Highlight">%s</span> đã được sửa hoặc xóa bởi người sử dụng khác.'
  else N'The {name_EN} <span class="Highlight">%s</span> has been modified or deleted by another user.' end
]]></text>
  </command>

  <command event="Inserting">
    <text><![CDATA[
if exists(select * from @@table where ((ma_xxx like rtrim(@ma_xxx) + '%') or rtrim(@ma_xxx) like rtrim(ma_xxx) + '%'))
  begin
    select 'ma_xxx' as field, replace(@$exists, '%s', rtrim(@ma_xxx)) as message
    return
  end
select @datetime0 = getdate(), @datetime2 = getdate(), @user_id0 = @@userID, @user_id2 = @@userID
]]></text>
  </command>

  <command event="Updating">
    <text><![CDATA[
if not exists(select * from @@table where ma_xxx = $ma_xxx.OldValue)
  begin
    select 'ma_xxx' as field, replace(@$recordHasBeenChanged, '%s', rtrim($ma_xxx.OldValue)) as message
    return
  end
if @ma_xxx <> $ma_xxx.OldValue
  begin
    if exists(select * from @@table where ((ma_xxx like rtrim(@ma_xxx) + '%') or rtrim(@ma_xxx) like rtrim(ma_xxx) + '%') and ma_xxx <> $ma_xxx.OldValue)
      begin
        select 'ma_xxx' as field, replace(@$exists, '%s', rtrim(@ma_xxx)) as message
        return
      end
  end
]]></text>
  </command>

  <command event="Updated">
    <text><![CDATA[
update @@table set datetime2 = getdate(), user_id2 = @@userID where ma_xxx = @ma_xxx
]]></text>
  </command>

  <!-- Tùy chọn: chặn xóa nếu đã phát sinh dữ liệu -->
  <command event="Deleting">
    <text><![CDATA[
if exists(select * from {bảng_giao_dịch} where ma_xxx = @ma_xxx) begin
  select N'{Tên} đã phát sinh, không thể xóa được.' as message
  return
end
]]></text>
  </command>
</commands>
```

> 💡 **Kiểm tra mã `like %`** ở `Inserting` không phải để chống trùng exact — mà để chống **mã lồng nhau** (vd: đã có "DV01" thì không cho nhập "DV011" hoặc "DV0"). Đây là pattern chuẩn của FBO cho mã phân cấp.

---

## `<script>` — JavaScript chuẩn

### Pattern thường dùng (Bank pattern)

```javascript
function active$Form{Controller}(f) {
  f.add_onResponseComplete(on$Form{Controller}$ResponseComplete);
}
function close$Form{Controller}(f) {
  try {f.remove_onResponseComplete(on$Form{Controller}$ResponseComplete)} catch (ex) {}
}
function on$Form{Controller}$ResponseComplete(sender, e) {
  var f = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'Checking':
      objectBehavior$Dir$Irregular.checkCode(f, 'ma_xxx');
      break;
    // CHỈ thêm case khi có request lấy thông tin PHỤ (ngoài tên) — pattern 4b
    case 'Customer':  // lấy địa chỉ, MST, ĐT — KHÔNG phải lấy tên
      f.setItemValues('dia_chi, ma_so_thue, dien_thoai',
                      [result[0].Value, result[1].Value, result[2].Value]);
      break;
    default:
      break;
  }
}
```

> ⚠️ **Nếu AutoComplete chỉ hiển thị tên** (pattern 4a — mặc định), **KHÔNG thêm case trong switch** và **KHÔNG viết hàm onChange**. Chỉ cần `case 'Checking'` là đủ.

### Khi có TAB (Job pattern)

```javascript
function onTabChanged(sender, e) {
  // Khi user click sang tab khác, focus về field đầu tab đó
  sender.parentForm.focusWhenTabChanged(['ngay_vv', 'nh_vv1']);
  // Tham số là array tên field đầu tiên của mỗi tab
}
function active$Form{Controller}(f) {
  f.add_onResponseComplete(on$Form{Controller}$ResponseComplete);
  f._tabContainer.add_activeTabChanged(onTabChanged);
  f._tabContainer._loaded = true;
}
function close$Form{Controller}(f) {
  if (f._tabContainer) try {f._tabContainer.remove_activeTabChanged(onTabChanged);} catch (ex) {}
  try {f.remove_onResponseComplete(on$Form{Controller}$ResponseComplete)} catch (ex) {}
}
```

### Khi có Suggestion (gợi ý mã tự động — Job pattern)

```javascript
function active$Form{Controller}(f) {
  objectBehavior$Dir$Code.create(f, 'ma_xxx', 'ma_xxx', '{tên_bảng}');
  f.add_onResponseComplete(on$Form{Controller}$ResponseComplete);
  // ... thêm tab nếu có
}
function close$Form{Controller}(f) {
  objectBehavior$Dir$Code.dispose(f);
  // ...
}
function on$Form{Controller}$ResponseComplete(sender, e) {
  var f = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'Suggestion':
      objectBehavior$Dir$Code.suggestion(f, result[0].Value);
      break;
    case 'Checking':
      objectBehavior$Dir$Code.checkCode(f);  // dùng Code thay vì Irregular
      break;
  }
}
```

### onChange field để lấy thông tin phụ (KHÔNG dùng chỉ để lấy tên)

> ⚠️ **CHỈ viết onChange khi cần lấy thông tin phụ** ngoài tên (địa chỉ, MST, ĐT...).
> Framework AutoComplete **đã tự gán tên** qua `reference` + `information`.
> **KHÔNG viết** onChange + request + response chỉ để lấy tên hiển thị cho field `ten_xxx%l`.

```javascript
// VÍ DỤ: nhập mã khách → lấy địa chỉ, MST, ĐT (thông tin PHỤ ngoài tên)
function onChange${Controller}$Customer(o) {
  var c = 'ma_kh', f = o.parentForm;
  if ($func.trim(f.getItemValue(c)) != '')
    f.request('Customer', 'Customer', [c], o);
}
```

### ENTITY script đặt cuối CDATA

```xml
<script>
  <text>
    <![CDATA[
// ... JS code ...
]]>
    &ScriptIrregular;       <!-- cho check code thông thường -->
    <!-- HOẶC -->
    &ScriptSuggestion;      <!-- cho check code có gợi ý -->
  </text>
</script>
```

---

## `<response>` — Xử lý request từ JavaScript

> ⚠️ **Chỉ cần `<response>` khi JS có request lấy thông tin phụ** (pattern 4b) hoặc khi có Suggestion.
> **KHÔNG tạo `<response>`** chỉ để trả về tên cho AutoComplete — framework đã tự xử lý.

```xml
<!-- VÍ DỤ: trả về địa chỉ, MST, ĐT (thông tin PHỤ ngoài tên) -->
<response>
  <action id="Customer">
    <text><![CDATA[
if exists(select 1 from dmkh where ma_kh = @ma_kh) begin
  select rtrim(dia_chi) as dia_chi,
         rtrim(ma_so_thue) as ma_so_thue,
         rtrim(dien_thoai) as dien_thoai
  from dmkh where ma_kh = @ma_kh
  return
end
]]></text>
  </action>
</response>
```

### Nếu có Suggestion (gợi ý mã)
```xml
<response>
  &XMLSuggestion;  <!-- ENTITY chuẩn, không cần viết thêm -->
</response>
```

---

## `<css>` — Style tùy chỉnh (tùy chọn)

```xml
<css>
  <text><![CDATA[
.LabelDescription{color:#73A6DE;}
]]></text>
</css>
```

Sử dụng inline trong `<header>`:
```xml
<header v="Số gửi bản sao &lt;span class=&quot;LabelDescription&quot;&gt;(Fax)&lt;/span&gt;" e="Fax Number"></header>
```

---

## ✅ Checklist trước khi xuất file

- [ ] `table`, `code`, `order` trên `<dir>` đúng tên DB
- [ ] Field PK có đủ: `isPrimaryKey="true"`, `allowNulls="false"`, `dataFormatString="@upperCaseFormat"`, `<items style="Mask"/>`
- [ ] Field FK có cả 2: field input (AutoComplete) + field tên hiển thị (`external="true"`, `readOnly="true"`)
- [ ] Field FK AutoComplete **KHÔNG có** `clientScript` onChange / `<response>` / JS request chỉ để lấy tên (framework tự xử lý qua `reference` + `information`). Chỉ thêm khi lấy **thông tin phụ khác** ngoài tên (pattern 4b)
- [ ] Field flag 0/1 có `dataFormatString="0, 1"`, `<items style="Mask"/>`, `<footer>`
- [ ] Field enum nhiều giá trị dùng `dataFormatString="v1, v2, v3"` + `type="Int16"` nếu int — **KHÔNG thêm SQL validate**
- [ ] Field `status` đặt cuối, có `inactivate="true"`, `clientDefault="1"`
- [ ] Nếu có tab: mọi field thuộc tab phải có `categoryIndex`; field header KHÔNG có
- [ ] Số ký tự layout `<item>` khớp số cột của tab/view chứa nó
- [ ] `<commands>` đủ: Loading, Closing, **⚠️ Declare (BẮT BUỘC — khai báo @$exists, @$recordHasBeenChanged)**, Inserting, Updating, Updated (+ Deleting nếu cần)
- [ ] `active$Form{Controller}` và `close$Form{Controller}` khớp với tên trong `Loading`/`Closing`
- [ ] `&ScriptIrregular;` hoặc `&ScriptSuggestion;` đặt cuối thẻ `<script>`, ngoài CDATA
