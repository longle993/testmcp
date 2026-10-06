# [SPEC_import] — Tính năng Import Excel (Tải dữ liệu hàng loạt)

> Dùng khi: danh mục cần tính năng nhập liệu hàng loạt từ file Excel.
> Liên quan đến **3 thành phần**: Grid (toolbar + JS), Filter (upload form), Upload (xử lý import).

---

## Tổng quan luồng hoạt động

```
1. Grid hiển thị toolbar có nút "Import Data" + "Download Template"
2. User nhấn "Download" → Loading tạo ticket → JS download file Excel template trống
3. User điền dữ liệu vào Excel → nhấn "Import Data"
4. JS kiểm tra _authorize → mở Filter ImportForm (showForm)
5. User chọn file + kiểu sao chép → submit
6. Upload.xml xử lý: đọc file → validate từng cột → INSERT/UPDATE vào DB
7. Kết quả hiển thị (thành công hoặc danh sách lỗi)
```

---

## Quy ước tên file

| Thành phần | Thư mục gốc | Tên khi share |
|---|---|---|
| Upload XML | `Controllers/Templates/Upload/{Controller}.xml` | `upload_{Controller}.xml` |
| Filter Import Form | `Controllers/Filter/{Controller}Import.xml` | `filter_{Controller}Import.xml` |
| Grid (bổ sung toolbar) | `Controllers/Grid/{Controller}.xml` | `grid_{Controller}.xml` |

---

## Phần 1 — Grid: Toolbar + Script + CSS

### DOCTYPE — ENTITY cần thêm

```xml
<!DOCTYPE grid [
  <!ENTITY DowloadScript SYSTEM "..\Include\Javascript\DownloadScript.txt">

  <!ENTITY TransferID "{Controller}">
  <!ENTITY FastBusiness.Encryption.Begin "">
  <!ENTITY CreateTicket "declare @ticket varchar(32)
select @ticket = lower(replace(newid(),'-',''))
insert into @@sysDatabaseName..ticket values(@ticket, @@userID, '&TransferID;', '', '', '@@appDatabaseName', getdate());">
  <!ENTITY FastBusiness.Encryption.End "">

  <!ENTITY % ExportImportTemplate SYSTEM "..\Include\ExportImportTemplate.ent">
  %ExportImportTemplate;
  <!ENTITY ExportImportTemplate.UploadController "controller=&TransferID;&amp;form=&TransferID;">

  <!-- ... các ENTITY khác của Grid nếu có ... -->
]>
```

### `<commands>` — Loading tạo ticket + kiểm tra quyền

```xml
<commands>
  <command event="Loading">
    <text>
      &CreateTicket;<![CDATA[
select 'this._authorize = ' + rtrim(@@sysDatabaseName.dbo.FastBusiness$System$GetAuthorize(@@admin, @@userID, ']]>&TransferID;<![CDATA[', 'New')) + ';this._key = ''' + @ticket + ''';load$Grid(this);' as message
return
]]>
    </text>
  </command>
  <command event="Closing">
    <text><![CDATA[select 'dispose$Grid(this);' as message
return]]></text>
  </command>
</commands>
```

> Loading gán `_authorize` (0/1 — quyền tạo mới) và `_key` (ticket bảo mật) vào grid object.

### `<script>` — ExecuteCommand pattern

```xml
<script>
  <text>
    <![CDATA[/* <flatten type="Javascript"> */
function load$Grid(g) {
  g.add_onResponseComplete(on$Grid{Controller}$ResponseComplete);
  g.add_commandEvent(on$Grid{Controller}$ExecuteCommand);
}
function dispose$Grid(g) {
  try {g.remove_commandEvent(on$Grid{Controller}$ExecuteCommand);} catch (ex) {}
  try {g.remove_onResponseComplete(on$Grid{Controller}$ResponseComplete);} catch (ex) {}
}
/* </flatten> */
function on$Grid{Controller}$ExecuteCommand(sender, e) {
  var action = e.type.Action, g = sender;
  switch (action) {
    case 'ImportData':
      show$Form(g, '{Controller}Import');
      break;
    case 'Download':
      ]]>&UserDefinedDownload;<![CDATA[
      break;
  }
}
/* <flatten type="Javascript"> */
function on$Grid{Controller}$ResponseComplete(sender, e) {
  var g = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'Download':
      g._key = result[0].Value;
      break;
  }
}
function show$Form(g, c) {
  (g._authorize == 1) ? g.showForm(c) : $message.show($df.getResources(g._language, "Message.NotAccess"));
}
/* </flatten> */]]>
    &DowloadScript;
  </text>
</script>
```

