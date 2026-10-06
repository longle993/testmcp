# [SPEC_grid-detail] — Danh mục / Khai báo có Grid Detail (Master-Detail)

> Dùng khi: danh mục hoặc khai báo cần nhập liệu nhiều dòng chi tiết (grid) bên trong form Dir.
> Đây là dạng phức tạp cao, ít phát sinh hơn danh mục thông thường.
>
> ⭐ Mẫu chuẩn:
> - **Dạng 1 — Khóa do user nhập**: `Dir_LoanContract.xml` + `Grid_LoanContractInterestRate.xml` + `Grid_LoanContractPayment.xml`
> - **Dạng 2 — Khóa tăng tự động**: `Dir_AllocationTran.xml` + `Grid_AllocationDetail.xml`

---

## Tổng quan

Danh mục có Grid Detail = form nhập liệu (Dir) chứa một hoặc nhiều lưới nhập liệu dạng nhiều dòng (Grid Detail). Khi lưu Dir, SQL xử lý cả dữ liệu header lẫn dữ liệu chi tiết.

### Thành phần

| Thành phần | File | Vai trò |
|---|---|---|
| Dir (master form) | `dir_{Controller}.xml` | Form header + khai báo field Grid |
| Grid Detail | `grid_{GridController}.xml` | Lưới nhập liệu nhiều dòng, nhúng trong Dir |
| Grid Browse | `grid_{Controller}.xml` | Danh sách Browse ngoài (như danh mục thường) |
| ASPX | `aspx_{Controller}.aspx` | Trang web — như danh mục thường |

### Hai dạng khóa chính

| | Dạng 1 — Khóa do user nhập | Dạng 2 — Khóa tăng tự động |
|---|---|---|
| Ví dụ | Khế ước (`ma_ku`) | Bút toán phân bổ (`stt_rec`) |
| PK field | Hiển thị, user nhập hoặc chọn AutoComplete | Ẩn (`hidden="true"`), readonly |
| Tạo giá trị PK | User nhập trực tiếp | SQL tính `max + 1` trong Inserting |
| `type` trên `<dir>` | *(không có hoặc mặc định)* | `type="Voucher"` |
| `<partition>` | Không cần | ✅ Bắt buộc (cho cả Dir và Grid Detail) |
| Check mã lồng | Có thể có (Nested pattern) | Không (vì PK là số tự tăng) |

---

## Phần 1 — Dir: Khai báo field Grid Detail

### Field Grid Detail trong `<fields>`

Mỗi grid detail là một field `external="true"` với `<items style="Grid">`:

```xml
<field name="{tên_biến_grid}" external="true" clientDefault="0" defaultValue="0" rows="{chiều_cao_px}" filterSource="Tidy" categoryIndex="{tab_chứa_grid}">
  <header v="" e=""></header>
  <label v="Tên hiển thị VN" e="Display Name EN"></label>
  <items style="Grid" controller="{GridDetailController}" row="1">
    <item value="ForeignKey">
      <text v="{Type}: {fk_field_detail}, {pk_field_master}" e="{Type}: {fk_field_detail}, {pk_field_master}"></text>
    </item>
  </items>
</field>
```

#### Thuộc tính field Grid Detail

| Thuộc tính | Giá trị | Mô tả |
|---|---|---|
| `name` | Tự đặt (vd: `ctdmku`, `dmpb1`) | Tên biến — framework tạo biến bảng `@{name}` trong SQL |
| `external="true"` | Luôn `true` | Không phải cột DB thực |
| `clientDefault` | `"0"` hoặc `""` | Giá trị khởi tạo |
| `defaultValue` | `"0"` hoặc `"''"` | `"0"` khi grid bắt buộc, `"''"` khi không |
| `rows` | `"218"`, `"220"` | Chiều cao grid (px) |
| `filterSource` | `"Tidy"` | Luôn đặt `"Tidy"` |
| `categoryIndex` | Số tab | Grid nên nằm trong tab riêng |
| `allowNulls` | `"false"` | Thêm nếu grid bắt buộc có ít nhất 1 dòng |

#### Thẻ con `<items style="Grid">`

| Thuộc tính | Mô tả |
|---|---|
| `controller` | Tên Grid Detail XML (PascalCase, không `.xml`) |
| `row="1"` | Luôn = `"1"` |

#### `<item value="ForeignKey">` — Liên kết khóa

Khai báo cách truyền khóa từ master sang detail:

```xml
<text v="{Type}: {fk_trong_detail}, {pk_trong_master}"
      e="{Type}: {fk_trong_detail}, {pk_trong_master}"></text>
```

| Phần | Mô tả | Ví dụ |
|---|---|---|
| `Type` | Kiểu dữ liệu khóa: `String` hoặc `Decimal` | `String` cho mã text, `Decimal` cho số |
| `fk_trong_detail` | Tên cột FK trong bảng detail | `ma_ku` |
| `pk_trong_master` | Tên cột PK trong bảng master | `ma_ku` |

**Ví dụ Dạng 1** (khóa text):
```xml
<text v="String: ma_ku, ma_ku" e="String: ma_ku, ma_ku"></text>
```

**Ví dụ Dạng 2** (khóa số tự tăng):
```xml
<text v="Decimal: stt_rec, stt_rec" e="Decimal: stt_rec, stt_rec"></text>
```

### Ví dụ khai báo 2 grid trong cùng Dir (LoanContract)

