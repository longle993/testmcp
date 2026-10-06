# [SPEC_voucher-dir] — Dir XML cho Chứng từ (Form nhập liệu)

> Dùng khi: tạo form thêm/sửa/xóa chứng từ.
> Đây là file phức tạp nhất — chứa fields, views, commands (SQL lifecycle), script (JS), response.
> Đọc `SPEC_voucher-overview.md` trước.

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE dir [
  <!-- Entity chung bắt buộc — xem SPEC_voucher-overview -->
  ...
  <!-- Entity riêng chứng từ này -->
  <!ENTITY DetailVariable "@dXX">
  <!ENTITY DetailTable "dXX$$partition$current">
  <!ENTITY AfterUpdate "...">
  <!ENTITY Post "...">      <!-- nghiệp vụ riêng, có thể để trống -->
  <!ENTITY Delete "...">    <!-- xóa bảng post khi xóa chứng từ -->
]>

<dir table="{bảng_master_cấu_trúc}" code="stt_rec" order="ngay_ct, so_ct" id="{ma_ct}" type="Voucher" uniKey="true" replication="1" navigation="true" name="cookie" check="true" xmlns="urn:schemas-fast-com:data-dir">
  <title v="..." e="..."/>
  <partition .../>
```

### Thuộc tính `<dir>` — Khác danh mục

| Thuộc tính | Giá trị | Mô tả |
|---|---|---|
| `type` | `"Voucher"` | Bắt buộc cho chứng từ |
| `id` | `"{ma_ct}"` | Mã chứng từ |
| `uniKey` | `"true"` | Khóa duy nhất |
| `replication` | `"1"` | Hỗ trợ sao chép |
| `navigation` | `"true"` | Cho phép chuyển chứng từ (trước/sau) |
| `name` | `"cookie"` | Tên cookie lưu trạng thái |
| `check` | `"true"` | Bật kiểm tra trước lưu (Checking event) |

---

## Fields — Phần master

### Trường bắt buộc (mọi chứng từ)

```xml
<!-- stt_rec — khóa ẩn -->
<field name="stt_rec" isPrimaryKey="true" readOnly="true" hidden="true">
  <header v="" e=""/>
</field>

<!-- Mã giao dịch -->
<field name="ma_gd" allowNulls="false" clientDefault="Default" defaultValue="{giá_trị_mặc_định}">
  <header v="Mã giao dịch" e="Transaction Code"/>
  <items style="AutoComplete" controller="TransactionCode" reference="ten_gd%l" key="ma_ct = @@id and status = '1'" check="ma_ct = @@id" information="ma_gd$dmmagd.ten_gd%l" row="1"/>
  <clientScript><![CDATA[onchange="onChange$Voucher$Transaction(this);"]]></clientScript>
</field>
<field name="ten_gd%l" readOnly="true" external="true" clientDefault="Default" defaultValue="''">
  <header v="" e=""/>
</field>
<field name="loai_ct" hidden="true" width="0" clientDefault="Default">
  <header v="" e=""/>
</field>

<!-- Diễn giải -->
<field name="dien_giai">
  <header v="Diễn giải" e="Memo"/>
</field>

<!-- Sổ & Số chứng từ (Entity include) -->
&XMLVoucherBookAndNumberFields;

<field name="so_ct" dataFormatString="@upperCaseFormat" align="right" allowNulls="false">
  <header v="Số {tên}" e="{Name} Number"/>
  <items style="Mask"/>
</field>

<!-- Ngày -->
<field name="ngay_lct" type="DateTime" dataFormatString="@datetimeFormat" align="left" allowNulls="false">
  <header v="Ngày lập" e="Voucher Date"/>
</field>
<field name="ngay_ct" type="DateTime" dataFormatString="@datetimeFormat" align="left" allowNulls="false" inactivate="true">
  <header v="Ngày hạch toán" e="Posting Date"/>
</field>
```

### Tiền tệ (nếu chứng từ có ngoại tệ)

```xml
<field name="ma_nt" clientDefault="Default" allowNulls="false" inactivate="true">
  <header v="Mã nt" e="Currency"/>
  <items style="AutoComplete" controller="Currency" reference="ten_nt%l"
         key="status = '1'" check="1=1" information="ma_nt$dmnt.ten_nt%l"/>
</field>
<field name="ten_nt%l" clientDefault="Default" readOnly="true" hidden="false"
       external="true" defaultValue="''">
  <header v="" e=""/>
</field>
<field name="ty_gia" type="Decimal" dataFormatString="@exchangeRateInputFormat"
       clientDefault="Default" defaultValue="1">
  <header v="Tỷ giá" e="Ex. Rate"/>
  <items style="Numeric"/>
