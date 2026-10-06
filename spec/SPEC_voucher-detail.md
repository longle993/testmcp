# [SPEC_voucher-detail] — Grid Detail cho Chứng từ

> Dùng khi: tạo Grid nhập liệu chi tiết (nhiều dòng) trong form chứng từ.
> Grid Detail chứng từ khác danh mục ở: tách kỳ, khóa stt_rec/stt_rec0, Gather, validRow.
> Đọc `SPEC_voucher-overview.md` trước.

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE grid [
  <!ENTITY % GridInitialize SYSTEM "..\Include\Grid.ent">
  %GridInitialize;

  <!ENTITY XMLGetUOMConversion SYSTEM "..\Include\XML\GetUOMConversion.xml">
  <!ENTITY ScriptCheckGridAction SYSTEM "..\Include\Javascript\CheckGridAction.txt">
  <!ENTITY ScriptEmptyExternalField SYSTEM "..\Include\Javascript\EmptyExternalField.txt">
  <!ENTITY ScriptInsertRetrieveItems SYSTEM "..\Include\Javascript\InsertRetrieveItems.txt">

  <!ENTITY TransferID "{Controller}">
  <!ENTITY Code "{ma_ct}">

  <!ENTITY VisibleFieldController "{Controller}Detail">
  <!ENTITY % VoucherVisibleField SYSTEM "..\Include\VoucherVisibleField.ent">
  %VoucherVisibleField;
]>
```

### Thẻ `<grid>` — Khác grid browser

```xml
<grid table="{bảng_detail_cấu_trúc}" code="stt_rec" order="stt_rec, line_nbr" type="Detail" freezeColumns="3" id="{ma_ct}" uniKey="true" xmlns="urn:schemas-fast-com:data-grid">
  <title v="" e=""/>
  <subTitle v="" e=""/>
  <partition .../>  <!-- partition giống grid browser -->
```

**Điểm khác grid browser:**
- `type="Detail"` (không phải `Voucher`)
- `order="stt_rec, line_nbr"` (sắp xếp theo dòng)
- `freezeColumns="3"` (cố định 3 cột đầu)
- `table` = bảng **detail** (không phải master)

### Partition — theo loại chứng từ

**Phân kỳ:**
```xml
<partition table="c74$000000" prime="d74$" inquiry="i74$" field="ngay_ct" expression="convert(char(6), {0}, 112)" increase="dateadd(month, 1, {0})" default="000000"/>
```
> `prime` trỏ đến prefix bảng **detail** (`d74$`), không phải master.

**Không phân kỳ:**
```xml
<partition table="ctsx" prime="ctsx" inquiry="isx" field="ngay_ct" expression="''" increase="{0}" default=""/>
```

---

## Fields

### Trường nghiệp vụ (đầu tiên, hiện trên grid)

```xml
<field name="ma_vt" width="100" allowNulls="false" aliasName="a">
  <header v="Mã hàng" e="Item Code"/>
  <items style="AutoComplete" controller="Item" reference="ten_vt%l" key="status = '1'" check="1 = 1" information="ma_vt$dmvt.ten_vt%l" new="Default"/>
  <handle source="dmvt.ma_vt" foreward="true"/>
  <clientScript><![CDATA[onchange="onChange$GridVoucherDetail$Item(this);"]]></clientScript>
</field>
<field name="ten_vt%l" readOnly="true" external="true" defaultValue="''" inactivate="true" width="300" aliasName="b">
  <header v="Tên mặt hàng" e="Item Description"/>
</field>
```

### Trường ĐVT (nếu có nhiều đơn vị)

```xml
<field name="dvt" width="50" allowNulls="false" aliasName="a">
  <header v="Đvt" e="UOM"/>
  <items style="AutoComplete" controller="UOMItem" reference="ten_dvt%l" key="(ma_vt = '{$%c$%r.[ma_vt]}' or ma_vt = '*')" information="dvt$vdmvtqddvt.ten_dvt%l" normal="true"/>
  <handle key="[nhieu_dvt]"/>
  <clientScript><![CDATA[onchange="onChange$GridVoucherDetail$UOM(this);"]]></clientScript>
</field>
<field name="he_so" type="Decimal" width="0" inactivate="true" hidden="true" dataFormatString="&CoefficientInputFormat;" clientDefault="0">
  <header v="" e=""/>
  <items style="Numeric"/>
</field>
```

### Trường tiền (nếu chứng từ có tiền)

```xml
<field name="so_luong" type="Decimal" dataFormatString="@quantityInputFormat" clientDefault="0" width="80">
  <header v="Số lượng" e="Quantity"/>
  <items style="Numeric"/>
