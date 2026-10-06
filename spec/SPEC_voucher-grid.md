# [SPEC_voucher-grid] — Grid Browser & Filter cho Chứng từ

> Dùng khi: tạo file Grid Browser (danh sách chứng từ) và Filter (điều kiện lọc).
> Đọc `SPEC_voucher-overview.md` trước để hiểu cấu trúc bảng và entity chung.

---

## A. Grid Browser

### Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE grid [
  <!ENTITY CommandWhenWhenVoucherBeforeInit SYSTEM "..\Include\Command\WhenVoucherBeforeInit.txt">
  <!ENTITY CommandWhenWhenVoucherBeforeAddNew SYSTEM "..\Include\Command\WhenVoucherBeforeAddNew.txt">
  <!ENTITY CommandWhenWhenVoucherAfterInit SYSTEM "..\Include\Command\WhenVoucherAfterInit.txt">
  <!ENTITY XMLStandardVoucherToolbar SYSTEM "..\Include\XML\ExternalVoucherToolbar.xml">

  <!ENTITY TransferID "{Controller}">
  <!ENTITY Code "{ma_ct}">

  <!ENTITY % PrintRight SYSTEM "..\Include\PrintRightGrid.ent">
  %PrintRight;

  <!ENTITY VisibleFieldController "{Controller}Grid">
  <!ENTITY % VoucherVisibleField SYSTEM "..\Include\VoucherVisibleField.ent">
  %VoucherVisibleField;

  <!ENTITY % Control.Filter SYSTEM "..\Include\Filter.ent">
  %Control.Filter;
]>
```

### Thẻ `<grid>` — Khác danh mục

```xml
<grid table="{bảng_master_cấu_trúc}" code="stt_rec" order="ngay_ct, so_ct" type="Voucher" id="{ma_ct}" uniKey="true" xmlns="urn:schemas-fast-com:data-grid">
  <title v="..." e="..."/>
  <subTitle v="Cập nhật ...: thêm, sửa, xóa..." e="Add, Edit, Delete ..."/>
  <partition .../>  <!-- xem SPEC_voucher-overview -->
  ...
</grid>
```

**Điểm khác danh mục:**
- `type="Voucher"` (danh mục không có)
- `id="{ma_ct}"` — mã chứng từ (VD: `PND`, `HDA`, `SX1`)
- `uniKey="true"` — khóa duy nhất
- Bắt buộc có thẻ `<partition>`

### Fields — Các trường cơ bản

```xml
<fields>
  <field name="stt_rec" isPrimaryKey="true" width="0" hidden="true">
    <header v="" e=""/>
  </field>
  <field name="ma_dvcs" width="100" allowFilter="&GridVoucherAllowFilter;">
    <header v="Đơn vị" e="Unit"/>
    <query>&InsertCommandFilter;</query>
  </field>
  <field name="ngay_ct" type="DateTime" dataFormatString="@datetimeFormat" width="100" allowFilter="&GridVoucherAllowFilter;">
    <header v="Ngày" e="Date"/>
    <query>&InsertCommandFilter;</query>
  </field>
  <field name="so_ct" width="100" align="right" allowFilter="&GridVoucherAllowFilter;">
    <header v="Số" e="Number"/>
    <query>&InsertCommandFilter;</query>
  </field>
  <!-- Thêm các trường nghiệp vụ theo prompt -->
</fields>
```

**Lưu ý:** Mỗi field trên Grid Browser cần `allowFilter="&GridVoucherAllowFilter;"` và `<query>&InsertCommandFilter;</query>` để hỗ trợ filter.

### Field ngoại (external) — join tên hiển thị

```xml
<field name="ten_kh%l" width="300" external="true" aliasName="b" allowFilter="&GridVoucherAllowFilter;">
  <header v="Tên khách" e="Customer Name"/>
  <query>&InsertCommandFilter;</query>
</field>
```

### Commands — Cố định

```xml
<commands>
  <!-- Grid đơn giản (không Import) dùng XMLStandardVoucherToolbar -->
  <command event="Loading">
    <text>
      &CommandWhenWhenVoucherBeforeInit;
      &CommandWhenWhenVoucherBeforeAddNew;
      &CommandWhenWhenVoucherAfterInit;
    </text>
  </command>