</field>
```

### Trạng thái

```xml
<field name="status" inactivate="true" clientDefault="{giá_trị_mặc_định}">
  <header v="Trạng thái" e="Status"/>
  <items style="DropDownList">
    <item value="0"><text v="0. Lập chứng từ" e="0. No Action"/></item>
    <!-- Thêm status theo nghiệp vụ -->
    <item value="2"><text v="2. Chuyển sổ cái" e="2. Release"/></item>
    &VoucherLogStatusField;
  </items>
</field>
```

### Field Grid Detail — Nhúng grid vào form

```xml
<field name="{tên_bảng_detail}" allowNulls="false" external="true" clientDefault="0" defaultValue="0" rows="216" filterSource="Tidy" categoryIndex="1">
  <header v="" e=""/>
  <label v="Chi tiết" e="Detail"/>
  <items style="Grid" controller="{Controller}Detail" row="1">
    <item value="ForeignKey">
      <text v="String: stt_rec, stt_rec" e="String: stt_rec, stt_rec"/>
    </item>
  </items>
</field>
```

> `name` = tên bảng detail (VD: `d74` cho phân kỳ, `ctsx` cho không phân kỳ).
> `categoryIndex="1"` = nằm trong Tab "Chi tiết".

### Trường tổng (dưới grid)

```xml
<field name="t_so_luong" type="Decimal" dataFormatString="@quantityInputFormat" categoryIndex="-1" disabled="true">
  <header v="Tổng cộng" e="Total"/>
  <items style="Numeric"/>
</field>
<field name="t_tien_nt" type="Decimal" dataFormatString="@foreignCurrencyAmountInputFormat" categoryIndex="-1" disabled="true">
  <header v="Tổng tiền nt" e="Total FC"/>
  <items style="Numeric"/>
</field>
```

### Trường ẩn cuối cùng

```xml
<field name="ma_dvcs" hidden="true" readOnly="true">
  <header v="" e=""/>
</field>
<field name="cookie" external="true" hidden="true" readOnly="true" defaultValue="''" allowContain="true">
  <header v="" e=""/>
</field>
```

---

## Views — Layout form

```xml
<views>
  <view id="Dir" height="276" anchor="6" split="8">
    <item value="100, 30, 70, 129, 100, 8, 100, 8, 58, 42, 8, 100, 0, 0"/>
    <!-- Dòng master (bên trái + phải) -->
    <item value="...pattern...: [field1].Label, [field1], [ten_field1], ..."/>
    ...
    <!-- Grid detail chiếm full width -->
    <item value="1: [{tên_bảng_detail}]"/>
    <!-- Dòng tổng (dưới grid) -->
    <item value="...pattern...: [t_so_luong].Label, [t_so_luong], ..."/>

    &ListView;
    &PostView;

    <categories>
      <category index="1" columns="769" anchor="1">
        <header v="Chi tiết" e="Detail"/>
      </category>
      <!-- Tab khác nếu có -->
      &ListCategory;
      &PostCategory;
      <category index="-1" columns="..." anchor="3">
        <header v="" e=""/>
      </category>
    </categories>
  </view>
</views>
```

---

## Commands — SQL Lifecycle

### Loading (mở form Edit)

```xml
<command event="Loading">
  <text>
    &CommandWhenVoucherLoading;
    &CommandGetVoucherNumber;
    <![CDATA[
declare @message nvarchar(4000)
]]>&CommandQueryVoucherNumber;<![CDATA[ + ';active$Voucher$(this);']]>
    <!-- Thêm query nghiệp vụ nếu cần -->
    &CommandCheckVoucherHandleBeforeEdit;
    &CommandWhenVoucherBeforeEdit;
    <![CDATA[
select @message as message
return
]]>
  </text>
</command>
```

### Scattering (load lần 2 — chưa tắt form)

```xml
<command event="Scattering">
  <text>
    &CommandGetVoucherNumber;
    <![CDATA[
declare @message nvarchar(4000)
]]>&CommandScatterVoucherNumber;<![CDATA[ + ';scatter$Voucher$(this);']]>
    &CommandCheckVoucherHandleBeforeEdit;
    &CommandWhenVoucherBeforeEdit;
    <![CDATA[
select @message as message
return
]]>
  </text>
</command>
```

### InitExternalFields (lấy dữ liệu external)

```xml
<command event="InitExternalFields">
  <text>
    &CommandExternalFieldDeclare;
    &CommandExternalFieldSelect;<![CDATA[ from @@prime$partition$current where stt_rec = @stt_rec]]>
    &CommandExternalFieldSet;
    &CommandExternalFieldQuery;<![CDATA[
return
]]>
  </text>