```xml
<!-- Grid 1: Chi tiết lãi suất — tab 2 -->
<field name="ctdmku" external="true" clientDefault="0" defaultValue="0" rows="220" filterSource="Tidy" categoryIndex="2">
  <header v="" e=""></header>
  <label v="Chi tiết lãi suất" e="Interest Rate"></label>
  <items style="Grid" controller="LoanContractInterestRate" row="1">
    <item value="ForeignKey">
      <text v="String: ma_ku, ma_ku" e="String: ma_ku, ma_ku"></text>
    </item>
  </items>
</field>

<!-- Grid 2: Chi tiết thanh toán — tab 3 -->
<field name="ctdmku2" external="true" clientDefault="0" defaultValue="0" rows="220" filterSource="Tidy" categoryIndex="3">
  <header v="" e=""></header>
  <label v="Chi tiết thanh toán" e="Payment"></label>
  <items style="Grid" controller="LoanContractPayment" row="1">
    <item value="ForeignKey">
      <text v="String: ma_ku, ma_ku" e="String: ma_ku, ma_ku"></text>
    </item>
  </items>
</field>
```

---

## Phần 2 — Dir: Layout view có Grid Detail

### Tab chứa grid — `columns` = tổng width 1 cột

Khi tab chỉ chứa grid, `columns` = 1 số duy nhất (tổng chiều rộng):

```xml
<categories>
  <category index="1" columns="120, 30, 70, 110, 120, 100" anchor="4">
    <header v="Thông tin chính" e="General"/>
  </category>
  <!-- Tab chỉ chứa grid → 1 cột duy nhất -->
  <category index="2" columns="566" anchor="1">
    <header v="Thông tin lãi suất" e="Interest Rate"/>
  </category>
  <category index="3" columns="566" anchor="1">
    <header v="Thông tin thanh toán" e="Payment"/>
  </category>
</categories>
```

### Layout dòng grid — `"1: [field_grid]"`

Grid detail chiếm toàn bộ dòng, layout đơn giản:

```xml
<!-- Trong tab có 1 cột → layout 1 ký tự -->
<item value="1: [ctdmku]"/>
<item value="1: [ctdmku2]"/>
```

### Thuộc tính `anchor` trên `<view>` và `<category>`

| Thuộc tính | Mô tả |
|---|---|
| `anchor` trên `<view>` | Số dòng header (ngoài tab) cố định khi resize |
| `anchor` trên `<category>` | Số dòng trong tab cố định (thường `"1"` cho tab grid, `"4"` - `"8"` cho tab field) |

```xml
<view id="Dir" height="280" anchor="5">
```

### Ví dụ layout — Dạng 2 (AllocationTran, không tab riêng cho header)

```xml
<view id="Dir" height="278" anchor="7">
  <item value="20, 100, 25, 5, 45, 25, 303, 25, 25, 35, 120, 25"/>
  <item value="10101------1: [stt].Label, [stt], [stt_rec], [cookie]"/>
  <item value="101000000000: [ten_bt].Label, [ten_bt]"/>
  <item value="101010000000: [loai_pb].Label, [loai_pb], [loai_pb].Description"/>
  <item value="101000100000: [tk].Label, [tk], [ten_tk_pb%l]"/>

  <!-- Grid detail chiếm 1 tab -->
  <item value="1: [dmpb1]"/>

  <!-- Các field khác ở tab khác -->
  <item value="1010001: [ngay_hl_tu].Description, [ngay_hl_tu], [ngay_hl_den]"/>
  <item value="101010000: [status].Label, [status], [status].Description"/>

  <categories>
    <category index="1" columns="769" anchor="1">
      <header v="Chi tiết" e="Detail"/>
    </category>
    <category index="-1" columns="20, 100, 25, 5, 45, 25, 100, 25, 25, 35, 120, 25" anchor="8">
      <header v="" e=""/>
    </category>
  </categories>
</view>
```

> `categoryIndex="-1"` → tab ẩn (không hiện header tab, dùng cho field phụ).

---

## Phần 3 — Dir: Cookie field (Concurrency Check)

Danh mục có grid detail thường dùng **cookie** để kiểm tra xung đột sửa đồng thời:

### Khai báo field cookie

```xml
<field name="cookie" external="true" hidden="true" readOnly="true"
       defaultValue="''" allowContain="true">
  <header v="" e=""></header>
</field>
```

Thuộc tính `name="cookie"` trên `<dir>` phải khớp:
```xml
<dir table="dmku" code="ma_ku" order="ma_ku" name="cookie" check="true" ...>
```

| Thuộc tính `<dir>` | Mô tả |
|---|---|
| `name="cookie"` | Tên field chứa giá trị cookie |
| `check="true"` | Bật kiểm tra xung đột |

### InitExternalFields — Gán giá trị cookie

```xml
<command event="InitExternalFields">
  <text><![CDATA[
select convert(varchar(19), datetime2, 121) as cookie from @@table where {pk} = @{pk}
return
]]></text>
</command>
```

> Cookie = giá trị `datetime2` (thời điểm cập nhật cuối). Framework so sánh cookie khi lưu để phát hiện xung đột.

---

## Phần 4 — Dir: `<partition>` (Dạng 2 — Khóa tăng tự động)

Khi khóa chính là số tự tăng, Dir cần khai báo `<partition>`:

