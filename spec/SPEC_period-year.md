# [SPEC_period-year] — Field Kỳ (Period) và Năm (Year)

> Bổ sung cho `SPEC_dir.md`, `SPEC_filter.md`, `SPEC_grid.md` khi danh mục
> có field kỳ (tháng/kỳ) và năm.
>
> Hai trường hợp: **(A) Kỳ/năm là khóa chính** — phổ biến nhất (danh mục
> theo kỳ, kỳ kế toán). **(B) Kỳ/năm không phải khóa** — chỉ là
> trường dữ liệu thông thường.

---

## 1. Khai báo field `ky` hoặc `thang` và `nam`

### Khi là khóa chính (isPrimaryKey)

```xml
<field name="ky" isPrimaryKey="true" type="Decimal" dataFormatString="#0" allowNulls="false">
  <header v="Kỳ" e="Period"></header>
  <items style="Numeric"/>
</field>
<field name="nam" isPrimaryKey="true" type="Decimal" dataFormatString="###0" allowNulls="false">
  <header v="Năm" e="Year"></header>
  <items style="Numeric"/>
</field>
```

### Khi không phải khóa chính

Bỏ `isPrimaryKey="true"` và `allowNulls="false"` (nếu không bắt buộc):

```xml
<field name="ky" type="Decimal" dataFormatString="#0">
  <header v="Kỳ" e="Period"></header>
  <items style="Numeric"/>
</field>
<field name="nam" type="Decimal" dataFormatString="###0">
  <header v="Năm" e="Year"></header>
  <items style="Numeric"/>
</field>
```

### So sánh thuộc tính

| Thuộc tính | `ky` | `nam` | Ghi chú |
|---|---|---|---|
| `type` | `Decimal` | `Decimal` | Luôn là Decimal |
| `dataFormatString` | `"#0"` | `"###0"` | `#0` = tối đa 2 chữ số; `###0` = tối đa 4 chữ số |
| `<items style>` | `Numeric` | `Numeric` | Luôn dùng Numeric |
| `allowNulls` | `false` nếu PK | `false` nếu PK | |

---

## 2. Dir — khi kỳ/năm là khóa

### Layout

Kỳ/năm thường dùng layout `11` (Label + Input ngắn), không merge cột:

```xml
<!-- Ví dụ 7 cột: "120, 40, 60, 20, 80, 230, 0" -->
<item value="11-----: [ky].Label, [ky]"/>
<item value="11-----: [nam].Label, [nam]"/>
```

> Số `-` phụ thuộc tổng số cột của view. Các cột sau Label + Input đều để trống vì input ngắn.

### Disable và gán giá trị từ Grid (quan trọng)

Khi kỳ/năm là khóa và được truyền từ Filter → Grid → Dir, **người dùng không được tự nhập** — framework tự gán và disable:

```javascript
function init$MyController(f) {
  var g = f.grid;
  // Gán giá trị từ biến trên grid object (do Filter truyền)
  f.setItemValues('ky, nam', [g._ky, g._nam]);
  // Disable để user không sửa
  f.getItem('ky').disabled = true;
  f.getItem('nam').disabled = true;
  // Focus vào field đầu tiên người dùng cần nhập
  f.getItem('ma_xxx').focus();
}

// Gọi khi active (mở form mới) và khi scatter (load dữ liệu sửa)
function active$MyController(f) {
  f.add_onResponseComplete(on$MyController$ResponseComplete);
  init$MyController(f);
}
function scatter$MyController(f) { init$MyController(f); }
```

> **Lưu ý:** `init$...` được gọi cả ở `active$` (khi Mới) và `scatter$` (khi Sửa — event `Scattering`).
> Điều này đảm bảo giá trị kỳ/năm luôn được gán và disabled dù ở chế độ nào.

Commands cần thêm event `Scattering`:

```xml
<command event="Scattering">
  <text><![CDATA[select 'scatter$MyController(this);' as message
return]]></text>
</command>
```

### Composite key ENTITY

Khi kỳ/năm nằm trong composite key, khai báo 3 ENTITY chuẩn:

```xml
<!DOCTYPE dir [
  <!ENTITY k0 "@ky = $ky.OldValue and @nam = $nam.OldValue and @ma_xxx = $ma_xxx.OldValue">
  <!ENTITY k1 "ky = @ky and nam = @nam and ma_xxx = @ma_xxx">
  <!ENTITY k2 "ky = $ky.OldValue and nam = $nam.OldValue and ma_xxx = $ma_xxx.OldValue">
]>
```

- `k1` = điều kiện với giá trị **mới** (dùng trong Inserting, Updated)
- `k2` = điều kiện với giá trị **cũ** (dùng trong Updating để kiểm tra tồn tại)
- `k0` = so sánh mới vs cũ — nếu bằng nhau thì key không đổi (dùng trong Updating để bỏ qua check trùng khi key không thay đổi)

---

## 3. Filter — kỳ/năm là điều kiện lọc

### Khai báo field trong Filter

```xml
<field name="ky" isPrimaryKey="true" type="Decimal" dataFormatString="#0" allowNulls="false" aliasName="Period" defaultValue="(new Date()).getMonth() + 1">
  <header v="Kỳ" e="Period"></header>
  <items style="Numeric"/>
</field>
<field name="nam" isPrimaryKey="true" type="Decimal" dataFormatString="###0" allowNulls="false" aliasName="Year" defaultValue="(new Date()).getFullYear()">
  <header v="Năm" e="Year"></header>
  <items style="Numeric"/>
</field>
```