</commands>
```

**Grid có thêm chức năng (Import, Print...)** — cần thêm event Showing, Closing, script.

### Queries — **Phần quan trọng nhất**

```xml
<queries>
  <query event="Loading">
    <text>
      <![CDATA[exec FastBusiness$App$Voucher$Loading
@@id, @@master, @@prime, @@partition, @@expression, @@extension, @@pageCount,
'stt_rec', @@textList, @@textExternal,
'{join_clause}', @@textOrderBy, @@admin, @@userID, @@viewAccessMode, 0, @@queryString]]>
    </text>
  </query>

  <query event="Declare">
    <text>&DeclareCommandFilter;</text>
  </query>

  <query event="Finding">
    <text>
      <![CDATA[exec FastBusiness$App$Voucher$Finding
@@id, @@master, @@prime, @@inquiry, @@partition, @@expression, @@increase, @@extension,
@@refresh, @@pageIndex, @@pageCount, @@lastPage, @@lastCount, @@firstItem, @@lastItem,
@@keyMaster, @@keyDetail, 'stt_rec', @@textList, @@textExternal,
'{join_clause}', @@textOrderBy, @@admin, @@userID, @@viewAccessMode,
@ngay_ct1, @ngay_ct2, @so_ct1, @so_ct2, @status, @user_id0, @ma_dvcs]]>
    </text>
  </query>
</queries>
```

**`{join_clause}`**: chuỗi LEFT JOIN để lấy tên hiển thị. VD:
- `'a left join dmkh b on a.ma_kh = b.ma_kh'` (có tên khách)
- `''` (không cần join)

> Loading và Finding dùng chung join_clause.

### Toolbar — 2 dạng

**Dạng 1: Chuẩn (không import)**
```xml
&XMLStandardVoucherToolbar;
```
Đặt sau `</views>`, trước `</grid>`.

**Dạng 2: Có Import/Download** — khai báo tường minh:
```xml
<toolbar>
  <button command="New"><title v="Toolbar.New" e="Toolbar.New"/></button>
  <button command="Edit"><title v="Toolbar.Edit" e="Toolbar.Edit"/></button>
  <button command="Delete"><title v="Toolbar.Delete" e="Toolbar.Delete"/></button>
  <button command="Clone"><title v="Toolbar.Copy" e="Toolbar.Copy"/></button>
  <button command="Search"><title v="Toolbar.Search" e="Toolbar.Search"/></button>
  <button command="View"><title v="Toolbar.View" e="Toolbar.View"/></button>
  <button command="Print"><title v="Toolbar.Print" e="Toolbar.Print"/></button>
  <button command="-"><title v="-" e="-"/></button>
  <button command="ImportData">
    <title v="Lấy dữ liệu từ tệp..." e="Import Data from File..."/>
  </button>
  <button command="Download">
    <title v="Tải tệp mẫu..." e="Download Template File..."/>
  </button>
  <button command="Separate"><title v="-" e="-"/></button>
  <button command="Freeze"><title v="Toolbar.Freeze" e="Toolbar.Freeze"/></button>
</toolbar>
```

---

## B. Filter

### Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE dir [
  <!ENTITY XMLWhenFilterInit SYSTEM "..\Include\XML\WhenFilterInit.xml">
  <!ENTITY XMLWhenFilterLoading SYSTEM "..\Include\XML\WhenFilterLoading.xml">
  <!ENTITY XMLWhenFilterClosing SYSTEM "..\Include\XML\WhenFilterClosing.xml">
  <!ENTITY ScriptFilterInit SYSTEM "..\Include\Javascript\FilterInit.txt">
  <!ENTITY TableDetail "{tên_bảng_detail}">

  <!ENTITY % VoucherDeleteLog SYSTEM "..\Include\VoucherDeleteLog.ent">
  %VoucherDeleteLog;
]>

<dir table="{bảng_master_cấu_trúc}" code="stt_rec" order="ngay_ct, so_ct"
     cache="true" xmlns="urn:schemas-fast-com:data-dir">
  <!-- Loại không phân kỳ: thêm id="{ma_ct}" -->
  <title v="Điều kiện lọc" e="Filter Condition"/>
```

> **`TableDetail`**: entity khai báo tên bảng detail, dùng cho attribute `information` trong lookup detail.
> Phân kỳ: `d74` (không cần $yyyymm). Không phân kỳ: `ctsx`.

### Fields cố định