```xml
<dir table="dmpb" code="stt_rec" order="stt_rec" type="Voucher" name="cookie" check="true" ...>
  <partition table="dmpb" prime="dmpb" inquiry="" field="stt_rec" expression="''" increase="{0}" default=""/>
  ...
</dir>
```

| Thuộc tính | Mô tả |
|---|---|
| `table` | Tên bảng |
| `prime` | Bảng chính |
| `field` | Cột khóa tự tăng |
| `increase="{0}"` | Framework quản lý tăng số |

### Field PK ẩn

```xml
<field name="stt_rec" type="Decimal" isPrimaryKey="true" width="0" hidden="true" readOnly="true">
  <header v="" e=""></header>
</field>
```

### SQL Inserting — Tính giá trị PK mới

```sql
declare @idNumber int
select @idNumber = max(stt_rec) from dmpb
select @idNumber = isnull(@idNumber, 0) + 1
select @stt_rec = isnull(@idNumber, 1),
       @datetime0 = getdate(), @datetime2 = getdate(),
       @user_id0 = @@userID, @user_id2 = @@userID
-- Gán PK cho bảng detail
update @dmpb1 set stt_rec = @idNumber
```

### SQL Inserted — Trả PK về client

```sql
-- Sau khi insert detail
select @idNumber as stt_rec
return
```

> Framework nhận lại giá trị PK để hiển thị khi chuyển sang chế độ Edit.

---

## Phần 5 — Dir: SQL xử lý Grid Detail

### Biến bảng tự động

Framework tự tạo biến bảng `@{tên_field_grid}` chứa dữ liệu từ grid. Ví dụ:
- Field `name="ctdmku"` → biến `@ctdmku`
- Field `name="dmpb1"` → biến `@dmpb1`

### Kiểm tra thay đổi — `$field.NewValue` vs `$field.OldValue`

Framework cung cấp 2 giá trị đặc biệt cho field grid:

| Biến | Mô tả |
|---|---|
| `$ctdmku.NewValue` | Hash/giá trị mới của grid |
| `$ctdmku.OldValue` | Hash/giá trị cũ |

So sánh 2 giá trị để biết grid có thay đổi hay không:

```sql
if $ctdmku.NewValue <> $ctdmku.OldValue ...
```

### Template SQL — Dạng 1 (Khóa do user nhập)

#### Inserting

```sql
-- Validate trùng mã (như danh mục thường)
if exists(select * from @@table where {pk} ...) begin
  select '{pk}' as field, replace(@$exists, '%s', rtrim(@{pk})) as message
  return
end
select @datetime0 = getdate(), @datetime2 = getdate(),
       @user_id0 = @@userID, @user_id2 = @@userID

-- Gán FK cho dữ liệu detail
update @{grid1} set {fk} = @{pk}
update @{grid2} set {fk} = @{pk}
```

#### Inserted

```sql
-- Insert dữ liệu detail vào bảng thực
insert into {bảng_detail1} select * from @{grid1}
insert into {bảng_detail2} select * from @{grid2}
```

#### Updating

```sql
-- Validate (như danh mục thường)
...

-- Gán FK mới cho detail
update @{grid1} set {fk} = @{pk}
update @{grid2} set {fk} = @{pk}

-- Nếu đổi mã PK và grid KHÔNG thay đổi → chỉ update FK trong bảng thực
if (@{pk} <> ${pk}.OldValue) begin
  if ${grid1}.NewValue = ${grid1}.OldValue
    update {bảng_detail1} set {fk} = @{pk} where {fk} = ${pk}.OldValue
  if ${grid2}.NewValue = ${grid2}.OldValue
    update {bảng_detail2} set {fk} = @{pk} where {fk} = ${pk}.OldValue
end

-- Nếu grid có thay đổi → xóa cũ (insert mới ở Updated)
if ${grid1}.NewValue <> ${grid1}.OldValue
  delete {bảng_detail1} where {fk} = ${pk}.OldValue
if ${grid2}.NewValue <> ${grid2}.OldValue
  delete {bảng_detail2} where {fk} = ${pk}.OldValue
```

#### Updated

```sql
-- Insert dữ liệu mới nếu grid có thay đổi
if ${grid1}.NewValue <> ${grid1}.OldValue
  insert into {bảng_detail1} select * from @{grid1}
if ${grid2}.NewValue <> ${grid2}.OldValue
  insert into {bảng_detail2} select * from @{grid2}

-- Update audit
update @@table set datetime2 = getdate(), user_id2 = @@userID where {pk} = @{pk}
```

#### Deleting

```sql
-- Kiểm tra phát sinh (nếu cần)
...
-- Xóa detail trước, master sau
delete {bảng_detail1} where {fk} = @{pk}
delete {bảng_detail2} where {fk} = @{pk}
```

> Lưu ý: framework tự xóa master record. Trong `Deleting` chỉ cần xóa detail và validate.

### Template SQL — Dạng 2 (Khóa tăng tự động)

#### Inserting

```sql
-- Validate logic nghiệp vụ (không check trùng PK vì tự tăng)
if exists(select * from @@table where stt = @stt) begin
  select 'stt' as field, replace(@$exists, '%s', rtrim(@stt)) as message
  return
end

-- Tính PK mới
declare @idNumber int
select @idNumber = max(stt_rec) from {bảng_master}
select @idNumber = isnull(@idNumber, 0) + 1
select @stt_rec = isnull(@idNumber, 1),
       @datetime0 = getdate(), @datetime2 = getdate(),
       @user_id0 = @@userID, @user_id2 = @@userID

-- Gán PK cho detail
update @{grid} set stt_rec = @idNumber
```

