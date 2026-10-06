# [SPEC_report-filter] — Filter XML cho Báo cáo

> Dùng khi: tạo file Filter XML cho báo cáo.
> Thư mục: Controllers/Filter/{Controller}.xml
> Đọc `SPEC_report-overview.md` trước.

---

## Khác biệt so với Filter danh mục

| Đặc điểm | Filter danh mục | **Filter báo cáo** |
|---|---|---|
| Thẻ `<dir>` | `table`, `code`, `order` | **`id`, `type="Report"`** |
| `id` | Không có | **Số thứ tự bảng hiển thị ở Grid** (tính từ 0) |
| `type` | Không có | **`"Report"`** |
| Processing | Không có (Grid tự query bảng) | **Có — gọi Store trả dữ liệu** |
| Entity | Ít hơn | **Nhiều hơn** (ReportSign, ReportMargin, Outline) |
| Tab chữ ký/canh chỉnh | Không | **Có** (mặc định) |

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE dir [
  <!-- Entity chuẩn Filter -->
  <!ENTITY XMLWhenFilterLoading SYSTEM "..\Include\XML\WhenFilterLoading.xml">
  <!ENTITY XMLWhenFilterClosing SYSTEM "..\Include\XML\WhenFilterClosing.xml">
  <!ENTITY ScriptFilterInit SYSTEM "..\Include\Javascript\FilterInit.txt">
  <!ENTITY XMLWhenFilterQuerying SYSTEM "..\Include\XML\WhenFilterQuerying.xml">
  <!ENTITY OutlineCss SYSTEM "..\Include\Javascript\OutlineCss.txt">
  <!ENTITY OutlineEntry SYSTEM "..\Include\Javascript\OutlineEntry.txt">
  <!ENTITY OnSelectionOutline SYSTEM "..\Include\Javascript\OnSelectionOutline.txt">

  <!-- Entity riêng báo cáo -->
  <!ENTITY Controller "{Controller}">
  <!ENTITY DynamicReportFields ",'&Controller;', '#$query', '@@sysDatabaseName'">
  <!ENTITY JavascriptReportFilter SYSTEM "..\Include\Javascript\ReportFilter.txt">

  <!-- Entity số tham chiếu (nếu cần) -->
  <!ENTITY % ReferenceNumber SYSTEM "..\Include\ReferenceNumber.ent">
  %ReferenceNumber;

  <!-- Entity ngoại tệ (nếu cần) -->
  <!ENTITY % Tiny.Currency SYSTEM "..\Include\Tiny.Currency.ent">
  %Tiny.Currency;

  <!-- Entity chữ ký + canh chỉnh (mặc định có) -->
  <!ENTITY LineCounter "{N}">
  <!ENTITY ExtensionCounter "1">
  <!ENTITY ReportMarginCategoryIndex "&ReportSign.Filter.CategoryIndex;">
  <!ENTITY % ReportMargin SYSTEM "..\Include\ReportMargin.ent">
  %ReportMargin;
  <!ENTITY % TabHeightFomula SYSTEM "..\Include\TabHeightFomula.ent">
  %TabHeightFomula;
]>

<dir id="{datasetIndex}" type="Report" cache="true" xmlns="urn:schemas-fast-com:data-dir">
  <title v="Điều kiện lọc" e="Filter Condition"/>
  <fields> ... </fields>
  <views> ... </views>
  <commands> ... </commands>
  <script> ... </script>
  &OutlineCss;