**Giải thích pattern:**

| Hàm | Vai trò |
|---|---|
| `load$Grid` | Đăng ký 2 event: `commandEvent` (bắt toolbar click) + `onResponseComplete` |
| `on$Grid{Controller}$ExecuteCommand` | Xử lý custom toolbar: `ImportData` → mở form import, `Download` → tải template |
| `on$Grid{Controller}$ResponseComplete` | Nhận ticket mới sau khi download |
| `show$Form` | Kiểm tra quyền trước khi mở form import |

### `<css>` — Icon cho toolbar button

```xml
<css>
  <text><![CDATA[
div.ImportData{background-image:url(../images/Upload.png);background-repeat:no-repeat;background-position:0 0;}
div.Download{background-image:url(../Images/Download.png);background-repeat:no-repeat;background-position:0 0;}
div.ImportDataOverGreen, div.DownloadOverGreen{background-position:0 -22px;}
]]></text>
</css>
```

> CSS class tự đặt theo `command` name. `OverGreen` = hover state.

### `<response>` — Lấy ticket cho download

```xml
<response>
  <action id="Download">
    <text>
      &CreateTicket;<![CDATA[
select @ticket as value
return
]]>
    </text>
  </action>
</response>
```

### `<toolbar>` — Nút Import + Download

```xml
<toolbar>
  <button command="New"><title v="Toolbar.New" e="Toolbar.New"/></button>
  <button command="Edit"><title v="Toolbar.Edit" e="Toolbar.Edit"/></button>
  <button command="Delete"><title v="Toolbar.Delete" e="Toolbar.Delete"/></button>
  <button command="View"><title v="Toolbar.View" e="Toolbar.View"/></button>
  <button command="Export"><title v="Toolbar.Export" e="Toolbar.Export"/></button>
  <button command="-"><title v="-" e="-"/></button>
  <!-- Nút import + download — command name = CSS class -->
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

> `command="ImportData"` và `command="Download"` là custom command — framework gọi `ExecuteCommand` handler, không tự xử lý.

---

## Phần 2 — Filter Import Form (`{Controller}Import.xml`)

Form popup để user chọn file Excel và tùy chọn trước khi upload.

### DOCTYPE

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE dir [
  <!ENTITY Identity "{Controller}ImportForm">

  <!ENTITY UploadButtonCss    SYSTEM "..\Include\XML\ImageUploadForm.txt">
  <!ENTITY UploadCreateTicket SYSTEM "..\Include\XML\UploadCreateTicket.txt">
  <!ENTITY UploadField        SYSTEM "..\Include\XML\UploadField.txt">
  <!ENTITY UploadCommand      SYSTEM "..\Include\Command\UploadCommand.txt">
  <!ENTITY UploadScript       SYSTEM "..\Include\Javascript\UploadScript.txt">

  <!ENTITY % ImportErrorMode  SYSTEM "..\Include\ImportErrorMode.ent">
  %ImportErrorMode;
]>
```

> `Identity` = `{Controller}ImportForm` — dùng xuyên suốt trong JS để tạo tên hàm, phải đặt đúng.

### `<dir>`, `<fields>`, `<views>`