</field>
<field name="gia_nt" type="Decimal" dataFormatString="@foreignCurrencyPriceInputFormat" clientDefault="0" width="90">
  <header v="Đơn giá" e="Unit Price"/>
  <items style="Numeric"/>
</field>
<field name="tien_nt" type="Decimal" dataFormatString="@foreignCurrencyAmountInputFormat" clientDefault="0" width="100">
  <header v="Thành tiền" e="Amount"/>
  <items style="Numeric"/>
</field>
```

### Trường ẩn (hidden fields) — Bắt buộc

```xml
<!-- Các trường boolean ẩn cho handle -->
<field name="nhieu_dvt" type="Boolean" width="0" external="true" hidden="true" aliasName="b">
  <header v="" e=""/>
  <handle key="[nhieu_dvt = 1]" field="ma_vt"/>
</field>

<!-- Khóa chính -->
<field name="stt_rec" isPrimaryKey="true" readOnly="true" hidden="true">
  <header v="" e=""/>
</field>
<field name="stt_rec0" isPrimaryKey="true" width="0" hidden="true">
  <header v="" e=""/>
</field>
<field name="line_nbr" type="Int32" width="0" align="right" hidden="true">
  <header v="" e=""/>
</field>
```

---

## Views — Liệt kê tất cả field

```xml
<views>
  <view id="Grid">
    <!-- Field hiển thị theo thứ tự cột -->
    <field name="ma_vt"/>
    <field name="ten_vt%l"/>
    <field name="dvt"/>
    ...
    <!-- Field ẩn cuối cùng -->
    <field name="stt_rec"/>
    <field name="stt_rec0"/>
    <field name="line_nbr"/>
  </view>
</views>
```

---

## Commands — Cố định

```xml
<commands>
  <command event="Loading">
    <text>
      <![CDATA[select 'this._voucherCode=@@id';load$GridVoucherDetail$(this);' as message
return]]>
    </text>
  </command>
  <command event="Closing">
    <text>
      <![CDATA[select 'dispose$GridVoucherDetail$(this);' as message
return]]>
    </text>
  </command>