</dir>
```

---

## Thuộc tính `<dir>` cho báo cáo

| Thuộc tính | Bắt buộc | Mô tả |
|---|---|---|
| `id` | ✅ | **Index** của bảng dữ liệu hiển thị ở Grid (tính từ 0). VD: Store trả 2 bảng, bảng 2 hiển thị → `id="1"` |
| `type` | ✅ | Cố định: `"Report"` |
| `cache` | ✅ | `"true"` — giữ giá trị lọc khi quay lại |
| `xmlns` | ✅ | `urn:schemas-fast-com:data-dir` |

> **Không có** `table`, `code`, `order` — vì báo cáo không bind trực tiếp vào bảng DB.

### Cách xác định `id`

Store báo cáo trả nhiều bảng:

| Store trả về | Bảng hiển thị ở Grid | `id` |
|---|---|---|
| 1 bảng | Bảng 1 | `0` |
| 2 bảng (bảng 1 = filter info, bảng 2 = data) | Bảng 2 | `1` |
| 3 bảng (bảng 1 = filter, bảng 2 = data, bảng 3 = pivot) | Bảng 2 | `1` |

---

## Entity — Giải thích

### Entity luôn có

| Entity | Mô tả |
|---|---|
| `XMLWhenFilterLoading` | Sự kiện Loading chuẩn |
| `XMLWhenFilterClosing` | Sự kiện Closing chuẩn |
| `ScriptFilterInit` | JS khởi tạo Filter (`init$VoucherFilter$`) |
| `XMLWhenFilterQuerying` | Hỗ trợ query Filter (nếu báo cáo có Querying) |
| `OutlineCss` | CSS cho outline/tree |
| `OutlineEntry` | JS cho outline |
| `OnSelectionOutline` | JS xử lý selection outline |
| `Controller` | Tên controller báo cáo |
| `DynamicReportFields` | Chuỗi tham số mở rộng cho Store |
| `JavascriptReportFilter` | JS chuẩn cho Report Filter |

### Entity chữ ký & canh chỉnh (mặc định có)

| Entity | Mô tả |
|---|---|
| `LineCounter` | Số dòng field trên tab chính — **đếm số `<item>` trong view tab chính** |
| `ExtensionCounter` | Thường = `"1"` |
| `ReportMarginCategoryIndex` | Lấy từ `&ReportSign.Filter.CategoryIndex;` |
| `ReportSign.Filter.Fields` | Fields cho tab chữ ký |
| `ReportSign.Filter.Views` | Views cho tab chữ ký |
| `ReportSign.Filter.Categories` | Categories cho tab chữ ký |
| `ReportMarginFieldExtend` | Fields cho tab canh chỉnh |
| `ReportMarginView` | View cho tab canh chỉnh |
| `ReportSign.Filter.Initialize` | SQL khởi tạo chữ ký |
| `ReportSign.Filter.Active` | JS active chữ ký |
| `ReportSign.Filter.Query` | SQL lấy chữ ký khi Processing |
| `ReportMarginProcessing` | SQL lấy canh chỉnh khi Processing |

> **Không cần chữ ký**: khi prompt ghi "không cần chữ ký", bỏ tất cả entity `ReportSign...` và `ReportMargin...`.

---

## `<fields>` — Các field điều kiện lọc

### Field ngày từ/đến (hầu như luôn có)

```xml
<field name="tu_ngay" type="DateTime" dataFormatString="@datetimeFormat"
       allowNulls="false" aliasName="fromDate" defaultValue="new Date()">
  <header v="Từ ngày" e="Date from"/>
  <footer v="Từ/đến ngày" e="Date from/to"/>
</field>
<field name="den_ngay" type="DateTime" dataFormatString="@datetimeFormat"
       allowNulls="false" aliasName="toDate" defaultValue="new Date()">
  <header v="Đến ngày" e="Date to"/>
</field>
```

### Field chứng từ từ/đến số

```xml
<field name="so_ct1" align="right" dataFormatString="@upperCaseFormat">
  <header v="Chứng từ từ/đến số" e="Voucher No. from/to"/>
  <items style="Mask"/>
</field>
<field name="so_ct2" align="right" dataFormatString="@upperCaseFormat">
  <header v="" e=""/>
  <items style="Mask"/>
</field>
```

### Field Lookup chọn nhiều (tài khoản, đơn vị)

```xml
<field name="tk" categoryIndex="1">
  <header v="Danh sách tài khoản" e="Account List"/>
  <items style="Lookup" controller="Account" key="status = '1'" check="1 = 1"/>
</field>
```

### Field AutoComplete (khách hàng, ngoại tệ, ...)

```xml
<field name="ma_kh" categoryIndex="1">
  <header v="Mã khách" e="Customer"/>
  <items style="AutoComplete" controller="Customer" reference="ten_kh%l"
         key="status = '1'" check="1 = 1"/>
</field>
<field name="ten_kh%l" readOnly="true" external="true" categoryIndex="1">
  <header v="" e=""/>