```xml
<dir table="{bảng}" code="{cột_mã}" order="{cột_mã}"
     xmlns="urn:schemas-fast-com:data-dir">
  <title v="Lấy dữ liệu từ tệp" e="Import Data From File"></title>

  <fields>
    &UploadField;
    <field name="type" dataFormatString="0, 1" clientDefault="0" align="right">
      <header v="Kiểu sao chép" e="Type"></header>
      <footer v="1 - Chép đè, 0 - Không" e="1 - Overwrite, 0 - No"></footer>
      <items style="Mask"/>
    </field>
    <field name="ticket" hidden="true" readOnly="true">
      <header v="" e=""></header>
    </field>
    &FilterFormModeField;
  </fields>

  <views>
    <view id="Dir">
      <item value="120, 30, 70, 100, 100, 100, 30"/>
      <item value="1100010: [upload].Label, [upload], [upload].Description"/>
      <item value="1110001: [type].Label, [type], [type].Description, [ticket]"/>
      &FilterFormModeView;
    </view>
  </views>
```

Layout 7 cột (`120, 30, 70, 100, 100, 100, 30`):
- `1100010`: Label + Input(merge col2-3) + trống(col4-6) + Description(col7)
- `1110001`: Label + col phụ + Input(flag) + Description(merge col4-6) + field ẩn(col7)

### `<commands>`

```xml
  <commands>
    &UploadCommand;

    <command event="Checking">
      <text><![CDATA[
var f = this, g = f.grid, k = f.getItem('ticket').value, importMemvars = [];
var fileName = get$]]>&Identity;<![CDATA[FileName($get('fileupload').value), allowExt = 'xlsx';
var err = (this._language == 'v' ? 'Định dạng tệp không đúng.' : 'Invalid file type.');
if (fileName != '') {
  var ext = fileName.split('.').pop();
  if (allowExt.indexOf(ext.toLowerCase()) < 0) {
    f._checked = false;
    $message.show(err);
  }
  else {
    init$]]>&Identity;<![CDATA[IFrame(f);
    set$]]>&Identity;<![CDATA[Memvar(f, importMemvars, 'type');
    ]]>&FilterFormModeVar;<![CDATA[
    var form = $get('uploadForm'),
        query = {Language: f._language, Controller: g._controller,
                 Key: k, Memvars: importMemvars, Time: new Date().getTime().toString()};
    $get('query', form).value = Sys.Serialization.JavaScriptSerializer.serialize(query);
    form.action = $func.resolveClientUrl('~/AppHandler/Import.ashx', g._baseUrl);
    form.target = g._controller + '_IFrame';
    form.submit();
    f.request('GetTicket', 'GetTicket', []);
    f._checked = false;
    ]]>&FilterFormModeEndSubmit;<![CDATA[
  }
}
]]></text>
    </command>
  </commands>
```

### `<script>` và `<response>`

```xml
  <script>
    <text>
      &UploadScript;
      &FilterFormModeScript;
      <![CDATA[
function init$]]>&Identity;<![CDATA[(f) {
  var g = f.grid;
  document.body._form = f;
  $get('fileupload').value = '';
  f.getItem('upload').field.AllowNulls = false;
  $get(f.get_id() + '_form_upload').style.color = 'grey';
  f.request('GetTicket', 'GetTicket', []);
}
function on$]]>&Identity;<![CDATA[$ResponseComplete(f, e) {
  var f = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'GetTicket':
      f.setItemValue('ticket', result[0].Value);
      break;
  }
}
]]>
    </text>
  </script>

  <response>
    <action id="GetTicket">
      <text>
        &UploadCreateTicket;<![CDATA[
select @ticket as value
return
]]>
      </text>
    </action>
  </response>

  &UploadButtonCss;
</dir>
```

---

## Phần 3 — Upload XML (`upload_{Controller}.xml`)

File khai báo mapping Excel → DB và SQL xử lý import.