</command>
```

### Inserting (trước khi insert)

```xml
<command event="Inserting">
  <text>
    &CommandCheckVoucherNumberDeclare;
    &CommandCheckLockedDate;
    &CommandCheckVoucherHandleBeforeSave;
    &CommandCheckVoucherNumberExecute;
    &CommandGetIdentityNumber;
    <![CDATA[
select @ma_dvcs = @@unit, @ma_ct = @@id,
       @datetime0 = getdate(), @datetime2 = getdate(),
       @user_id0 = @@userID, @user_id2 = @@userID
update @{detail} set stt_rec = @stt_rec, ma_ct = @ma_ct,
       ngay_ct = @ngay_ct, so_ct = @so_ct
]]>
  </text>
</command>
```

> Nếu không phân kỳ, thêm: `@ma_nt = '', @ty_gia = 1` nếu không có ngoại tệ.

### Inserted (sau khi insert master, insert detail + post)

```xml
<command event="Inserted">
  <text>
    <![CDATA[
insert into {detail_table}$$partition$current select * from @{detail}
]]>
    &AfterUpdate;
    <!-- Post nghiệp vụ nếu có -->
    <![CDATA[
exec FastBusiness$App$IncreaseVoucherNumber @ma_nk, @@id, @so_ct
]]>
    &ListDeclare;
    &ListWarning;
    &ListCommand;
    &PostInserted;
    &ListQuery;<![CDATA[, @stt_rec as stt_rec, @@unit as ma_dvcs else
select @stt_rec as stt_rec, @@unit as ma_dvcs
return
]]>
  </text>
</command>
```

### Updating (trước khi update)

```xml
<command event="Updating">
  <text>
    &CommandCheckVoucherNumberDeclare;
    &CommandRecordHasBeenChanged;
    &CommandCheckLockedDate;     <!-- chỉ phân kỳ -->
    &CommandCheckVoucherHandleBeforeSave;
    &CommandCheckVoucherNumberExecute;
    <![CDATA[
<!-- Phân kỳ: xử lý chuyển kỳ -->
#IF '$partition$current' <> '$partition$previous' #THEN
  insert into @@prime$partition$current select * from @@prime$partition$previous where stt_rec = @stt_rec
  #IF ${detail}.NewValue = ${detail}.OldValue #THEN
    insert into {detail}$$partition$current select * from {detail}$$partition$previous where stt_rec = @stt_rec
    delete {detail}$$partition$previous where stt_rec = @stt_rec
  #END
  delete @@prime$partition$previous where stt_rec = @stt_rec
  delete @@inquiry$partition$previous where stt_rec = @stt_rec
#END

#IF ${detail}.NewValue = ${detail}.OldValue #THEN
  update {detail}$$partition$current set stt_rec = @stt_rec, ngay_ct = @ngay_ct, so_ct = @so_ct, ma_ct = @@id where stt_rec = @stt_rec
#ELSE
  update @{detail} set stt_rec = @stt_rec, ngay_ct = @ngay_ct, so_ct = @so_ct, ma_ct = @@id
  delete {detail}$$partition$previous where stt_rec = @stt_rec
#END
]]>
  </text>
</command>
```

> **Không phân kỳ**: bỏ block `#IF '$partition$current' <> '$partition$previous'`, thay `$$partition$current` bằng tên bảng trực tiếp.

### Updated (sau khi update)

```xml
<command event="Updated">
  <text>
    <![CDATA[
#IF ${detail}.NewValue <> ${detail}.OldValue #THEN
  insert into {detail}$$partition$current select * from @{detail}
#END
update @@prime$partition$current set datetime2 = getdate(), user_id2 = @@userID where stt_rec = @stt_rec
]]>
    &AfterUpdate;
    <!-- Post nghiệp vụ -->
    &EndUpdatedVoucherNumber;
    &ListDeclare;
    &ListWarning;
    &ListCommand;
    &ListQuery;
  </text>
</command>
```

### Deleting (xóa chứng từ)

```xml
<command event="Deleting">
  <text>
    &CommandCheckVoucherHandleBeforeDelete;
    &CommandWhenVoucherBeforeDelete;
    <![CDATA[
delete @@inquiry$partition$current where stt_rec = @stt_rec]]>&VoucherLogKey;<![CDATA[
delete {detail}$$partition$current where stt_rec = @stt_rec]]>&VoucherLogKey;<![CDATA[
delete @@master where stt_rec = @stt_rec]]>&VoucherLogKey;<![CDATA[
]]>
    <!-- Xóa bảng post nếu có -->
    &Delete;
    &VoucherLogUpdateStatus;
    &VoucherLogBeginComment;
  </text>
</command>
```

### Deleted