</field>
```

### Field DropDownList (mẫu báo cáo)

```xml
<field name="mau_bc" categoryIndex="1">
  <header v="Mẫu báo cáo" e="Report Form"/>
  <clientScript>&OnSelectionOutline;</clientScript>
  <items style="DropDownList">
    <item value="10">
      <text v="Mẫu chuẩn" e="Standard Form"/>
    </item>
    <item value="20">
      <text v="Mẫu ngoại tệ" e="FC Form"/>
    </item>
  </items>
</field>
```

### Field Mask (ghi nợ/có, hiện/ẩn)

```xml
<field name="ghi_no_co" dataFormatString="1, 2, *" clientDefault="*"
       align="right" categoryIndex="1">
  <header v="Ghi nợ/có" e="Debit/Credit"/>
  <footer v="1 - Nợ, 2 - Có, * - Tất cả" e="1 - Debit, 2 - Credit, * - All"/>
  <items style="Mask"/>
</field>
```

### Field ẩn (maxLength — chứa giá trị kỹ thuật)

```xml
<field name="maxLength" type="Int16" readOnly="true" hidden="true" maxLength="-100">
  <header v="" e=""/>
</field>
```

### Field chữ ký & canh chỉnh (entity)

```xml
&ReportSign.Filter.Fields;
&ReportMarginFieldExtend;
```

> Đặt cuối cùng trong `<fields>`, trước thẻ đóng `</fields>`.

---

## Báo cáo HR — Bộ điều kiện lọc chuẩn

> Áp dụng khi tạo báo cáo thuộc phân hệ **HR / Nhân sự**.

### Quy tắc bắt buộc trước khi tạo Filter

Khi prompt yêu cầu tạo **báo cáo HR**, trước khi sinh file `Controllers/Filter/{Controller}.xml`, phải hỏi người dùng:

**"Báo cáo HR này có sử dụng bộ điều kiện lọc HR chuẩn gồm Kỳ/Năm, Bộ phận, Nhân viên, Nhóm bộ phận và Nhóm nhân viên không?"**

Bộ điều kiện HR chuẩn gồm:

- `ky` — Kỳ
- `nam` — Năm
- `ma_bp` + `ten_bp%l` — Bộ phận
- `ma_nv` + `ten_nv` — Nhân viên
- `nh_bp1`, `nh_bp2`, `nh_bp3` + các field tên nhóm — Nhóm bộ phận
- `nh_nv1`, `nh_nv2`, `nh_nv3` + các field tên nhóm — Nhóm nhân viên

Nếu người dùng trả lời **có**:

1. Thêm trực tiếp các field HR chuẩn bên dưới vào Filter.
2. **Không cần đọc `SPEC_lookup.md`.**
3. **Không cần đọc `SPEC_lookup-other.md`.**
4. **Không cần tìm báo cáo HR khác làm nguồn tham khảo cho các field này.**
5. Giữ nguyên `controller`, `reference`, `key`, `check` đã khai báo trong mẫu chuẩn.
6. Thêm các tham số tương ứng vào `Processing` và Store theo đúng thứ tự thực tế của Filter.
7. Nếu người dùng chỉ chọn một phần bộ lọc HR thì chỉ thêm các field được chọn.

Nếu người dùng trả lời **không**, tạo Filter theo yêu cầu báo cáo bình thường.

> Nên hỏi **một lần theo cả bộ**, không hỏi từng field riêng lẻ. Người dùng có thể trả lời như: "Có, nhưng bỏ nhóm nhân viên" hoặc "Chỉ dùng kỳ/năm + bộ phận".

### Field HR chuẩn

```xml
<field name="ky" type="Decimal" dataFormatString="#0" allowNulls="false" aliasName="fromPeriod" defaultValue="(new Date()).getMonth() + 1;">
	<header v="Kỳ" e="Period"></header>
	<items style="Numeric"></items>
</field>
<field name="nam" type="Decimal" dataFormatString="###0" allowNulls="false">
	<header v="Năm" e="Year"></header>
	<items style="Numeric"></items>
</field>