Điểm khác biệt so với Dir:

| Thuộc tính | Filter | Dir |
|---|---|---|
| `aliasName` | `"Period"` / `"Year"` | Không dùng |
| `defaultValue` | JS: tháng/năm hiện tại | Không có (do gán từ grid) |

> `aliasName` là tên định danh nội bộ framework dùng cho field lọc.
> `defaultValue` tự điền tháng hiện tại cho `ky` và năm hiện tại cho `nam` khi mở Filter.

### Filter JS — truyền kỳ/năm sang Grid qua ExternalKey

```javascript
case 'Checking':
  var g = f.grid, k = [];
  // Lưu vào grid object để Dir đọc lại
  g._ky  = f.getItem('ky').value;   // dùng .value, không phải getItemValue
  g._nam = f.getItem('nam').value;
  // Thêm vào externalKey — Type là 'Numeric' (khác String)
  if (g._ky)  Array.add(k, {Name: 'ky',  Opr: '=', Value: g._ky,  Type: 'Numeric', Ignore: false});
  if (g._nam) Array.add(k, {Name: 'nam', Opr: '=', Value: g._nam, Type: 'Numeric', Ignore: false});
  g.set_externalKey(k);
  // SubTitle hiển thị kỳ và năm
  g._alterTitle = [null, [['%s1', g._ky.toString(), true], ['%s2', g._nam.toString(), true]]];
  break;
```

> **Điểm quan trọng:** `Type: 'Numeric'` — khác hoàn toàn với String field (dùng `'String'`).
> Lấy giá trị bằng `f.getItem('ky').value` (`.value` trực tiếp), **không** dùng `f.getItemValue('ky')`.

---

## 4. Grid — kỳ/năm khi đã lọc qua Filter

### Khai báo field: ẩn hoàn toàn

Vì kỳ/năm đã được Filter lọc (externalKey), Grid không cần hiển thị → ẩn:

```xml
<field name="ky" isPrimaryKey="true" type="Decimal" width="0" hidden="true">
  <header v="Kỳ" e="Period"></header>
  <items style="Numeric"/>
</field>
<field name="nam" isPrimaryKey="true" type="Decimal" width="0" hidden="true">
  <header v="Năm" e="Year"></header>
  <items style="Numeric"/>
</field>
```

> Vẫn khai báo đầy đủ trong `<fields>` và `<views>` — cần thiết cho composite key — nhưng `width="0"` và `hidden="true"`.

### subTitle hiển thị kỳ năm

```xml
<subTitle v="Kỳ %s1, năm %s2..." e="Period %s1, Year %s2..."></subTitle>
```

- `%s1` = kỳ, `%s2` = năm — được gán từ Filter JS qua `g._alterTitle`.

---

## 5. Tổng kết — Điểm khác biệt so với field thông thường

| Điểm | Field thông thường | Field kỳ/năm |
|---|---|---|
| `type` | *(bỏ qua)* hoặc `DateTime` | Luôn `Decimal` |
| `dataFormatString` | `@upperCaseFormat` v.v. | `"#0"` (kỳ) / `"###0"` (năm) |
| `<items style>` | `Mask` / `AutoComplete` | `Numeric` |
| Filter `defaultValue` | Không có | `(new Date()).getMonth()+1` / `.getFullYear()` |
| Filter `aliasName` | Không có | `"Period"` / `"Year"` |
| Filter ExternalKey `Type` | `'String'` | `'Numeric'` |
| Filter lấy giá trị | `getItemValue(...)` | `getItem(...).value` |
| Grid | Hiển thị bình thường | `width="0"` + `hidden="true"` |
| Dir (khi PK) | User tự nhập | Gán từ `g._ky / g._nam`, disabled |
| Dir `scatter$` | Thường không cần | Cần thêm để re-gán kỳ/năm khi Sửa |

## 6. Checklist

- [ ] `type="Decimal"` cho cả `ky` và `nam`
- [ ] `dataFormatString="#0"` cho `ky`; `"###0"` cho `nam`
- [ ] `<items style="Numeric"/>` cho cả hai
- [ ] **Filter**: `aliasName="Period"/"Year"`, `defaultValue` dùng JS Date
- [ ] **Filter Loading**: query bảng kỳ → gán `_minYear/_maxYear/_minPeriod/_maxPeriod`
- [ ] **Filter Inserting**: kiểm tra khóa sổ (`dmstt` / tương đương)
- [ ] **Filter Checking**: validate range bằng JS (kiểm tra `_checked`)
- [ ] **Filter JS Checking**: `Type: 'Numeric'` trong ExternalKey; `.value` khi lấy giá trị
- [ ] **Grid**: `width="0"`, `hidden="true"` cho ky/nam nếu đã lọc qua Filter
- [ ] **Grid subTitle**: dùng `%s1`/`%s2` cho kỳ/năm
- [ ] **Dir (PK)**: thêm `scatter$` event + `init$` hàm chung để gán và disable
- [ ] **Dir (PK)**: `init$` gọi `f.setItemValues('ky, nam', [g._ky, g._nam])`