#### Inserted

```sql
insert into {bảng_detail} select * from @{grid}
-- Trả PK về client
select @idNumber as stt_rec
return
```

#### Updating — Câu lệnh điều kiện `#IF ... #THEN ... #END`

Dạng 2 thường dùng **preprocessor directive** để tối ưu SQL:

```sql
#IF ${grid}.NewValue <> ${grid}.OldValue #THEN
  -- SQL này chỉ chạy khi grid có thay đổi
  update @{grid} set stt_rec = @stt_rec
#END
```

> `#IF ... #THEN ... #END` là preprocessor của framework — tương tự `#ifdef` trong C.
> Framework sẽ loại bỏ khối SQL nếu điều kiện không đúng, giúp tối ưu performance.

#### Updated (Dạng 2)

```sql
#IF ${grid}.NewValue <> ${grid}.OldValue #THEN
  delete {bảng_detail} where stt_rec = @stt_rec
  insert into {bảng_detail} select * from @{grid}
#END

update @@table set datetime2 = getdate(), user_id2 = @@userID where stt_rec = @stt_rec
```

#### Deleting (Dạng 2)

```sql
-- Xóa detail
delete {bảng_detail} where stt_rec = @stt_rec
-- Framework tự xóa master
```

---

## Phần 6 — Dir: JavaScript (Checking + Tab)

### Checking — Validate dữ liệu grid trước khi lưu

Event `Checking` chạy trên client (JavaScript) trước khi gửi lên server. Dùng để:
- Kiểm tra trùng giá trị trong grid
- Gán số thứ tự dòng (`line_nbr`)
- Validate logic đặc thù

```javascript
// Trong <command event="Checking">
var f = this;
// Lấy grid object qua _controlBehavior
var g = f.getItem('{tên_field_grid}')._controlBehavior;
```

### API Grid Detail trong Checking

| API | Mô tả |
|---|---|
| `g._rowCount` | Số dòng hiện tại |
| `g._getColumnOrder('{field}')` | Lấy index cột theo tên |
| `g._getItemValue(row, col)` | Lấy giá trị ô (row bắt đầu từ 1) |
| `g._setItemValue(row, col, value)` | Gán giá trị ô |
| `g._getItem(row, col)` | Lấy DOM element của ô |
| `g._errorObject` | Gán ô lỗi để focus |
| `g.setSequenceNumber('{field}')` | Tự gán số thứ tự 1, 2, 3... (dùng cho `line_nbr`) |
| `g.setContinuance('{field}')` | Tự gán mã chuỗi tăng dần `'001'`, `'002'`... (dùng cho `stt_rec0`) |
| `g.setItemFieldValue(field, row, value)` | Gán giá trị cho field tại dòng cụ thể |
| `g.validRowExpression(field, expression)` | Validate biểu thức JS trên từng dòng |

### Pattern: Gather event — Gán line_nbr + stt_rec0 ⭐

Ngoài cách gán `line_nbr` trong `<command event="Checking">` (JS flatten), có pattern khác sử dụng **event `Gather`** trong `ExecuteCommand` của Dir. Pattern này phù hợp khi grid detail có `stt_rec0` (mã chuỗi tự tăng).

**Trong Dir `<script>` — đăng ký `commandEvent`:**

```javascript
function active$Form{Controller}(f) {
  f.add_onResponseComplete(on$Form{Controller}$ResponseComplete);
  f.add_commandEvent(on$Form{Controller}$ExecuteCommand);
}
function close$Form{Controller}(f) {
  try {f.remove_commandEvent(on$Form{Controller}$ExecuteCommand);} catch (ex) {}
  try {f.remove_onResponseComplete(on$Form{Controller}$ResponseComplete)} catch (ex) {}
}
```

**Xử lý event `Gather`:**

```javascript
function on$Form{Controller}$ExecuteCommand(sender, e) {
  var action = e.type.Action, f = sender,
      g = f.getItem('{tên_field_grid}')._controlBehavior;
  switch (action) {
    case 'Gather':
      // Gán line_nbr = 1, 2, 3... theo thứ tự dòng
      g.setSequenceNumber('line_nbr');
      // Gán stt_rec0 = '001', '002', '003'... theo line_nbr
      g.setContinuance('stt_rec0');
      break;
    default:
      break;
  }
}
```

> **`Gather`** là event framework gọi trước khi thu thập dữ liệu grid để gửi lên server (trước `Inserting`/`Updating`).
> `setSequenceNumber('line_nbr')` — gán số nguyên 1, 2, 3...
> `setContinuance('stt_rec0')` — gán chuỗi `'001'`, `'002'`, `'003'`... dựa trên thứ tự `line_nbr`.

**So sánh 2 cách gán `line_nbr`:**

| | Checking (JS flatten) | Gather (ExecuteCommand) |
|---|---|---|
| Vị trí code | `<command event="Checking">` trong Dir commands | `on$Form{Controller}$ExecuteCommand` trong Dir script |
| Khi nào chạy | Trước khi gửi form | Trước khi thu thập dữ liệu grid |
| Có thể gán `stt_rec0` | ❌ Không (phải loop thủ công) | ✅ Có (`setContinuance`) |
| Dùng khi | Cần validate phức tạp (trùng ngày, logic đặc thù) | Chỉ cần gán số thứ tự + mã chuỗi |