</commands>
```

---

## Script — Pattern chuẩn

### Khởi tạo / hủy

```javascript
function load$GridVoucherDetail$(g) {
  g.add_itemValueChanged(onChange$GridVoucherDetail$);
  g.add_onResponseComplete(on$GridVoucherDetail$ResponseComplete);
  g.add_commandEvent(on$GridVoucherDetail$ExecuteCommand);
  // Khai báo công thức tính
  g.$a = {
    tien_nt: '[tien_nt]:=[so_luong]*[gia_nt]',
    tien_tg: '[tien]:=[tien_nt]*[$ty_gia]',
    t_so_luong: ['t_so_luong', 'so_luong'],
    t_tien_nt: ['t_tien_nt', 'tien_nt'],
    t_tien: ['t_tien', 'tien'],	
    t_tt: '[t_tt]:=[t_tien]+[t_thue]',
    t_tt_nt: '[t_tt_nt]:=[t_tien_nt]+[t_thue_nt]'
  };
  // Biến truyền lên server khi request
  g.$h = [['voucherCode', 'String', g._voucherCode], 'ma_dvcs'];
}
function dispose$GridVoucherDetail$(g) {
  g.$a = null;
  try {g.remove_commandEvent(on$GridVoucherDetail$ExecuteCommand);} catch (ex) {}
  try {g.remove_itemValueChanged(onChange$GridVoucherDetail$);} catch (ex) {}
  try {g.remove_onResponseComplete(on$GridVoucherDetail$ResponseComplete);} catch (ex) {}
}
```

### Công thức tính (`g.$a`)

**Dạng 1: Tính field trong grid (expression)**
```javascript
// '[field_kết_quả]:=[field1]*[field2]'
gia: '[gia]:=[gia_nt]*[$ty_gia]',        // $ = lấy từ form master
tien_nt: '[tien_nt]:=[so_luong]*[gia_nt]',
```

**Dạng 2: Tính tổng từ chi tiết grid (aggregate)**
```javascript
// ['field_tổng_trên_form', 'field_trong_grid']
t_so_luong: ['t_so_luong', 'so_luong'],
// Có điều kiện: ['field_tổng', 'expression', 'condition']
t_tien_nt2: ['t_tien_nt2', '[tien_nt2] - [ck_nt]', '[km_yn] == 0'],
```

**Dạng 3: Tính tổng trên form (aggregate)**
```javascript
// ['field1_trên_form', ['field2_trên form'] + ['field3_trên form']]
t_tt: '[t_tt]:=[t_tien]+[t_thue]',
t_tt_nt: '[t_tt_nt]:=[t_tien_nt]+[t_thue_nt]'
```

### Khi field thay đổi

```javascript
function onChange$GridVoucherDetail$(sender, eventArgs) {
  var o = eventArgs.get_object(), g = o.grid, name = o.field.Name;
  switch (name) {
    case 'so_luong':      
      g.validExpression(o, [g.$a.tien_nt, g.$a.tien_sl], [g.$a.t_so_luong, g.$a.t_tien_nt, g.$a.t_tien], [g.$a.t_tt_nt, g.$a.t_tt], 'gia_nt2');
      break;
    // Thêm case cho các field tính toán khác
  }
}
```

### ExecuteCommand — Sự kiện chuẩn

```javascript
function on$GridVoucherDetail$ExecuteCommand(sender, e) {
  var action = e.type.Action, g = sender, f = g.get_element().parentForm;
  switch (action) {
    case 'AfterRemoveRow':
      on$GridVoucherDetail$RowChange(g, f);
      break;
    case 'AfterCloneRow':
      g.setItemFieldValue('stt_rec0', e.type.Value, '');
      on$GridVoucherDetail$RowChange(g, f);
      break;
    case 'Check':
      // validRowExpression cho các trường bắt buộc có điều kiện
      g.validRowExpression('ma_vi_tri', '([vi_tri_yn] == 0) || ([ma_vi_tri] != "")');
      break;
    case 'Copying':
      // Reset các trường reference khi copy
      set$Voucher$EmptyExternalField(g, 'dh_so, dh_ln, stt_rec_dh, stt_rec0dh');
      break;
  }
}
function on$GridVoucherDetail$RowChange(g, f) {
  g.executeAggregate([g.$a.t_so_luong]);
}
```

### ResponseComplete — Xử lý response từ server

```javascript
function on$GridVoucherDetail$ResponseComplete(sender, e) {
  var g = e.object, context = e.type.Context, result = e.type.Result, o = e.type.Object;
  switch (context) {
    case 'Item':
      g.setItemGridBehavior(o, [
        ['ma_kho', result[0].Value, '', true],
        ['dvt', result[1].Value, '', true],
        ['he_so', result[2].Value, null, null]
      ]);
      g.live(o, 'dvt');
      break;
    case 'UOM':
      g.setItemGridBehavior(o, [['he_so', result[0].Value, null, true]]);
      break;
  }
}
```

### Request handlers

```javascript
function onChange$GridVoucherDetail$Item(o) {
  o.grid.request(o, 'Item', 'Item', ['ma_vt'], o.grid.$h, true);
}
function onChange$GridVoucherDetail$UOM(o) {
  o.grid.request(o, 'UOM', 'UOM', ['ma_vt', 'dvt'], null, true);
}
```

---

## Response — SQL xử lý request từ detail

```xml
<response>
  &XMLGetUOMConversion;

  <action id="Item">
    <text><![CDATA[
if exists(select 1 from dmvt where ma_vt = @ma_vt) begin
  declare @unitCode varchar(32)
  select @unitCode = case when @$ma_dvcs = '' then @@unit else @$ma_dvcs end
  -- Trả về các trường mặc định khi chọn vật tư
  exec FastBusiness$Voucher${Module}$Item @unitCode, @ma_vt, @voucherCode, @@userID, @@admin
  return
end
]]></text>
  </action>
</response>
```

---

## Queries — Load dữ liệu detail

```xml
<queries>
  <query event="Loading">
    <text><![CDATA[
select @@fieldExternal from @@prime$partition$current a
  left join dmvt b on a.ma_vt = b.ma_vt
where @@whereClause order by @@orderByClause
]]></text>
  </query>
</queries>
```

---

## Toolbar — Chuẩn cho detail

```xml
<toolbar>
  <button command="Insert"><title v="Toolbar.Insert" e="Toolbar.Insert"/></button>
  <button command="Grow"><title v="Toolbar.Grow" e="Toolbar.Grow"/></button>
  <button command="Down"><title v="Toolbar.Down" e="Toolbar.Down"/></button>
  <button command="Clone"><title v="Toolbar.Clone" e="Toolbar.Clone"/></button>
  <button command="Remove"><title v="Toolbar.Remove" e="Toolbar.Remove"/></button>
  <button command="Separate"><title v="-" e="-"/></button>
  <button command="Export"><title v="Toolbar.Export" e="Toolbar.Export"/></button>
  <button command="Freeze"><title v="Toolbar.Freeze" e="Toolbar.Freeze"/></button>
</toolbar>
```

> Nếu có chức năng Retrieve (lấy từ đơn hàng, phiếu xuất...) thì thêm button `Retrieve` trước `Separate`.