```xml
<command event="Deleted">
  <text>
    &VoucherLogEndComment;
    <![CDATA[
declare @invoke nvarchar(4000)
select @invoke = ''
]]>&ListDeleted;<![CDATA[
]]>&PostDeleted;<![CDATA[
select @invoke as invoke
]]>
  </text>
</command>
```

### Checking (JS kiểm tra trước lưu)

```xml
<command event="Checking">
  <text>
    <![CDATA[/* <flatten type="Javascript"> */
var f = this, id = f.get_id(), v = f._language == 'v';
var g = f.getItem('{detail}')._controlBehavior;
// Thêm validate nghiệp vụ ở đây
/* </flatten> */]]>
    &ListChecking;
    &PostChecking;
  </text>
</command>
```

---

## Script — JS chuẩn cho Dir chứng từ

### Các hàm bắt buộc

```javascript
function init$Voucher$(f) {
  initialize(f, '{contact_field}', 'ma_gd', 'loai_ct', 'status');
  // contact_field: 'ong_ba' hoặc null
}
function active$Voucher$(f) {
  // ScriptActiveVoucher + setup tab + load visible field
}
function scatter$Voucher$(f) {
  // ScriptScatterVoucher + reload
}
function close$Voucher$(f) {
  // ScriptCloseVoucher + cleanup
}
```

### ExecuteCommand — Gather (đánh lại số dòng)

```javascript
function on$Voucher$ExecuteCommand(sender, e) {
  var action = e.type.Action, f = sender,
      g = f.getItem('{detail}')._controlBehavior;
  switch (action) {
    case 'Gather':
      g.setSequenceNumber('line_nbr');
      g.setContinuance('stt_rec0');
      break;
    case 'Explore':
      break;
    case 'Shown':
      break;
  }
}
```

> **Gather**: framework gọi trước khi lưu. `setSequenceNumber` đánh lại line_nbr, `setContinuance` đánh lại stt_rec0 cho dòng mới.

### ResponseComplete

```javascript
function on$Voucher$ResponseComplete(sender, e) {
  var f = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'Loading':
      // VoucherNumberLoading entity
      break;
    case 'Scattering':
      // VoucherNumberScattering entity
      break;
    case 'Transaction':
      f.setItemValue('loai_ct', result[0].Value);
      break;
    case 'Navigating':
      break;
    // Thêm case nghiệp vụ
  }
}
```

### Currency (nếu có ngoại tệ)

```javascript
objectBehavior$Voucher$Currency = {
  // ScriptCurrency entity
  create: function(f) {
    var d = f.getItem('ngay_lct'), p = f.getItem('ngay_ct'),
        c = f.getItem('ma_nt'), r = f.getItem('ty_gia'),
        g = f.getItem('{detail}')._controlBehavior;
    var v = f._currencyBehavior = $create(
      FastBusiness.AjaxControlExtender.Currency,
      {id: f._id + '_currencyBehavior', currencyDate: d, referenceDate: p,
       exchangeRate: r, form: f, baseCurrency: f._baseCurrency, flush: true},
      null, null, c);
    v.set_currencyFields('t_tien_nt:t_tien');
    v.addGridFields(g, 'gia_nt,tien_nt:gia,tien');
    v.addTotalFields(g, [['t_tien_nt', 'tien_nt'], ['t_tien', 'tien']]);
    v.set_referenceFields(null);
    // CurrencyDateChanged entity
  }
}
```

---

## Response — SQL cho các request từ JS

```xml
<response>
  <action id="Reading">
    <text>&CommandSetVoucherNumber;</text>
  </action>

  <action id="Transaction">
    <text><![CDATA[
select rtrim(loai_ct) as loai_ct from dmmagd where ma_ct = '{ma_ct}' and ma_gd = @ma_gd
return
]]></text>
  </action>

  &XMLGetVoucherNumber;
  &XMLGetExchangeRate;   <!-- nếu có ngoại tệ -->

  <!-- Thêm action nghiệp vụ (Customer, ...) -->
  &ListTicket;
</response>
```

---

## Sự khác biệt giữa phân kỳ và không phân kỳ

| Yếu tố | Phân kỳ (IR) | Không phân kỳ (MO) |
|---|---|---|
| Detail variable | `@d74` | `@ctsx` |
| Detail table SQL | `d74$$partition$current` | `ctsx` |
| Insert detail | `insert into d74$$partition$current select * from @d74` | `insert into ctsx select * from @ctsx` |
| Updating — chuyển kỳ | Có block `#IF '$partition$current' <> '$partition$previous'` | Không cần |
| CommandCheckLockedDate | Có | Không (thường) |
| Currency entities | Thường có | Thường không |
| ngay_lct xử lý | Tự động | Có thể cần JS: `ngay_lct = ngay_ct` |