### Ví dụ: Kiểm tra trùng ngày + gán line_nbr

```javascript
/* <flatten type="Javascript"> */
var f = this, g = f.getItem('ctdmku')._controlBehavior;

// Kiểm tra trùng ngày hiệu lực
for (var i = 0; i < g._rowCount - 1; i++) {
  var c = g._getColumnOrder('ngay_hl');
  var v = g._getItemValue(i + 1, c);
  for (var j = i + 1; j < g._rowCount; j++) {
    var d = g._getItemValue(j + 1, c);
    if ($func.compareDateTimeValue(v, d)) {
      f._checked = false;
      g._errorObject = g._getItem(j + 1, c);
      break;
    }
  }
  if (!f._checked) break;
}

// Gán line_nbr nếu hợp lệ
if (f._checked) {
  var c = g._getColumnOrder('line_nbr');
  for (var i = 0; i < g._rowCount; i++) {
    g._setItemValue(i + 1, c, i + 1);
  }
}

// Hiện thông báo lỗi nếu không hợp lệ
if (!f._checked) {
  var errorMessage = (f._language == 'v'? 'Trường <span class="Highlight">Ngày hiệu lực</span> giá trị nhập không hợp lệ...' : 'Field <span class="Highlight">Effective Date</span> has invalid value...');
  $func.hideWait(f.get_id());
  f.focus('ctdmku');
  $message.show(errorMessage, String.format('$find(\'{0}\')._errorObject.focus()', g.get_id()));
}
/* </flatten> */
```

### Ví dụ đơn giản: Chỉ gán line_nbr (không validate phức tạp)

```javascript
/* <flatten type="Javascript"> */
var f = this, g = f.getItem('{tên_field_grid}')._controlBehavior;
if (f._checked) g.setSequenceNumber('line_nbr');
/* </flatten> */
```

### Tab focus khi có grid detail

```javascript
function onTabChanged{Controller}(sender, e) {
  // Mỗi phần tử trong mảng = field đầu tiên của tab tương ứng
  // Tab chứa grid → dùng tên field grid
  sender.parentForm.focusWhenTabChanged([
    'ngay_ku',  // Tab 1: field đầu tiên
    'ctdmku',   // Tab 2: grid lãi suất
    'ctdmku2',  // Tab 3: grid thanh toán
    'ghi_chu'   // Tab 9: ghi chú
  ]);
}
```

---

## Phần 7 — Grid Detail XML

Grid Detail có cùng cấu trúc với Grid Browse, nhưng phục vụ **nhập liệu** (editable) thay vì chỉ hiển thị.

### Cấu trúc chuẩn

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE grid [
  <!ENTITY % Extra SYSTEM "..\Include\Extra\Extra.ent">
  %Extra;
]>

<grid table="{bảng_detail}" code="{cột_pk_master}" order="{sắp_xếp}"
      type="Detail" xmlns="urn:schemas-fast-com:data-grid">
  <title v="" e=""></title>
  <subTitle v="" e=""></subTitle>
  <fields> ... </fields>
  <views> ... </views>
  <commands> ... </commands>
  <script> ... </script>
  <queries> ... </queries>   <!-- ⭐ BẮT BUỘC — query Loading -->
  <css> ... </css>
  <toolbar> ... </toolbar>
</grid>
```

### Khác biệt so với Grid Browse

| Đặc điểm | Grid Browse | Grid Detail |
|---|---|---|
| Mục đích | Hiển thị danh sách, chọn để Edit/Delete | Nhập liệu nhiều dòng trong Dir |
| Toolbar | New, Edit, Delete, Search, View, Export | **Insert, Grow, Down, Clone, Remove** |
| `<title>` | Có tiêu đề | Để trống (`v="" e=""`) |
| Fields | Thường readOnly | Editable (có `<items style="...">`) |
| Field `line_nbr` | Không có | ✅ Luôn có — ẩn, isPrimaryKey |
| Field `stt_rec0` | Không có | Tùy chọn — mã dòng tự tăng dạng chuỗi (`'001'`, `'002'`...) |
| Loading message | `load$Grid(this)` | `load$Grid{TênGrid}$(this)` |
| Closing message | `dispose$Grid(this)` | `close$Grid{TênGrid}$(this)` |
| `type` trên `<grid>` | *(không có)* | `type="Detail"` |
| `<partition>` | Không có | Chỉ Dạng 2 (Dir partition) |
| `<queries>` | Không có | ✅ Luôn cần — query Loading lấy dữ liệu chi tiết |

### Khai báo fields

#### Field nhập liệu (editable)

```xml
<!-- Số -->
<field name="ls" type="Decimal" allowNulls="false" dataFormatString="##0.00" width="100" clientDefault="0">
  <header v="Lãi suất (%)" e="Interest Rate (%)"></header>
  <items style="Numeric"/>
</field>

<!-- Ngày -->
<field name="ngay_hl" type="DateTime" allowNulls="false" dataFormatString="@datetimeFormat" align="left" width="100">
  <header v="Hiệu lực từ" e="Effective Date"></header>
</field>

<!-- Text -->
<field name="ghi_chu" width="300">
  <header v="Ghi chú" e="Note"></header>
</field>

<!-- AutoComplete (FK trong grid) -->
<field name="tk" isPrimaryKey="true" allowNulls="false" width="100">
  <header v="Tài khoản" e="Account"></header>
  <items style="AutoComplete" controller="Account" reference="ten_tk%l" key="status = '1'" check="1 = 1" information="tk$dmtk.ten_tk%l" new="Default"/>