> 📁 **Vị trí thực tế**: `Web/App_Data/Controllers/Templates/Upload/{Controller}.xml` (2 cấp dưới `Controllers/`, nên ENTITY dùng `..\..\Include\` thay vì `..\Include\`).

### DOCTYPE — 2 biến thể

**Biến thể đơn giản** (như ItemGroup):

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE import [
  <!ENTITY IrregularValue SYSTEM "Include\Irregular.txt">

  <!ENTITY % ImportErrorMode SYSTEM "..\..\Include\ImportErrorMode.ent">
  %ImportErrorMode;

  <!ENTITY Error "
if @r is not null begin
  select '' as message, @field as field, @r as record
  return
end
">
  <!ENTITY Irregular "
if @r is not null begin
  select @irregular as message, @field as field, @r as record
  return
end
">
  <!ENTITY Duplicate "
if @r is not null begin
  select @duplicate as message, @field as field, @r as record
  return
end
">
  <!ENTITY Checking  "@@checking">
  <!ENTITY Inserting "@@inserting">
  <!ENTITY Updating  "@@updating">

  <!ENTITY % ExportImportTemplate SYSTEM "..\..\Include\ExportImportTemplate.ent">
  %ExportImportTemplate;
  <!ENTITY ExportQueryStaticFile
    "select case when @@language = 'V' then '{TênFileVN}' else '{NameFileEN}' end as file_name">

  <!ENTITY Controller "{Controller}">
  <!ENTITY % ListEditLog SYSTEM "..\..\Include\ListEditLog.ent">
  %ListEditLog;
]>
```

**Biến thể có Tiny.External** (bảng có nhiều field FK — như Job):

Thêm vào DOCTYPE (đặt trước `%ExportImportTemplate;`):

```xml
  <!ENTITY % Tiny.External SYSTEM "..\..\Include\Tiny.External.ent">
  %Tiny.External;
  %Tiny.External.{Controller};
```

### `<setting>`

```xml
<import xmlns="urn:schemas-fast-com:data-import">
  <setting>
    <startRow value="6"/>
    <maxFileSize value="10"/>
    <uploadTimeOut value="120"/>
    <importRecordTimeout value="1740"/>
    <allowFileExtension value="^(.XLSX)$"/>
    <onProcessFail     value="parent.on${Controller}ImportForm$Fail(this.frameElement)"/>
    <onProcessComplete value="parent.on${Controller}ImportForm$Complete(this.frameElement)"/>
    <uploadContentType value="application/ms-excel;application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"/>
    <baseTable value="{tên_bảng}"/>
    <table value="{tên_bảng}" alias="a"/>
    <temporary value="#k" alias="b"/>
    &UploadModeProcess;
  </setting>
```

> `startRow="6"` — header Excel ở dòng 5, data bắt đầu dòng 6.

### `<query>`

```xml
  <query>
    <command>
      <text>
        &ExportImportTemplateQuery;
      </text>
    </command>
  </query>
```

### `<fields identity="true">` — Mapping cột Excel → DB ⭐

Khai báo từng field: cột Excel nào map vào cột DB nào, kiểu dữ liệu, validate.

```xml
  <fields identity="true" name="stt">
    <field name="{tên_cột_db}" column="{chữ_cột_Excel}"
           [isPrimaryKey="true"]
           [allowNulls="false"]
           [upperCase="true"]
           [type="Decimal|DateTime|Int"]
           [defaultValue="{giá_trị}"]
           [check="{điều_kiện_SQL}"]
           [errorCode="00001|00002"]
           [updateValue="None"]
    />

    <!-- Audit fields — KHÔNG lấy từ Excel -->
    <field name="status"    column="None" insertValue="'1'"        updateValue="None"/>
    <field name="datetime0" column="None" type="DateTime" insertValue="getdate()" updateValue="None"/>
    <field name="datetime2" column="None" type="DateTime" insertValue="getdate()" updateValue="getdate()"/>
    <field name="user_id0"  column="None" type="Int"      insertValue="@@userID"  updateValue="None"/>
    <field name="user_id2"  column="None" type="Int"      insertValue="@@userID"  updateValue="@@userID"/>
  </fields>
```

#### Thuộc tính field

| Thuộc tính | Mô tả |
|---|---|
| `name` | Tên cột DB |
| `column` | Cột Excel (`A`, `B`, ...) hoặc `None` (không lấy từ Excel) |
| `isPrimaryKey="true"` | Khóa chính — xác định dòng khi UPDATE |
| `allowNulls="false"` | Bắt buộc nhập |
| `upperCase="true"` | Tự động viết hoa |
| `type` | `Decimal`, `DateTime`, `Int` (mặc định = string) |
| `defaultValue` | Giá trị mặc định khi ô trống |
| `check` | Điều kiện SQL — nếu TRUE → báo lỗi |
| `errorCode` | `"00001"` = giá trị không hợp lệ |
| `updateValue="None"` | Không cập nhật khi UPDATE |
| `insertValue` | Giá trị INSERT (cho field `column="None"`) |

#### Ví dụ — Job (`dmvv`)

```xml
  <fields identity="true" name="stt">
    <field name="ma_vv"    column="A" isPrimaryKey="true" allowNulls="false" upperCase="true" updateValue="None"/>
    <field name="ten_vv"   column="B" allowNulls="false"/>
    <field name="ten_vv2"  column="C"/>
    <field name="ngay_vv"  column="D" type="DateTime"/>
    <field name="so_vv"    column="E"/>
    <field name="vv_sd_pslk" column="F" type="Decimal" defaultValue="0"
           check="vv_sd_pslk not in ('0', '1')" errorCode="00001"/>
    <field name="ma_nt"    column="G" allowNulls="false" upperCase="true"
           check="ma_nt not in (select ma_nt from dmnt)" errorCode="00001"/>
    <field name="tien_nt"  column="H" type="Decimal"/>
    <field name="tien"     column="I" type="Decimal"/>
    <field name="ngay_vv1" column="J" type="DateTime"/>
    <field name="ngay_vv2" column="K" type="DateTime"/>
    <field name="ma_vv_me" column="L" upperCase="true"
           check="(ma_vv_me &lt;&gt; '' and ma_vv_me not in (select ma_vv from dmvv)) or ma_vv_me = ma_vv" errorCode="00001"/>
    <field name="ma_kh"    column="M" upperCase="true"
           check="ma_kh &lt;&gt; '' and ma_kh not in (select ma_kh from dmkh)" errorCode="00001"/>
    <field name="ma_nvbh"  column="N" upperCase="true"
           check="ma_nvbh &lt;&gt; '' and ma_nvbh not in (select ma_nvbh from dmnvbh)" errorCode="00001"/>
    <field name="ma_bp"    column="O" upperCase="true"
           check="ma_bp &lt;&gt; '' and ma_bp not in (select ma_bp from dmbp)" errorCode="00001"/>
    <field name="nh_vv1"   column="P" upperCase="true"
           check="nh_vv1 &lt;&gt; '' and nh_vv1 not in (select ma_nh from dmnhvv where loai_nh = 1)" errorCode="00001"/>
    <field name="nh_vv2"   column="Q" upperCase="true"
           check="nh_vv2 &lt;&gt; '' and nh_vv2 not in (select ma_nh from dmnhvv where loai_nh = 2)" errorCode="00001"/>
    <field name="nh_vv3"   column="R" upperCase="true"
           check="nh_vv3 &lt;&gt; '' and nh_vv3 not in (select ma_nh from dmnhvv where loai_nh = 3)" errorCode="00001"/>

    <field name="status"    column="None" insertValue="'1'"        updateValue="None"/>
    <field name="datetime0" column="None" type="DateTime" insertValue="getdate()" updateValue="None"/>
    <field name="datetime2" column="None" type="DateTime" insertValue="getdate()" updateValue="getdate()"/>
    <field name="user_id0"  column="None" type="Int"      insertValue="@@userID"  updateValue="None"/>
    <field name="user_id2"  column="None" type="Int"      insertValue="@@userID"  updateValue="@@userID"/>
  </fields>
```

#### Ví dụ — ItemGroup (`dmnhvt`) — composite PK

```xml
  <fields identity="true" name="stt">
    <field name="loai_nh" column="A" isPrimaryKey="true" type="Decimal" defaultValue="1"
           updateValue="None" check="loai_nh not in ('1', '2', '3')" errorCode="00002"/>
    <field name="ma_nh"   column="B" isPrimaryKey="true" allowNulls="false" upperCase="true" updateValue="None"/>
    <field name="ten_nh"  column="C" allowNulls="false"/>
    <field name="ten_nh2" column="D"/>

    <field name="status"    column="None" insertValue="'1'"        updateValue="None"/>
    <field name="datetime0" column="None" type="DateTime" insertValue="getdate()" updateValue="None"/>
    <field name="datetime2" column="None" type="DateTime" insertValue="getdate()" updateValue="getdate()"/>
    <field name="user_id0"  column="None" type="Int"      insertValue="@@userID"  updateValue="None"/>
    <field name="user_id2"  column="None" type="Int"      insertValue="@@userID"  updateValue="@@userID"/>
  </fields>
```

#### Pattern `check=` thường gặp

```xml
<!-- FK bắt buộc tồn tại -->
check="ma_nt not in (select ma_nt from dmnt)"

<!-- FK tùy chọn: chỉ check khi có giá trị -->
check="ma_kh &lt;&gt; '' and ma_kh not in (select ma_kh from dmkh)"

<!-- FK tùy chọn + không được tự tham chiếu -->
check="(ma_vv_me &lt;&gt; '' and ma_vv_me not in (select ma_vv from dmvv)) or ma_vv_me = ma_vv"

<!-- Flag chỉ nhận 0/1 -->
check="vv_sd_pslk not in ('0', '1')"

<!-- Enum -->
check="loai_nh not in ('1', '2', '3')"

<!-- FK nhóm theo loại -->
check="nh_vv1 &lt;&gt; '' and nh_vv1 not in (select ma_nh from dmnhvv where loai_nh = 1)"
```

> `&lt;&gt;` = XML escape của `<>` trong attribute.

### `<template>` — Cấu trúc file Excel

```xml
  <template>
    <setting>
      <downloadFile>
        <text v="Tên file tiếng Việt" e="File Name English"/>
      </downloadFile>
    </setting>
    <fields row="5">
      <field name="{tên_cột_db}" width="{px}" [starColor="&EIT.StarColor.Require;">
        <text v="Tên cột VN" e="Column Name EN"/>
      </field>
      &EIT.NoteField;
    </fields>
  </template>
```

Độ rộng cột: mã `16`, tên ngắn `24`, tên dài `32`, ngày `16`, số tiền `16`, flag `12`.

> ⚠️ **Quy tắc bắt buộc — fields ↔ template phải khớp 1-1:**
> Mọi `<field>` trong `<fields>` có `column="A/B/C/..."` (tức là đọc từ Excel) **bắt buộc phải có một entry tương ứng trong `<template>`** với cùng `name`.
> Nếu thiếu → framework báo lỗi khi user tải file Excel (download template thất bại).
> Ngược lại, field có `column="None"` (audit fields) **không được** khai báo trong `<template>`.
>
> **Quy trình kiểm tra:**
> 1. Liệt kê tất cả `name` có `column ≠ "None"` từ `<fields>`.
> 2. Đảm bảo danh sách đó xuất hiện đầy đủ (và đúng thứ tự cột A→Z) trong `<template><fields>`.
> 3. `&EIT.NoteField;` luôn đặt cuối cùng trong template.

### `<processing>` — SQL xử lý import

**Đầu processing (chung):**

```sql
if @@admin = 0 and @type = '1' begin
  if @@sysDatabaseName.dbo.FastBusiness$System$GetAuthorize(@@admin, @@userID, '{Controller}', 'Edit') = 0
    select @type = '0'
end

declare @message nvarchar(4000), @q nvarchar(4000),
        @duplicate nvarchar(4000), @irregular nvarchar(4000),
        @irregularChars varchar(128), @field varchar(32), @r int

select @irregularChars = ]]>&IrregularValue;

select @irregular = case @@language
  when 'v' then N'Giá trị tại ô <span class="Highlight">%invalidCell</span> có chứa các ký tự: ' + @irregularChars
  else 'The value of cell <span class="Highlight">%invalidCell</span> contains any of the following characters: ' + @irregularChars end
select @duplicate = case @@language
  when 'v' then N'Giá trị tại ô <span class="Highlight">%invalidCell</span> đã có hoặc lồng nhau.'
  else 'The value of cell <span class="Highlight">%invalidCell</span> is invalid or already exists.' end

create index i on @@table ({cột_pk})
```

**Tiền xử lý** (nếu cần gán giá trị mặc định):

```sql
-- VD: gán ngoại tệ mặc định
declare @baseCurrency varchar(32)
select @baseCurrency = rtrim(val) from options where name = 'm_ma_nt0'
update @@table set ma_nt = case when ma_nt <> '' then ma_nt else @baseCurrency end
```

**Xóa dòng đã tồn tại nếu type=0:**

```sql
-- Single PK
if @type = '0' delete @@table from @@table a join {tên_bảng} b on a.{pk} = b.{pk}
-- Composite PK
if @type = '0' delete @@table from @@table a join {tên_bảng} b on a.{pk1} = b.{pk1} and a.{pk2} = b.{pk2}
```

**Checking + validate ký tự không hợp lệ:**

```sql
]]>&Checking;

select @field = '{cột_mã}'
if @$mode = 1 begin
  ]]>&IrregularMessage;
  ]]>&StartErrorCount;
  ]]>&InsertErrorTable; select @field, stt, @message from @@table
    where {cột_mã} like '%[' + @irregularChars + ']%'
  ]]>&EndErrorCount;
end else begin
  select @r = min(stt) from @@table where {cột_mã} like '%[' + @irregularChars + ']%'
  ]]>&Irregular;
end
```

**Tách dòng trùng + Insert/Update:**

```sql
-- Single PK
select a.* into #k from @@table a join {tên_bảng} b with (nolock) on a.{pk} = b.{pk}
-- Do not delete following line
-- #OverwriteChecking
if @type = '1' delete @@table where {pk} in (select {pk} from #k)

-- Composite PK
select a.* into #k from @@table a join {tên_bảng} b on a.{pk1} = b.{pk1} and a.{pk2} = b.{pk2}
delete @@table from @@table a join #k b on a.{pk1} = b.{pk1} and a.{pk2} = b.{pk2}

]]>&EndErrorMode;

]]>&Inserting;

if @type = '1' begin
  ]]>&ListWhenBeforeImportUpdateLog;
  ]]>&Updating;
end
```

### Ví dụ processing đầy đủ — ItemGroup (composite PK)

```xml
  <processing>
    <text><![CDATA[
if @@admin = 0 and @type = '1' begin
  if @@sysDatabaseName.dbo.FastBusiness$System$GetAuthorize(@@admin, @@userID, 'ItemGroup', 'Edit') = 0 select @type = '0'
end

declare @message nvarchar(4000), @q nvarchar(4000), @duplicate nvarchar(4000),
        @irregular nvarchar(4000), @irregularChars varchar(128), @field varchar(32), @r int

select @irregularChars = ]]>&IrregularValue;<![CDATA[
select @irregular = case @@language when 'v'
  then N'Giá trị tại ô <span class="Highlight">%invalidCell</span> có chứa các ký tự: ' + @irregularChars
  else 'The value of cell <span class="Highlight">%invalidCell</span> contains any of the following characters: ' + @irregularChars end
select @duplicate = case @@language when 'v'
  then N'Giá trị tại ô <span class="Highlight">%invalidCell</span> đã có hoặc lồng nhau.'
  else 'The value of cell <span class="Highlight">%invalidCell</span> is invalid or already exists.' end

create index i on @@table (loai_nh, ma_nh)

if @type = '0' delete @@table from @@table a join dmnhvt b on a.loai_nh = b.loai_nh and a.ma_nh = b.ma_nh

]]>&Checking;<![CDATA[

select @field = 'ma_nh'
if @$mode = 1 begin
  ]]>&IrregularMessage;<![CDATA[
  ]]>&StartErrorCount;<![CDATA[
  ]]>&InsertErrorTable;<![CDATA[ select @field, stt, @message from @@table
    where ma_nh like '%[' + @irregularChars + ']%'
  ]]>&EndErrorCount;<![CDATA[
end else begin
  select @r = min(stt) from @@table where ma_nh like '%[' + @irregularChars + ']%'
  ]]>&Irregular;<![CDATA[
end

select a.* into #k from @@table a join dmnhvt b on a.loai_nh = b.loai_nh and a.ma_nh = b.ma_nh
delete @@table from @@table a join #k b on a.loai_nh = b.loai_nh and a.ma_nh = b.ma_nh

]]>&EndErrorMode;<![CDATA[

]]>&Inserting;<![CDATA[

if @type = '1' begin
  ]]>&ListWhenBeforeImportUpdateLog;<![CDATA[
  ]]>&Updating;<![CDATA[
end
]]>
    </text>
  </processing>
</import>
```

---

## Biến đặc biệt trong processing

| Biến | Ý nghĩa |
|---|---|
| `@@table` | Bảng tạm chứa dữ liệu từ Excel |
| `@type` | `'0'` = bỏ qua trùng, `'1'` = chép đè |
| `@$mode` | `1` = batch (liệt kê hết lỗi), `0` = dừng ngay |
| `@@admin`, `@@userID` | Phân quyền |
| `@@language` | `'v'` = Việt |
| `@irregularChars` | Ký tự không hợp lệ cho mã |
| `@r` | Dòng lỗi |
| `@field` | Tên cột đang validate |
| `%invalidCell` | Địa chỉ ô lỗi — framework tự thay |

---

## Checklist

### Grid
- [ ] DOCTYPE: `DowloadScript`, `TransferID`, `CreateTicket`, `%ExportImportTemplate`, `ExportImportTemplate.UploadController`
- [ ] Loading: `&CreateTicket;` + gán `_authorize` + `_key`
- [ ] Script: `load$Grid` đăng ký cả `commandEvent` + `onResponseComplete`
- [ ] Script: `on$Grid{Controller}$ExecuteCommand` xử lý `ImportData` + `Download`
- [ ] Script: `show$Form` kiểm tra `_authorize`
- [ ] CSS: icon cho `div.ImportData` + `div.Download`
- [ ] Response: action `Download` → `&CreateTicket;` → `select @ticket`
- [ ] Toolbar: `command="ImportData"` + `command="Download"`

### Filter Import Form
- [ ] DOCTYPE: 5 ENTITY upload + `%ImportErrorMode`
- [ ] `Identity` = `{Controller}ImportForm`
- [ ] Fields: `&UploadField;` + `type` + `ticket` + `&FilterFormModeField;`
- [ ] Views: layout 7 cột + `&FilterFormModeView;`
- [ ] Commands: `&UploadCommand;` + event `Checking`
- [ ] Script: `init${Identity}` + `on${Identity}$ResponseComplete`
- [ ] Response: `GetTicket` → `&UploadCreateTicket;`
- [ ] Kết thúc: `&UploadButtonCss;`

### Upload XML
- [ ] DOCTYPE: `IrregularValue`, 3 ENTITY lỗi, `Checking/Inserting/Updating`, `%ImportErrorMode`, `%ExportImportTemplate`, `ExportQueryStaticFile`, `%ListEditLog`
- [ ] `<setting>`: `startRow`, `baseTable`, `onProcessFail`/`onProcessComplete` đúng callback
- [ ] `<fields identity="true">`: mọi cột Excel có field + đúng `column=`, `type=`, `check=`
- [ ] Audit fields: `status`, `datetime0/2`, `user_id0/2` với `column="None"`
- [ ] `<template>`: `row="5"`, **đủ field cho mọi `column ≠ "None"`** (thiếu 1 field → lỗi tải file), `&EIT.NoteField;` cuối
- [ ] `<processing>`: declare, index, check irregular, tách #k, EndErrorMode, Inserting, Updating