<field name="ma_bp" onDemand="true">
	<header v="Bộ phận" e="Department"></header>
	<items style="AutoComplete" controller="hrDepartment" reference="ten_bp%l" key="(@@admin = 1 or ma_bp in (select a.ma_bp from hrbp a, @@sysDatabaseName..hrquyenbp b where dbo.ff_Inlist(a.bp_ref, b.r_access2) = 1 and b.user_id = @@userID)) and status = '1'" check="@@admin = 1 or ma_bp in (select a.ma_bp from hrbp a, @@sysDatabaseName..hrquyenbp b where dbo.ff_Inlist(a.bp_ref, b.r_access2) = 1 and b.user_id = @@userID)"/>
</field>
<field name="ten_bp%l" readOnly="true" external="true">
	<header v="" e=""></header>
</field>
<field name="ma_nv" onDemand="true">
	<header v="Nhân viên" e="Employee"></header>
	<items style="AutoComplete" controller="hrEmployee" reference="ten_nv" key="(@@admin = 1 or bo_phan in (select a.ma_bp from hrbp a, @@sysDatabaseName..hrquyenbp b where dbo.ff_Inlist(a.bp_ref, b.r_access2) = 1 and b.user_id = @@userID)) and status = '1'" check="@@admin = 1 or bo_phan in (select a.ma_bp from hrbp a, @@sysDatabaseName..hrquyenbp b where dbo.ff_Inlist(a.bp_ref, b.r_access2) = 1 and b.user_id = @@userID)"/>
</field>
<field name="ten_nv" readOnly="true" external="true">
	<header v="" e=""></header>
</field>

<field name="nh_bp1" onDemand="true">
	<header v="Nhóm bộ phận" e="Department Group"></header>
	<items style="AutoComplete" controller="hrDepartmentGroup" reference="ten_nhbp1%l" key="status='1' and loai_nh=1" check="loai_nh=1"/>
</field>
<field name="ten_nhbp1%l" readOnly="true" external="true" defaultValue="''">
	<header v="" e=""></header>
</field>
<field name="nh_bp2" onDemand="true">
	<header v="" e=""></header>
	<items style="AutoComplete" controller="hrDepartmentGroup" reference="ten_nhbp2%l" key="status='1' and loai_nh=2" check="loai_nh=2"/>
</field>
<field name="ten_nhbp2%l" readOnly="true" external="true" defaultValue="''">
	<header v="" e=""></header>
</field>
<field name="nh_bp3" onDemand="true">
	<header v="" e=""></header>
	<items style="AutoComplete" controller="hrDepartmentGroup" reference="ten_nhbp3%l" key="status='1' and loai_nh=3" check="loai_nh=3"/>
</field>
<field name="ten_nhbp3%l" readOnly="true" external="true" defaultValue="''">
	<header v="" e=""></header>
</field>

<field name="nh_nv1" onDemand="true">
	<header v="Nhóm nhân viên 1" e="Employee Group 1"></header>
	<footer v="Nhóm nhân viên" e="Employee Group"/>
	<items style="AutoComplete" controller="hrEmployeeGroup" reference="ten_nh_nv1%l" key="status = '1' and loai_nh = 1" check="loai_nh = 1"/>
</field>
<field name="ten_nh_nv1%l" readOnly="true" external="true" defaultValue="''">
	<header v="" e=""></header>
</field>
<field name="nh_nv2" onDemand="true">
	<header v="Nhóm nhân viên 2" e="Employee Group 2"></header>
	<items style="AutoComplete" controller="hrEmployeeGroup" reference="ten_nh_nv2%l" key="status = '1' and loai_nh = 2" check="loai_nh = 2"/>
</field>
<field name="ten_nh_nv2%l" readOnly="true" external="true" defaultValue="''">
	<header v="" e=""></header>
</field>
<field name="nh_nv3" onDemand="true">
	<header v="Nhóm nhân viên 3" e="Employee Group 3"></header>
	<items style="AutoComplete" controller="hrEmployeeGroup" reference="ten_nh_nv3%l" key="status = '1' and loai_nh = 3" check="loai_nh = 3"/>
</field>
<field name="ten_nh_nv3%l" readOnly="true" external="true" defaultValue="''">
	<header v="" e=""></header>