</field>
<field name="ten_tk%l" readOnly="true" external="true" defaultValue="''" width="300" inactivate="true">
  <header v="Tên tài khoản" e="Description"></header>
</field>

<!-- Lookup chọn nhiều (trong grid detail) -->
<!-- Prompt ghi "lookup chọn nhiều từ ..." → dùng style="Lookup" -->
<field name="tk_no" width="100">
  <header v="TK nợ" e="Debit Account"></header>
  <items style="Lookup" controller="Account" key="status = '1'" check="1 = 1" />
</field>
<!-- Không cần field tên đi kèm. Chi tiết: SPEC_lookup-multiselect.md -->
```

#### Field `line_nbr` — Luôn có

```xml
<field name="line_nbr" type="Int32" isPrimaryKey="true" width="0"
       align="right" hidden="true">
  <header v="" e=""></header>
</field>
```

> `line_nbr` giữ thứ tự dòng, framework dùng để xác định dòng khi update.
> Giá trị được gán từ JS (`setSequenceNumber` hoặc loop thủ công).

#### Field `stt_rec0` — Mã dòng tự tăng dạng chuỗi (tùy chọn)

Khi grid detail cần mã dòng dạng chuỗi tự tăng (`'001'`, `'002'`, `'003'`...) thay vì chỉ dùng `line_nbr` số nguyên:

```xml
<field name="stt_rec0" isPrimaryKey="true" width="0" hidden="true" aliasName="a">
  <header v="" e=""></header>
</field>
```

> `stt_rec0` là mã chuỗi (string), tự tăng theo `line_nbr`. Framework tạo giá trị dựa trên thứ tự dòng:
> `line_nbr = 1` → `stt_rec0 = '001'`, `line_nbr = 2` → `stt_rec0 = '002'`, v.v.
>
> Giá trị được gán bằng `g.setContinuance('stt_rec0')` trong event `Gather` (xem Phần 6).
> Khi Clone dòng, cần reset `stt_rec0` về rỗng trong event `AfterCloneRow` (xem Phần 7 — Script).

**Khi nào dùng `stt_rec0`:**
- Bảng detail cần mã dòng dạng chuỗi để phục vụ logic nghiệp vụ
- Grid có chức năng Clone dòng — cần reset mã để không trùng
- `order` trên `<grid>` thường là `"{fk}, stt_rec0"` thay vì `"{fk}, line_nbr"`

### `<commands>` — Loading / Closing

Tên hàm JS trong Loading/Closing phải khớp với tên trong `<script>`:

```xml
<commands>
  <command event="Loading">
    <text><![CDATA[
select 'load$Grid{TênGrid}$(this);' as message
return]]></text>
  </command>
  <command event="Closing">
    <text><![CDATA[select 'close$Grid{TênGrid}$(this);' as message
return]]></text>
  </command>
</commands>
```

> ⚠️ `{TênGrid}` = tên bảng detail hoặc controller grid detail (ví dụ: `hckbtttcttdct`, `LoanContractInterestRate`).
> Pattern: `load$Grid` + `{TênGrid}` + `$` — luôn có dấu `$` cuối.

**Ví dụ thực tế:**
```xml
<!-- Grid detail bảng hckbtttcttdct -->
select 'load$Gridhckbtttcttdct$(this);' as message

<!-- Grid detail bảng ctdmku -->
select 'load$Gridctdmku$(this);' as message
```

### `<script>` — Pattern chuẩn

```xml
<script>
  <text>
    <![CDATA[/* <flatten type="Javascript"> */
function load$Grid{TênGrid}$(g) {
  f = g.get_element().parentForm;
  g.add_commandEvent(on$GridGrid{TênGrid}$ExecuteCommand);
}
function close$Grid{TênGrid}$(g) {
  try {g.remove_commandEvent(on$GridGrid{TênGrid}$ExecuteCommand);} catch (ex) {}
}
/* </flatten> */
function on$GridGrid{TênGrid}$ExecuteCommand(sender, e) {
  var action = e.type.Action, g = sender, f = g.get_element().parentForm;
  switch (action) {
    case 'AfterCloneRow':
      // Reset stt_rec0 khi clone dòng — tránh trùng mã
      g.setItemFieldValue('stt_rec0', e.type.Value, '');
      break;
    case 'Check':
      // Validate expression trên từng dòng (tùy chọn)
      // g.validRowExpression('field', 'biểu_thức_JS');
      break;
    ]]>&GridExtraToolbarExecuteCommandScript;<![CDATA[
    default:
      break;
  }
}
]]>
    &GridExtraToolbarScript;
  </text>
</script>
```

> `&GridExtraToolbarExecuteCommandScript;` và `&GridExtraToolbarScript;` là ENTITY chuẩn xử lý toolbar mở rộng (copy/paste dòng). Bỏ qua nếu không cần.

#### Quy tắc đặt tên hàm trong Grid Detail

| Hàm | Pattern | Ví dụ (`TênGrid` = `hckbtttcttdct`) |
|---|---|---|
| Load | `load$Grid{TênGrid}$(g)` | `load$Gridhckbtttcttdct$(g)` |
| Close | `close$Grid{TênGrid}$(g)` | `close$Gridhckbtttcttdct$(g)` |
| ExecuteCommand | `on$GridGrid{TênGrid}$ExecuteCommand` | `on$GridGridhckbtttcttdct$ExecuteCommand` |

#### Event `AfterCloneRow` — Reset field khi clone dòng

Khi user nhấn Clone trên toolbar, framework gọi `AfterCloneRow`. Cần reset các field tự tăng (`stt_rec0`) để tránh trùng:

```javascript
case 'AfterCloneRow':
  // e.type.Value = index dòng mới được clone
  g.setItemFieldValue('stt_rec0', e.type.Value, '');
  break;
