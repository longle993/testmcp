# [SPEC_javascript-patterns] — JavaScript Client-side Patterns

> Dùng khi: viết JS trong thẻ `<script>` của Dir, Grid, Filter.

---

## API cơ bản

### Lấy / gán giá trị field
```javascript
var f = parentForm; // hoặc e.object
f.getItem('ma_vt')             // DOM element của field
f.getItem('ma_vt').value       // giá trị hiện tại
f.getItemValue('ma_vt')        // giá trị (shortcut)
f.setItemValue('ma_vt', '001') // gán giá trị
```

### Focus vào field
```javascript
f.getItem('ma_vt').focus();
```

### Gán giá trị AutoComplete (mã + tên)
```javascript
// setItemControlBehavior(field, giá_trị, tên_hiển_thị, focus)
f.setItemControlBehavior('dvt', 'KG', 'Kilogram', true);
```

### Read-only / Enable field
```javascript
f._setReadOnly(f.getItem('dvt'), true);  // đặt readonly
f._setReadOnly(f.getItem('dvt'), false); // bỏ readonly
o.disabled = true;  // disable field
```

### Tạo request lên server
```javascript
// request(responseId, context, danh_sách_field_gửi, element)
f.request('Item', 'Item', ['ma_vt'], o);
```

### Ẩn/hiện field trên form
```javascript
$common.setVisible(o.parentNode.parentNode.parentNode, false); // ẩn
$common.setVisible(o.parentNode.parentNode.parentNode, true);  // hiện
```

## Pattern: Active / Close form

```javascript
function activeFormMyController(f) {
  f.add_onResponseComplete(on$FormMyController$ResponseComplete);
}
function closeFormMyController(f) {
  try {f.remove_onResponseComplete(on$FormMyController$ResponseComplete);} catch (ex) {}
}
```

## Pattern: onResponseComplete (nhận kết quả từ server)

```javascript
function on$FormMyController$ResponseComplete(sender, e) {
  var f = e.object, context = e.type.Context, result = e.type.Result;
  switch (context) {
    case 'Loading': break;
    case 'Scattering':
      // set giá trị mặc định sau khi load
      break;
    case 'Item':
      // Nhận kết quả từ response "Item"
      // result[0].Value = cột 1, result[1].Value = cột 2, ...
      f.setItemControlBehavior('dvt', result[0].Value, result[1].Value, true);
      break;
    case 'Checking':
      // Từ Filter: xây externalKey truyền sang Grid
      break;
  }
}
```

## Pattern: onChange field (trigger request)

```javascript
function onChange$MyController$Item(o) {
  var f = o.parentForm;
  f.request('Item', 'Item', ['ma_vt'], o);
}
```

## Pattern: Grid — Load / Dispose / Render

```javascript
function load$Grid(g) {
  g.add_onRender(on$GridMyController$Render);
}
function dispose$Grid(g) {
  try {g.remove_onRender(on$GridMyController$Render);} catch (ex) {}
}
function on$GridMyController$Render(sender, eventArgs) {
  var g = eventArgs.grid;
  if (g._hiddenFields) {
    for (var i = 0; i < g._hiddenFields.length; i++)
      setGridHiddenFields(g, g._hiddenFields[i].Fields, g._hiddenFields[i].Value);
  }
}
function setGridHiddenFields(g, c, v) {
  var a = c.split(',');
  for (var i = 0; i < a.length; i++) {
    var l = g._getColumnOrder($func.trim(a[i]));
    if (l != -1) g._setColumnVisible(l, !v);
  }
}
```

## `$message.show` — Hiển thị thông báo / xác nhận

### Dạng 1: Thông báo thông thường (nút OK)

```javascript
$message.show(msg);
```

### Dạng 2: Hộp thoại Có/Không (kèm callback)

```javascript
$message.show(msg, 1, callbackScript);
```

| Tham số | Kiểu | Mô tả |
|---|---|---|
| `msg` | `string` | Nội dung thông báo |
| `type` | `0 \| 1` | `0` = thông báo thường (OK); `1` = xác nhận Có/Không |
| `callbackScript` | `string` | Chuỗi JS thực thi khi user chọn **Có** (chỉ dùng khi `type = 1`) |

> `callbackScript` là chuỗi JS thuần. Dùng `String.format('{0}', ...)` để nhúng biến động vào chuỗi.
> `String.format('onAuto(\'{0}\');', id)` → `"onAuto('abc123');"` — `\\'` là ký tự quote bên trong chuỗi, `{0}` là tham số đầu tiên.

### Pattern chuẩn — Confirm trước khi thực thi (ExecuteCommand)

Dùng `g.get_id()` lấy ID grid, truyền vào callback để `$find` lại grid sau khi user xác nhận:

```javascript
case 'Auto':
  if (g._loai_xl != 1) return;  // kiểm tra điều kiện trước
  var msg = (g._language == 'v')
    ? 'Hệ thống sẽ xóa và tạo lại điều chỉnh. Bạn có muốn tiếp tục không?'
    : 'The system will delete and recreate adjustments. Do you want to continue?';
  var id = g.get_id();
  $message.show(msg, 1, String.format('onAuto(\'{0}\');', id));
  break;
```

Hàm callback — chạy sau khi user chọn **Có**:

```javascript
function onAuto(id) {
  var g = $find(id);
  g.request(g, 'Auto', 'Auto', [
    ['ky',     'Decimal', g._ky],
    ['nam',    'Decimal', g._nam],
    ['ma_vt',  'String',  g._ma_vt],
    ['ma_kho', 'String',  g._ma_kho]
  ], false);
}
```

> `$find(id)` tìm lại control theo ID — cần thiết vì callback chạy ngoài scope ban đầu.
> `g.request(sender, responseId, context, params, keepOpen)` — phiên bản Grid request, `params` là mảng `[name, type, value]`.

### Hủy action mặc định của framework (`cancelEvent`)

Dùng trong `ExecuteCommand` để chặn framework xử lý tiếp (ví dụ chặn mở form sửa/xóa):

```javascript
case 'New':
case 'Edit':
case 'View':
case 'Delete':
  if (g._loai_xl == 1) {
    var msg = (g._language == 'v') ? 'Không thể thao tác.' : 'Operation not allowed.';
    $message.show(msg);
    e.type.cancelEvent = true;  // hủy action, framework không xử lý tiếp
    return;
  }
  break;
```

---

## ENTITY Script chuẩn

```xml
<!-- Dùng khi Dir có kiểm tra mã (irregular check) -->
<!ENTITY ScriptIrregular SYSTEM "..\Include\Javascript\Irregular.txt">

<!-- Dùng trong Filter -->
<!ENTITY JavascriptReportFilter SYSTEM "..\Include\Javascript\ReportFilter.txt">

<!-- Dùng trong Grid báo cáo -->
<!ENTITY JavascriptReportInit SYSTEM "..\Include\Javascript\ReportInit.txt">
<!ENTITY ScriptTagReport SYSTEM "..\Include\Javascript\TagReport.txt">
```