</field>
```

### Layout chuẩn

Khi sử dụng toàn bộ bộ field HR trên, có thể dùng trực tiếp layout sau:

```xml
<item value="120, 30, 10, 60, 100, 100, 130, 0, 0, 0"/>
<item value="110-------: [ky].Label, [ky]"/>
<item value="110-------: [nam].Label, [nam]"/>
<item value="11001001--: [ma_bp].Label, [ma_bp], [ten_bp%l], [ten_nhbp1%l]"/>
<item value="11001000--: [ma_nv].Label, [ma_nv], [ten_nv]"/>
<item value="110011-1--: [nh_bp1].Label, [nh_bp1], [nh_bp2], [nh_bp3], [ten_nhbp2%l]"/>
<item value="110011-111: [nh_nv1].Description, [nh_nv1], [nh_nv2], [nh_nv3], [ten_nh_nv1%l], [ten_nh_nv2%l], [ten_nh_nv3%l]"/>
```

Nếu chỉ sử dụng một phần bộ lọc HR thì phải điều chỉnh lại layout và `LineCounter` theo số dòng thực tế.

### Processing / Store cho HR

Khi các field HR được sử dụng, truyền các field điều kiện thực sự cần cho Store.

Ví dụ:

```xml
<command event="Processing">
	<text><![CDATA[
select @ky as ky, @nam as nam

exec hs_rpt{Controller}
	@ky,
	@nam,
	@ma_bp,
	@ma_nv,
	@nh_bp1,
	@nh_bp2,
	@nh_bp3,
	@nh_nv1,
	@nh_nv2,
	@nh_nv3,
	@@language,
	@@userID,
	@@admin
]]>&DynamicReportFields;
		&ReportMarginProcessing;
		&ReportSign.Filter.Query;
	</text>
</command>
```

> Thứ tự và số lượng tham số Store phải theo đúng nghiệp vụ thực tế. Không tự động truyền các field tên (`ten_bp`, `ten_nv`, `ten_nh...`) vì đây là các field hiển thị `external`, trừ khi Store thực sự yêu cầu.

### Lưu ý về phân quyền HR

`ma_bp` và `ma_nv` đã bao gồm điều kiện phân quyền bộ phận dựa trên:

- `@@admin`
- `@@userID`
- `@@sysDatabaseName..hrquyenbp`
- `dbo.ff_Inlist(...)`

Vì vậy khi sử dụng mẫu chuẩn này:

**Không thay `key` hoặc `check` bằng `status='1'` đơn giản và không tra lookup khác để thay thế**, trừ khi prompt yêu cầu thay đổi cơ chế phân quyền.

---

## `<views>` — Bố cục Filter

```xml
<views>
  <view id="Dir" height="&TabHeightFomula;">
    <!-- Dòng column widths -->
    <item value="120, 30, 70, 100, 100, 130"/>

    <!-- Các field tab chính (categoryIndex mặc định hoặc 1) -->
    <item value="1101--: [tu_ngay].Description, [tu_ngay], [den_ngay]"/>
    <item value="11011: [so_ct1].Label, [so_ct1], [so_ct2], [maxLength]"/>
    <item value="11000-: [tk].Label, [tk]"/>
    <item value="110100: [ma_kh].Label, [ma_kh], [ten_kh%l]"/>
    <!-- ... thêm field ... -->
    <item value="11000-: [mau_bc].Label, [mau_bc]"/>

    <!-- Entity chữ ký + canh chỉnh (đặt sau field cuối) -->
    &ReportSign.Filter.Views;
    &ReportMarginView;

    <!-- Khai báo Tab -->
    <categories>
      <category index="1" columns="120, 25, 75, 100, 100, 130">
        <header v="Thông tin chung" e="General"/>
      </category>
      <category index="2" columns="120, 30, 70, 100, 100, 130">
        <header v="Lựa chọn" e="Option"/>
      </category>
      <category index="3" columns="120, 30, 70, 100, 100, 130">
        <header v="Khác" e="Other"/>
      </category>
      &ReportSign.Filter.Categories;
    </categories>
  </view>
</views>
```

### Cú pháp layout `<item>`

- Cú pháp giống Dir danh mục: `"WidthFlags: [field]..."` (xem `SPEC_dir.md` → phần Views)
- `categoryIndex` trên field → xác định field thuộc tab nào
- Mặc định có **ít nhất 1 tab** "Thông tin chung"
- Tab chữ ký và canh chỉnh nằm cuối danh sách categories

---

## `<commands>` — Các sự kiện SQL

### Initialize — Khởi tạo

```xml
<command event="Initialize">
  <text>
    <![CDATA[
declare @message nvarchar(4000)
select @message = 'init$VoucherFilter$(this);'
]]>&ReportSign.Filter.Initialize;<![CDATA[
select @message as message
return
]]>
  </text>