```xml
<!-- Luôn có: Ngày từ/đến -->
<field name="ngay_ct1" type="DateTime" dataFormatString="@datetimeFormat" align="left"
       allowNulls="false" aliasName="fromDate" defaultValue="new Date()">
  <header v="Chứng từ từ ngày" e="Date From"/>
  <footer v="Ngày chứng từ từ/đến" e="Date from/to"/>
</field>
<field name="ngay_ct2" type="DateTime" dataFormatString="@datetimeFormat" align="left"
       allowNulls="false" aliasName="toDate" defaultValue="new Date()">
  <header v="Chứng từ đến ngày" e="Date to"/>
</field>

<!-- Luôn có: Số chứng từ từ/đến -->
<field name="so_ct1" dataFormatString="@upperCaseFormat" align="right" maxLength="-100"
       filterSource="voucherNumber">
  <header v="Số chứng từ từ/đến" e="Voucher No. from/to"/>
  <items style="Mask"/>
</field>
<field name="so_ct2" dataFormatString="@upperCaseFormat" align="right" maxLength="-100"
       filterSource="voucherNumber">
  <header v="" e=""/>
  <items style="Mask"/>
</field>
```

### Thuộc tính `filterSource` và `operation`

| filterSource | Dùng cho | Lưu vào cột bảng i$ |
|---|---|---|
| `voucherNumber` | `so_ct1`, `so_ct2` | _(xử lý riêng)_ |
| `master` | Trường thuộc bảng master | `m$` |
| `detail` | Trường thuộc bảng detail | `d$` |

**`operation`**: ID ngắn gọn cho trường, lưu dạng `#10$ma_kh#20$ma_gd#30$ma_nt` trong cột `m$` hoặc `d$` của bảng inquiry.

```xml
<!-- Master filter -->
<field name="ma_kh" filterSource="master" operation="10">
  <header v="Mã khách" e="Customer"/>
  <items style="AutoComplete" controller="Customer" reference="ten_kh%l" key="status = '1'" check="1 = 1"/>
</field>

<!-- Detail filter — thêm information="&TableDetail;" -->
<field name="ma_kho" filterSource="detail" categoryIndex="1" operation="10">
  <header v="Mã kho" e="Site"/>
  <items style="AutoComplete" controller="Site" reference="ten_kho%l" key="status = '1'" check="1 = 1" information="&TableDetail;"/>
</field>
```

### Fields cố định cuối form

```xml
<!-- Đơn vị -->
<field name="ma_dvcs" categoryIndex="-1">
  <header v="Đơn vị" e="Unit"/>
  <items style="AutoComplete" controller="Unit" reference="ten_dvcs%l" key="status = '1'" check="1 = 1"/>
</field>
<field name="ten_dvcs%l" readOnly="true" external="true" defaultValue="''">
  <header v="" e=""/>
</field>

<!-- Người sử dụng -->
<field name="user_id0" dataFormatString="0, 1" clientDefault="1" align="right" inactivate="true" categoryIndex="-1">
  <header v="Người sử dụng" e="User"/>
  <footer v="1 - Lọc theo người sử dụng, 0 - Không" e="1 - Filter by User, 0 - No"/>
  <items style="Mask"/>
</field>

<!-- Trạng thái — giá trị phụ thuộc nghiệp vụ -->
<field name="status" dataFormatString="*, 0, 1, 2, 3&VoucherLogStatusFilter;" clientDefault="*" align="right" inactivate="true" categoryIndex="-1">
  <header v="Trạng thái" e="Status"/>
  <footer v="Ký tự &lt;span style=&quot;color:#008200;&quot;&gt;[*]&lt;/span&gt; - Hiện tất cả các trạng thái&VoucherLogStatusDescription.v;"
          e="&lt;span style=&quot;color:#008200;&quot;&gt;[*]&lt;/span&gt; - Show all status information&VoucherLogStatusDescription.e;"/>
  <items style="Mask"/>
</field>
```

### Commands — 2 dạng

**Phân kỳ (có Loading, Closing):**
```xml
<commands>
  &XMLWhenFilterInit;
  &XMLWhenFilterLoading;
  &XMLWhenFilterClosing;
</commands>
```

**Không phân kỳ (chỉ Init):**
```xml
<commands>
  &XMLWhenFilterInit;
</commands>
```

### Script — Cố định

```xml
<script>
  <text>
    &ScriptFilterInit;
    <![CDATA[/* <flatten type="Javascript"> */
function onChange$VoucherFilter$Tab(sender, e) {
  sender.parentForm.focusWhenTabChanged(['{first_field_tab1}', '{first_field_tab9}']);
}
function active$VoucherFilter$(sender) {
  sender._tabContainer.add_activeTabChanged(onChange$VoucherFilter$Tab);
  sender._tabContainer._loaded = true;
}
function close$VoucherFilter$(sender) {
  if (sender._tabContainer) try {
    sender._tabContainer.remove_activeTabChanged(onChange$VoucherFilter$Tab);
  } catch (ex) {}
}
/* </flatten> */]]>
  </text>
</script>
```

> `{first_field_tab1}`: field đầu tiên trong tab categoryIndex=1 (VD: `ma_kho`).