```

#### Event `Check` — Validate expression trên từng dòng

Validate biểu thức JavaScript trên grid trước khi submit:

```javascript
case 'Check':
  // Kiểm tra tk_no bắt buộc nhập (trừ một số loại chứng từ)
  g.validRowExpression('tk_no', '([ma_ct] == "HCD") || ([ma_ct] == "HC9") || ([tk_no] != "")');
  g.validRowExpression('tk_co', '([ma_ct] == "HCD") || ([ma_ct] == "HC9") || ([tk_co] != "")');
  break;
```

> `g.validRowExpression(field, expression)` — framework kiểm tra biểu thức JS trên từng dòng.
> Nếu biểu thức trả về `false` → dòng đó báo lỗi tại `field`.

### `<css>` và `<toolbar>`

```xml
<css>
  <text>
<!--&GridExtraToolbarCss;-->
  </text>
</css>

<toolbar>
  <button command="Insert"><title v="Toolbar.Insert" e="Toolbar.Insert"/></button>
  <button command="Grow"><title v="Toolbar.Grow" e="Toolbar.Grow"/></button>
  <button command="Down"><title v="Toolbar.Down" e="Toolbar.Down"/></button>
  <button command="Clone"><title v="Toolbar.Clone" e="Toolbar.Clone"/></button>
  <button command="Remove"><title v="Toolbar.Remove" e="Toolbar.Remove"/></button>

  <!--&GridExtraToolbar;-->

  <button command="Separate"><title v="-" e="-"/></button>
  <button command="Freeze"><title v="Toolbar.Freeze" e="Toolbar.Freeze"/></button>
</toolbar>
```

#### Toolbar commands cho Grid Detail

| Command | Mô tả |
|---|---|
| `Insert` | Thêm dòng mới |
| `Grow` | Thêm dòng cuối |
| `Down` | Di chuyển dòng xuống |
| `Clone` | Nhân bản dòng đang chọn |
| `Remove` | Xóa dòng đang chọn |
| `Freeze` | Đóng băng cột |

### Grid Detail — `type="Detail"` trên `<grid>`

Tất cả Grid Detail đều dùng `type="Detail"`:

```xml
<!-- Dạng 1 — FK là string (id, ma_ku, ...) -->
<grid table="{bảng_detail}" type="Detail" code="{fk}" order="{fk}, stt_rec0" xmlns="urn:schemas-fast-com:data-grid">
  ...
</grid>

<!-- Dạng 2 — Thêm <partition> khi master có khóa tự tăng -->
<grid table="dmpb1" code="stt_rec" order="line_nbr" type="Detail" xmlns="urn:schemas-fast-com:data-grid">
  <partition table="dmpb1" prime="dmpb1" inquiry="" field="" expression="''" increase="{0}" default=""/>
  ...
</grid>
```

> `type="Detail"` cho framework biết đây là grid nhập liệu nhúng trong Dir, không phải grid browse.

### `<queries>` — Query Loading dữ liệu chi tiết ⭐ BẮT BUỘC

> ⚠️ **Grid Detail luôn cần `<queries>`** để load dữ liệu chi tiết khi mở form Edit.
> Đây là phần bị thiếu phổ biến nhất khi tạo Grid Detail.

**Dạng đơn giản** — không cần join bảng khác (tất cả field đều có trong bảng detail):

```xml
<queries>
  <query event="Loading">
    <text><![CDATA[
select @@fieldExternal from {bảng_detail} a where @@whereClause order by @@orderByClause
]]></text>
  </query>
</queries>
```

**Dạng có JOIN** — khi grid có field `external="true"` (tên hiển thị) cần lấy từ bảng khác:

```xml
<queries>
  <query event="Loading">
    <text><![CDATA[
select @@fieldExternal from @@prime$partition$current a
  left join dmtk b on a.tk = b.tk
  left join dmvv c on a.ma_vv = c.ma_vv
where @@whereClause order by @@orderByClause
]]></text>
  </query>
