# [SPEC_show-hide] — Ẩn hiện field trên Form và Grid

> Dùng khi: cần ẩn/hiện field động theo điều kiện lọc hoặc giá trị field khác.

---

## 1. Ẩn/hiện trên Form Dir

### Ẩn field khi khởi tạo form (dựa trên biến từ Filter)

```javascript
function setObjectFormHidden(f, c, l, t, v) {
  var a = c.split(','), b = (t.indexOf(v) > -1);
  for (var i = 0; i < a.length; i++) {
    var name = $func.trim(a[i]), o = f.getItem(name);
    // Đánh dấu required nếu cần
    if (l) {
      o.field.AllowNulls = !b;
      if (b) {
        var grandNode = o.parentNode.parentNode;
        Sys.UI.DomElement.addCssClass(grandNode, 'Required');
        Sys.UI.DomElement.addCssClass(grandNode, f._id);
      }
    }
    // Ẩn nếu không thuộc loại này
    if (!b) $common.setVisible(o.parentNode.parentNode.parentNode, false);
  }
}

// Sử dụng: t = "SCDEFG", v = ký tự đại diện cho field
// Ví dụ: 'S' = Kho, 'C' = Khách, 'D' = Nhóm KH 1...
function activeFormDetail(f) {
  var t = f.grid._salesPriceType.split(',')[0];
  setObjectFormHidden(f, 'ma_kho', true, t, 'S');
  setObjectFormHidden(f, 'ma_kh', true, t, 'C');
  setObjectFormHidden(f, 'nh_kh1', true, t, 'D');
}
```

### Ẩn/hiện field trên Filter (enable/disable + readonly)

```javascript
function setObject$Filter$Type(f, c, t, v) {
  var a = c.split(','), b = (t.indexOf(v) > -1);
  for (var i = 0; i < a.length; i++) {
    o = f.getItem($func.trim(a[i]));
    o.disabled = !b;        // disable nếu không khớp
    f._setReadOnly(o, !b);  // readonly nếu không khớp
    if (!b) o.value = '';    // xóa giá trị nếu bị disable
  }
}
```

## 2. Ẩn/hiện cột trên Grid

### Từ Filter truyền hiddenFields sang Grid

```javascript
// Trong Filter (case 'Checking')
g._hiddenFields = [
  {Fields: 'ma_kho', Value: !(t.indexOf('S') > -1)},
  {Fields: 'ma_kh, ten_kh%l', Value: !(t.indexOf('C') > -1)},
  {Fields: 'nh_kh1, ten_nh_kh1%l', Value: !(t.indexOf('D') > -1)}
];
```

### Grid xử lý trong onRender

```javascript
function on$GridRender(sender, eventArgs) {
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

## 3. Thuộc tính filterSource trên field Dir

Khi field có `filterSource="Optional"`, field đó chỉ hiện khi điều kiện lọc cho phép:

```xml
<field name="ma_kho" isPrimaryKey="true" filterSource="Optional">
  <header v="Kho" e="Site"></header>
  <items style="AutoComplete" controller="Site" .../>
</field>
```

## Cơ chế hoạt động tổng thể

```
Filter (chọn loại) → response trả về xtype (VD: "SCDE,0")
  → Filter JS: ẩn/hiện field trên Filter, gán g._salesPriceType
  → Filter JS (Checking): gán g._hiddenFields cho Grid
  → Grid onRender: ẩn/hiện cột theo g._hiddenFields
  → Dir activeForm: ẩn/hiện field trên form theo g._salesPriceType
```
