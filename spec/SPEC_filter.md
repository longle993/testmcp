# [SPEC_filter] — File XML Filter (Điều kiện lọc)

> Dùng khi: danh mục có màn hình lọc trước khi hiển thị Grid.
> Thư mục: Controllers/Filter/{TênController}.xml
> ASPX phải có FilterMode="true"

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE dir [
  <!ENTITY XMLWhenFilterLoading SYSTEM "..\Include\XML\WhenFilterLoading.xml">
  <!ENTITY XMLWhenFilterClosing SYSTEM "..\Include\XML\WhenFilterClosing.xml">
  <!ENTITY JavascriptReportFilter SYSTEM "..\Include\Javascript\ReportFilter.txt">
]>

<dir table="{bảng}" code="{khóa}" order="{sắp_xếp}"
     cache="true" xmlns="urn:schemas-fast-com:data-dir">
  <title v="Lọc chi tiết..." e="...Filter"></title>
  <fields> ... </fields>
  <views> ... </views>
  <commands> ... </commands>
  <script> ... </script>
  <response> ... </response>
</dir>
```

### Thuộc tính đặc biệt
- `cache="true"` → giữ giá trị Filter khi quay lại

## `<fields>` — Các field lọc

Giống Dir nhưng mỗi field là **điều kiện lọc**, không phải input insert/update.

Field lọc AutoComplete:
```xml
<field name="loai_gia" allowNulls="false">
  <header v="Loại giá bán" e="Sales Pricing Type"></header>
  <items style="AutoComplete" controller="SalesPriceType" reference="ten_gb%l"  key="status = '1'" check="1 = 1"/>  
</field>
<field name="ten_gb%l" readOnly="true" external="true">
  <header v="" e=""></header>
</field>
```

Field ẩn (chứa giá trị trung gian từ response):

Field lọc Lookup chọn nhiều:
```xml
<!-- Prompt ghi "lookup chọn nhiều từ ..." → dùng style="Lookup" -->
<field name="tk_no" dataFormatString="@upperCaseFormat">
  <header v="Tài khoản nợ" e="Debit Account"></header>
  <items style="Lookup" controller="Account" key="status = '1'" check="1 = 1" />
</field>
```
> Không có `reference`, `information`, `clientScript`. Không cần field tên đi kèm.
> Giá trị truyền vào SQL là chuỗi nhiều mã phân tách bởi dấu `,`.
> Chi tiết: xem `SPEC_lookup-multiselect.md`

Field ẩn (chứa giá trị trung gian từ response):
```xml
<field name="kieu_gb" readOnly="true" external="true" hidden="true">
  <header v="" e=""></header>
</field>
```

Field ngày mặc định hôm nay:
```xml
<field name="ngay_ban" type="DateTime" dataFormatString="@datetimeFormat"
  allowNulls="false" aliasName="effectiveDate" defaultValue="new Date()">
  <header v="Hạn hiệu lực từ" e="Effective from"></header>
</field>
```

## `<commands>` — Events

### Showing — Khởi tạo giá trị mặc định

```xml
<command event="Showing">
  <text><![CDATA[
declare @message nvarchar(4000)
-- Lấy giá trị mặc định
select @salesPriceType = '01', @salesPriceTypeName = ten_loai
from dmloaigia2 where loai_gia = '01'
-- Trả về JS để gán giá trị
select 'this._salesPriceType=''' + @salesPriceType + ''';set$Form$DefaultValue(this);' as message
return
]]></text>
</command>
```

### Loading & Closing — dùng ENTITY chuẩn

```xml
&XMLWhenFilterLoading;
&XMLWhenFilterClosing;
```

### Inserting — Khi nhấn "Xem" / áp dụng filter

```xml
<command event="Inserting">
  <text><![CDATA[
select '' as field, '' as message, 'remove$GridReport$Filter(this.grid);' as script
return
]]></text>
</command>
```

## `<script>` — JavaScript xử lý Filter

### Template chuẩn

```javascript
// ENTITY chuẩn
&JavascriptReportFilter;

// Khởi tạo
function active$VoucherFilter$(sender) {
  sender.add_onResponseComplete(on$Filter$ResponseComplete);
}
function close$VoucherFilter$(sender) {
  try {sender.remove_onResponseComplete(on$Filter$ResponseComplete);} catch (ex) {}
}

// Xử lý response
function on$Filter$ResponseComplete(sender, e) {
  var f = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'Checking':
      // Xây dựng externalKey để truyền điều kiện lọc sang Grid
      var g = f.grid, k = new Array();
      Array.add(k, {Name: 'loai_gia', Opr: '=', Value: f.getItemValue('loai_gia'),
                     Type: 'String', Ignore: false});
      Array.add(k, {Name: 'ngay_ban', Opr: '>=', Value: f.getItemValue('ngay_ban'),
                     Type: 'DateTime', Ignore: false});
      g.set_externalKey(k);
      // SubTitle
      g._alterTitle = [null, [['%s1', code, true], ['%s2', name, true]]];
      // Ẩn/hiện cột trên Grid
      g._hiddenFields = [
        {Fields: 'ma_kho', Value: !(t.indexOf('S') > -1)},
        {Fields: 'ma_kh, ten_kh%l', Value: !(t.indexOf('C') > -1)}
      ];
      break;
    case 'SalesPriceType':
      // Nhận kết quả response, gán vào field
      f.setItemControlBehavior('kieu_gb', result[0].Value, null, true);
      break;
  }
}
```

### ExternalKey — Truyền điều kiện lọc sang Grid

```javascript
// Cấu trúc mỗi phần tử
{Name: 'tên_field', Opr: '=|like|>=|<=', Value: giá_trị, Type: 'String|DateTime|Decimal|Numeric', Ignore: false}
```

**`Type` theo kiểu dữ liệu:**

| Kiểu field | Type trong ExternalKey | Ghi chú |
|---|---|---|
| Mã, text | `'String'` | Phổ biến nhất |
| Ngày | `'DateTime'` | |
| Số thực / tiền | `'Decimal'` | |
| Số nguyên (kỳ, năm...) | `'Numeric'` | ⚠️ Dễ nhầm sang String |

> **Lưu ý kỳ/năm:** field `ky` và `nam` là số nguyên → dùng `Type: 'Numeric'`, **không** phải `'String'`.
> Lấy giá trị bằng `f.getItem('ky').value` (`.value` trực tiếp), không dùng `f.getItemValue('ky')`.
> Xem chi tiết tại `SPEC_period-year.md`.

```javascript
// Ví dụ lọc LIKE (cho mã có thể lọc theo prefix)
function addExternalKey(f, k, c) {
  if (f.getItem(c).disabled) return;
  var v = $func.trim(f.getItemValue(c));
  if (v != '') Array.add(k, {Name: c, Opr: 'like', Value: v + '%', Type: 'String', Ignore: false});
}

// Ví dụ kỳ/năm (Numeric)
Array.add(k, {Name: 'ky',  Opr: '=', Value: f.getItem('ky').value,  Type: 'Numeric', Ignore: false});
Array.add(k, {Name: 'nam', Opr: '=', Value: f.getItem('nam').value, Type: 'Numeric', Ignore: false});
```

## `<response>` — Xử lý request khi đổi field trên Filter

```xml
<response>
  <action id="SalesPriceType">
    <text><![CDATA[
if exists(select 1 from dmloaigia2 where loai_gia = @loai_gia) begin
  select rtrim(xtype) as xtype from dmloaigia2 where loai_gia = @loai_gia
  return
end
]]></text>
  </action>
</response>
```