</queries>
```

#### Biến hệ thống trong `<queries>`

| Biến | Mô tả |
|---|---|
| `@@fieldExternal` | Framework tự sinh danh sách cột (bao gồm cả field external) |
| `@@whereClause` | Điều kiện WHERE tự động theo FK |
| `@@orderByClause` | ORDER BY tự động theo `order` trên `<grid>` |
| `@@prime$partition$current` | Bảng detail (dùng cho Dạng 2 có partition) |

> Nếu grid **không có** field external cần join → dùng tên bảng trực tiếp: `from {bảng_detail} a`.
> Nếu grid **có** field external cần join → dùng `@@prime$partition$current a` (Dạng 2) hoặc tên bảng + LEFT JOIN.

---

## Phần 8 — Quy ước đặt tên

### Bảng detail

Quy tắc: `ct{bảng_master}` hoặc `{bảng_master}{số}` hoặc `{bảng_master}ct`:

| Bảng master | Bảng detail | Ý nghĩa |
|---|---|---|
| `dmku` | `ctdmku` | Chi tiết lãi suất |
| `dmku` | `ctdmku2` | Chi tiết thanh toán |
| `dmpb` | `dmpb1` | Chi tiết phân bổ |
| `hckbtttcttd` | `hckbtttcttdct` | Chi tiết chứng từ tự động |

### Tên field grid trong Dir

Thường trùng tên bảng detail: `ctdmku`, `dmpb1`, `hckbtttcttdct`.

### Tên Grid Detail Controller

PascalCase, mô tả nội dung: `LoanContractInterestRate`, `LoanContractPayment`, `AllocationDetail`.

---

## Phần 9 — Tổng hợp: Template tạo mới

### Dạng 1 — Khóa do user nhập

```
Tạo danh mục {Tên} (bảng {dmxx}) — có Grid Detail:
Header: ma_xx (PK, user nhập), ten_xx, ten_xx2
Tab 1 "Thông tin chính": {liệt kê field}
Tab 2 "{Tên grid 1}": Grid {GridController1} (bảng {ct1}) — fields: {liệt kê}
Tab 3 "{Tên grid 2}": Grid {GridController2} (bảng {ct2}) — fields: {liệt kê}
Tab 9 "Khác": ghi_chu, status

Theo Dạng 1 (SPEC_grid-detail) — khóa do user nhập.
Có cookie, có tab.
Chặn sửa/xóa nếu phát sinh ở bảng: {tên_bảng}
```

### Dạng 2 — Khóa tăng tự động

```
Tạo khai báo {Tên} (bảng {dmxx}) — có Grid Detail:
Header: stt_rec (PK ẩn, tự tăng Decimal), stt (số thứ tự user nhập), ten_xx
Tab 1 "Chi tiết": Grid {GridController} (bảng {xx1}) — fields: {liệt kê}
Tab -1: ngay_hl_tu, ngay_hl_den, status

Theo Dạng 2 (SPEC_grid-detail) — khóa tăng tự động.
Có cookie, có partition.
```

### Dạng có stt_rec0 — Grid detail có mã dòng chuỗi tự tăng

```
Tạo danh mục {Tên} (bảng {dmxx}) — có Grid Detail:
Header: id (PK), noi_dung, ma_ct, tk_no, tk_co, status
Tab 1 "{Tên tab 1}": {liệt kê field}
Tab 2 "Chi tiết": Grid {GridController} (bảng {xxct}) — fields: {liệt kê}
  Grid detail có stt_rec0 (mã chuỗi tự tăng '001', '002') + line_nbr

Theo SPEC_grid-detail — có tab.
Grid detail dùng Gather event (setSequenceNumber + setContinuance).
Grid detail cần <queries> Loading.
```

---

## ✅ Checklist

### Dir
- [ ] Field PK: Dạng 1 = hiển thị + user nhập; Dạng 2 = `hidden="true"` + `readOnly="true"`
- [ ] Field grid: `external="true"`, `filterSource="Tidy"`, `<items style="Grid" controller="...">`
- [ ] ForeignKey: `<item value="ForeignKey">` đúng Type (`String` / `Decimal`) và tên field
- [ ] Cookie field: `name="cookie"` + `check="true"` trên `<dir>` + `InitExternalFields`
- [ ] Tab chứa grid: `columns` = 1 số (tổng width), `anchor="1"`
- [ ] Dạng 2: có `<partition>`, `type="Voucher"` trên `<dir>`
- [ ] Inserting: gán FK cho `@{grid}` (`update @{grid} set {fk} = @{pk}`)
- [ ] Inserted: `insert into {bảng_detail} select * from @{grid}`
- [ ] Updating: xử lý đổi mã + grid có/không thay đổi
- [ ] Updated: insert detail mới nếu có thay đổi
- [ ] Deleting: xóa detail trước (`delete {bảng_detail} where {fk} = @{pk}`)
- [ ] Gán line_nbr: qua `Checking` (JS flatten) HOẶC `Gather` (ExecuteCommand)
- [ ] Nếu có `stt_rec0`: `Gather` event gọi cả `setSequenceNumber('line_nbr')` + `setContinuance('stt_rec0')`
- [ ] Tab focus: tên field grid trong mảng `focusWhenTabChanged`

### Grid Detail
- [ ] `type="Detail"` trên `<grid>`
- [ ] `<title>` và `<subTitle>` để trống
- [ ] Field `line_nbr`: `type="Int32"`, `isPrimaryKey="true"`, `hidden="true"`
- [ ] Nếu có `stt_rec0`: `isPrimaryKey="true"`, `hidden="true"`, `width="0"`
- [ ] Toolbar: Insert, Grow, Down, Clone, Remove (KHÔNG phải New, Edit, Delete)
- [ ] Loading: `load$Grid{TênGrid}$(this)` — Closing: `close$Grid{TênGrid}$(this)`
- [ ] ⭐ `<queries>`: **BẮT BUỘC** — query Loading lấy dữ liệu chi tiết
- [ ] Dạng 2: có `<partition>` trên `<grid>`
- [ ] Nếu có field external (tên hiển thị): `<queries>` có LEFT JOIN
- [ ] Nếu có Clone: `AfterCloneRow` event reset `stt_rec0`
- [ ] Nếu có validate dòng: `Check` event dùng `validRowExpression`