</command>
```

### Showing — Ẩn/hiện field động (tùy chọn)

```xml
<command event="Showing">
  <text>
    <![CDATA[
declare @$hideFields varchar(4000)
select @$hideFields = ''
select @$hidefields = @$hidefields
  + case when @$hidefields = '' then '' else ',' end + name
  from [@@sysDatabaseName]..sysfields
  where controller = ']]>&Controller;<![CDATA[' and c = 1
select 'this._$hiddenFields="' + replace(replace(rtrim(@$hideFields), '\', '\\'), '"', '\"') + '";' as message
]]>
  </text>
</command>
```

> Dùng khi báo cáo cho phép admin ẩn/hiện field từ bảng `sysfields`.

### Processing — ⭐ Sự kiện quan trọng nhất

Đây là điểm khác biệt lớn nhất so với Filter danh mục. Processing chứa SQL gọi Store trả dữ liệu.

**Cấu trúc chuẩn:**

```xml
<command event="Processing">
  <text>
    <![CDATA[
-- Bảng 1: thông tin điều kiện lọc (cho mẫu in)
select cast(@tu_ngay as smalldatetime) as date_from,
       cast(@den_ngay as smalldatetime) as date_to

-- Bảng 2+: gọi Store lấy dữ liệu chính
exec rs_rpt{Controller} @tu_ngay, @den_ngay, @param1, @param2, ..., @maxLength, @@language, @@userID, @@admin
]]>&DynamicReportFields;{&ReferenceParameters;}
    &ReportMarginProcessing;
    &ReportSign.Filter.Query;
  </text>
</command>
```

**Giải thích:**
1. **SELECT thông tin lọc** → bảng 1 (index 0) — cho mẫu in lấy header
2. **EXEC Store** → bảng 2+ (index 1+) — dữ liệu chính hiển thị ở Grid
3. **`&DynamicReportFields;`** — tham số mở rộng cho dynamic report
4. **`&ReportMarginProcessing;`** — SQL lấy thông tin canh chỉnh
5. **`&ReportSign.Filter.Query;`** — SQL lấy thông tin chữ ký

### Entity commands chuẩn

```xml
&XMLWhenFilterLoading;
&XMLWhenFilterClosing;
&XMLWhenFilterQuerying;
```

---

## `<script>` — JavaScript

```xml
<script>
  <text>
    &OutlineEntry;
    &JavascriptReportFilter;
    &ScriptFilterInit;
    <![CDATA[
function onChange$VoucherFilter$Tab(f, e) {
  f.parentForm.focusWhenTabChanged(['{field_tab_1}', '{field_tab_2}', '{field_tab_3}']);
}

function active$VoucherFilter$(f) {
  f.add_onResponseComplete(on$Filter$ResponseComplete);
  f._tabContainer.add_activeTabChanged(onChange$VoucherFilter$Tab);
  f._tabContainer._loaded = true;
  // changeLookupReadonly nếu có field Lookup cần readonly theo quyền
  changeLookupReadonly(f, 'ma_dvcs');
  var o = f.getItem('maxLength');
  o.value = o.maxLength;
  ]]>&ReportSign.Filter.Active;<![CDATA[
}

/* <flatten type="Javascript"> */
function close$VoucherFilter$(f) {
  try {f.remove_onResponseComplete(on$Filter$ResponseComplete);} catch (ex) {}
  try {f._tabContainer.remove_activeTabChanged(onChange$VoucherFilter$Tab);} catch (ex) {}
}

function on$Filter$ResponseComplete(sender, e) {
  var f = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'Checking':
      var g = f.grid;
      var x = f.getItemValue('mau_bc'),
          dFrom = f.getItem('tu_ngay').value,
          dTo = f.getItem('den_ngay').value;
      // Ẩn/hiện cột theo mẫu báo cáo
      f.grid._hiddenFields = [
        {Fields: '{cột_ngoại_tệ}', Value: (x == '10')}
      ];
      // Gán subTitle
      g._alterTitle = [null, [['%d1', dFrom, true], ['%d2', dTo, true]]];
      // Xóa filter cũ
      remove$GridReport$Filter(g);
      break;
    default:
      break;
  }
}
/* </flatten> */
]]>
  </text>
</script>
```

### Giải thích các function JS

| Function | Mô tả |
|---|---|
| `onChange$VoucherFilter$Tab` | Focus vào field đầu tiên khi chuyển tab |
| `active$VoucherFilter$` | Gọi khi Filter active — gắn event, khởi tạo |
| `close$VoucherFilter$` | Cleanup khi đóng Filter |
| `on$Filter$ResponseComplete` | Xử lý sau khi Filter submit — ẩn/hiện cột, set subTitle |

### Xử lý trong Checking

- **`_hiddenFields`**: mảng object `{Fields: '...', Value: boolean}` — ẩn cột trên Grid
- **`_alterTitle`**: gán giá trị cho placeholder `%d1`, `%d2` trong subTitle
- **`remove$GridReport$Filter`**: xóa filter Grid cũ trước khi apply mới

---

## Entity LineCounter — Cách đếm

`LineCounter` = số dòng `<item>` trong view của tab chính (KHÔNG tính dòng column widths, KHÔNG tính tab chữ ký/canh chỉnh).

Ví dụ: tab chính có 12 dòng field → `<!ENTITY LineCounter "12">`.

---

## Ví dụ tối thiểu (báo cáo đơn giản)

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE dir [
  <!ENTITY XMLWhenFilterLoading SYSTEM "..\Include\XML\WhenFilterLoading.xml">
  <!ENTITY XMLWhenFilterClosing SYSTEM "..\Include\XML\WhenFilterClosing.xml">
  <!ENTITY ScriptFilterInit SYSTEM "..\Include\Javascript\FilterInit.txt">
  <!ENTITY OutlineCss SYSTEM "..\Include\Javascript\OutlineCss.txt">
  <!ENTITY OutlineEntry SYSTEM "..\Include\Javascript\OutlineEntry.txt">
  <!ENTITY OnSelectionOutline SYSTEM "..\Include\Javascript\OnSelectionOutline.txt">
  <!ENTITY Controller "rptMyReport">
  <!ENTITY DynamicReportFields ",'&Controller;', '#$query', '@@sysDatabaseName'">
  <!ENTITY JavascriptReportFilter SYSTEM "..\Include\Javascript\ReportFilter.txt">

  <!ENTITY LineCounter "4">
  <!ENTITY ExtensionCounter "1">
  <!ENTITY ReportMarginCategoryIndex "&ReportSign.Filter.CategoryIndex;">
  <!ENTITY % ReportMargin SYSTEM "..\Include\ReportMargin.ent">
  %ReportMargin;
  <!ENTITY % TabHeightFomula SYSTEM "..\Include\TabHeightFomula.ent">
  %TabHeightFomula;
]>

<dir id="1" type="Report" cache="true" xmlns="urn:schemas-fast-com:data-dir">
  <title v="Điều kiện lọc" e="Filter Condition"/>
  <fields>
    <field name="tu_ngay" type="DateTime" dataFormatString="@datetimeFormat"
           allowNulls="false" aliasName="fromDate" defaultValue="new Date()">
      <header v="Từ ngày" e="Date from"/>
      <footer v="Từ/đến ngày" e="Date from/to"/>
    </field>
    <field name="den_ngay" type="DateTime" dataFormatString="@datetimeFormat"
           allowNulls="false" aliasName="toDate" defaultValue="new Date()">
      <header v="Đến ngày" e="Date to"/>
    </field>
    <field name="ma_dvcs">
      <header v="Đơn vị" e="Unit"/>
      <items style="Lookup" controller="Unit" key="status = '1'" check="1 = 1"/>
    </field>
    <field name="mau_bc">
      <header v="Mẫu báo cáo" e="Report Form"/>
      <clientScript>&OnSelectionOutline;</clientScript>
      <items style="DropDownList">
        <item value="10"><text v="Mẫu chuẩn" e="Standard Form"/></item>
        <item value="20"><text v="Mẫu ngoại tệ" e="FC Form"/></item>
      </items>
    </field>

    <field name="maxLength" type="Int16" readOnly="true" hidden="true" maxLength="-100">
      <header v="" e=""/>
    </field>
    &ReportSign.Filter.Fields;
    &ReportMarginFieldExtend;
  </fields>
  <views>
    <view id="Dir" height="&TabHeightFomula;">
      <item value="120, 30, 70, 100, 100, 130"/>
      <item value="1101--: [tu_ngay].Description, [tu_ngay], [den_ngay]"/>
      <item value="11000-: [ma_dvcs].Label, [ma_dvcs]"/>
      <item value="11000-: [mau_bc].Label, [mau_bc]"/>
      <item value="11011: [maxLength]"/>
      &ReportSign.Filter.Views;
      &ReportMarginView;
      <categories>
        <category index="1" columns="120, 25, 75, 100, 100, 130">
          <header v="Thông tin chung" e="General"/>
        </category>
        &ReportSign.Filter.Categories;
      </categories>
    </view>
  </views>
  <commands>
    <command event="Initialize">
      <text><![CDATA[
declare @message nvarchar(4000)
select @message = 'init$VoucherFilter$(this);'
]]>&ReportSign.Filter.Initialize;<![CDATA[
select @message as message
return
]]>
      </text>
    </command>
    &XMLWhenFilterLoading;
    &XMLWhenFilterClosing;
    <command event="Processing">
      <text><![CDATA[
select cast(@tu_ngay as smalldatetime) as date_from,
       cast(@den_ngay as smalldatetime) as date_to
exec rs_rptMyReport @@language, @tu_ngay, @den_ngay, @ma_dvcs,
  @maxLength, @@userID, @@admin]]>&DynamicReportFields;
        &ReportMarginProcessing;
        &ReportSign.Filter.Query;
      </text>
    </command>
  </commands>
  <script>
    <text>
      &OutlineEntry;
      &JavascriptReportFilter;
      &ScriptFilterInit;
      <![CDATA[
function onChange$VoucherFilter$Tab(f, e) {
  f.parentForm.focusWhenTabChanged(['tu_ngay']);
}
function active$VoucherFilter$(f) {
  f.add_onResponseComplete(on$Filter$ResponseComplete);
  f._tabContainer.add_activeTabChanged(onChange$VoucherFilter$Tab);
  f._tabContainer._loaded = true;
  changeLookupReadonly(f, 'ma_dvcs');
  var o = f.getItem('maxLength');
  o.value = o.maxLength;
  ]]>&ReportSign.Filter.Active;<![CDATA[
}
/* <flatten type="Javascript"> */
function close$VoucherFilter$(f) {
  try {f.remove_onResponseComplete(on$Filter$ResponseComplete);} catch (ex) {}
  try {f._tabContainer.remove_activeTabChanged(onChange$VoucherFilter$Tab);} catch (ex) {}
}
function on$Filter$ResponseComplete(sender, e) {
  var f = e.object, context = e.type.Context;
  switch (context) {
    case 'Checking':
      var g = f.grid, x = f.getItemValue('mau_bc'),
          dFrom = f.getItem('tu_ngay').value,
          dTo = f.getItem('den_ngay').value;
      g._alterTitle = [null, [['%d1', dFrom, true], ['%d2', dTo, true]]];
      remove$GridReport$Filter(g);
      break;
    default:
      break;
  }
}
/* </flatten> */
]]>
    </text>
  </script>
  &OutlineCss;
</dir>
```

---

## Lưu ý quan trọng

1. **`id` trên `<dir>`** phải khớp với index bảng dữ liệu cần hiển thị ở Grid. Thường `id="1"` (bảng thứ 2) vì bảng 1 (index 0) chứa thông tin điều kiện lọc
2. **Processing bắt buộc**: đây là nơi duy nhất gọi Store trả dữ liệu
3. **Thứ tự trong Processing**: select thông tin lọc TRƯỚC, exec Store SAU
4. **Entity chữ ký/canh chỉnh**: mặc định có. Bỏ chỉ khi prompt nói "không cần chữ ký"
5. **`&DynamicReportFields;`**: nối tiếp ngay sau tham số cuối của exec Store, không xuống dòng
6. **LineCounter**: đếm chính xác số `<item>` layout trong tab chính
