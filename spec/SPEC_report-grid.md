# [SPEC_report-grid] — Grid XML cho Báo cáo

> Dùng khi: tạo file Grid XML hiển thị kết quả báo cáo.
> Thư mục: Controllers/Grid/{Controller}.xml
> Đọc `SPEC_report-overview.md` trước.

---

## Khác biệt so với Grid danh mục

| Đặc điểm | Grid danh mục | **Grid báo cáo** |
|---|---|---|
| Thẻ `<grid>` | `table`, `code`, `order` | **`type="Report"`, `valid`** |
| Nguồn dữ liệu | Bảng / View SQL | **Dataset từ Store (qua Filter Processing)** |
| Toolbar | CRUD (thêm/sửa/xóa) | **Chỉ xem + in (entity chuẩn)** |
| `valid` | Không | **`"systotal = 1"`** — chỉ tổng các dòng thỏa điều kiện |
| `filter` | Không | **Entity lọc dòng hiển thị** (tùy chọn) |
| Entity | `Controller`, lifecycle | **Report lifecycle (Loading/Closing), Toolbar, Init** |

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE grid [
  <!-- Entity chuẩn Report Grid -->
  <!ENTITY XMLWhenReportLoading SYSTEM "..\Include\XML\WhenReportLoading.xml">
  <!ENTITY XMLWhenReportClosing SYSTEM "..\Include\XML\WhenReportClosing.xml">
  <!ENTITY XMLStandardReportToolbar SYSTEM "..\Include\XML\StandardReportToolbar.xml">
  <!ENTITY JavascriptReportInit SYSTEM "..\Include\Javascript\ReportInit.txt">

  <!-- Entity tùy chọn -->
  <!ENTITY % ReferenceNumber SYSTEM "..\Include\ReferenceNumber.ent">
  %ReferenceNumber;
  <!ENTITY % Control.Filter SYSTEM "..\Include\Filter.ent">
  %Control.Filter;
  <!ENTITY % Repetition SYSTEM "..\Include\Repetition.ent">
  %Repetition;
]>

<grid type="Report" valid="systotal = 1" filter="{điều_kiện_lọc_dòng}" repetition="{điều_kiện_repetition}" xmlns="urn:schemas-fast-com:data-grid">
  <title v="{Tên báo cáo}" e="{Report Name}"/>
  <subTitle v="Từ ngày %d1 đến ngày %d2..." e="Date from %d1 to %d2..."/>
  <fields> ... </fields>
  <views> ... </views>
  <commands> ... </commands>
  <script> ... </script>
  &XMLStandardReportToolbar;
</grid>
```

---

## Thuộc tính `<grid>` cho báo cáo

| Thuộc tính | Bắt buộc | Mô tả |
|---|---|---|
| `type` | ✅ | Cố định: `"Report"` |
| `valid` | ✅ | Điều kiện tính tổng. Mặc định: `"systotal = 1"` — chỉ tổng dòng dữ liệu, bỏ dòng nhóm |
| `filter` | | Điều kiện lọc dòng hiển thị. VD: `"&Repetition.Key.001;"` |
| `repetition` | | Điều kiện repetition. VD: `"&Repetition.Key.002;"` |
| `xmlns` | ✅ | `urn:schemas-fast-com:data-grid` |

> **Không có** `table`, `code`, `order` — Grid báo cáo nhận dữ liệu từ Filter Processing.

### Về `valid`

`valid="systotal = 1"` nghĩa là: khi Grid tính aggregate (Sum, Count...), chỉ tính trên các dòng có `systotal = 1`. Dòng nhóm/header (systotal = 0) bị loại khỏi tổng.

### Về `filter` và `repetition`

Dùng entity từ `Repetition.ent` để hiện/ẩn dòng. Thường dùng khi báo cáo có outline (dòng nhóm + dòng chi tiết, toggle ẩn/hiện).

---

## Entity — Giải thích

| Entity | Mô tả | Bắt buộc |
|---|---|---|
| `XMLWhenReportLoading` | Command event Loading cho Report Grid | ✅ |
| `XMLWhenReportClosing` | Command event Closing cho Report Grid | ✅ |
| `XMLStandardReportToolbar` | Toolbar chuẩn (Freeze, Print, Export) | ✅ |
| `JavascriptReportInit` | JS khởi tạo Report Grid | ✅ |
| `ReferenceNumber` | Entity số tham chiếu (nếu báo cáo có số tham chiếu) | |
| `Control.Filter` | Entity filter cho Grid (allowFilter trên field số) | |
| `Repetition` | Entity cho outline / repetition | |

---

## `<fields>` — Khai báo cột

### Cú pháp field (giống Grid danh mục)

```xml
<field name="{tên_cột}" type="{kiểu}" width="{px}" dataFormatString="{format}" allowSorting="true" allowFilter="true" aggregate="{Sum|Count|...}">
  <header v="{Tiếng Việt}" e="{English}"/>
</field>
```

### Các kiểu format thường dùng

| Kiểu dữ liệu | `type` | `dataFormatString` |
|---|---|---|
| Ngày | `DateTime` | `@datetimeFormat` |
| Tiền VNĐ | `Decimal` | `@baseCurrencyAmountViewFormat` |
| Tiền ngoại tệ | `Decimal` | `@foreignCurrencyAmountViewFormat` |
| Tỷ giá | `Decimal` | `@exchangeRateViewFormat` |
| Số lượng | `Decimal` | `@quantityViewFormat` |
| Mã (uppercase) | (mặc định String) | `@upperCaseFormat` |

### Aggregate (tổng cột)

Thêm `aggregate="Sum"` trên field số cần tính tổng. Chỉ tổng dòng thỏa `valid`.

```xml
<field name="ps_no" type="Decimal" width="120" dataFormatString="@baseCurrencyAmountViewFormat" allowSorting="true" allowFilter="&GridReportAllowFilter.Number;" aggregate="Sum">
  <header v="Phát sinh nợ" e="Debit Amount"/>
</field>
```

> **`allowFilter="&GridReportAllowFilter.Number;"`**: entity cho phép lọc cột số trên Grid báo cáo.

### Field ẩn (dùng cho hyperlink, hệ thống)

```xml
<field name="stt_rec" width="0" hidden="true">
  <header v="" e=""/>
</field>
<field name="systotal" width="0" hidden="true">
  <header v="" e=""/>
</field>
```

### Field có Hyperlink (liên kết chứng từ)

```xml
<field name="ma_ct0" width="60" allowSorting="true" allowFilter="true" hyperlinkFormatString="~/AppHandler/Voucher.ashx Query: {Name: '[ma_ct]', Value: '[stt_rec]'}, Script: 'beforeDrillDown(this);'">
  <header v="Mã ct" e="VC. Code"/>
</field>
```

> Cần khai báo thêm field ẩn `ma_ct` và `stt_rec`.

### Field có Hyperlink xem chi tiết (drill-down)

```xml
<field name="ma_vt" width="100" allowSorting="true" allowFilter="true" hyperlinkFormatString="~/AppHandler/View.ashx Query: {Page: 'query_{page}', Controller: '{Controller}', Name: '[ma_vt]', Value: '[ma_vt] + this._queryFilterString'}, Script: 'beforeDrillDownWithCondition(this);'">
  <header v="Mã vật tư" e="Item Code"/>
</field>
```

> Dùng cho báo cáo có chi tiết. Cần thêm file query Grid riêng.

---

## `<views>` — Thứ tự cột hiển thị

```xml
<views>
  <view id="Grid">
    <field name="ngay_ct"/>
    <field name="ma_ct0"/>
    <field name="so_ct"/>
    <!-- ... các cột hiển thị ... -->
    <field name="ps_no"/>
    <field name="ps_co"/>
    <!-- Field ẩn đặt cuối -->
    <field name="ma_ct"/>
    <field name="stt_rec"/>
  </view>
</views>
```

---

## `<commands>` — Sự kiện

Grid báo cáo dùng entity chuẩn, **không viết command tùy chỉnh** (trừ trường hợp đặc thù):

```xml
<commands>
  &XMLWhenReportLoading;
  &XMLWhenReportClosing;
</commands>
```

---

## `<script>` — JavaScript

**Chuẩn (không tùy chỉnh):**

```xml
<script>
  <text>
    &JavascriptReportInit;
  </text>
</script>
```

**Có tùy chỉnh (VD: ẩn/hiện cột theo mẫu, drill-down):**

```xml
<script>
  <text>
    &JavascriptReportInit;
    <![CDATA[
function init$GridReport$(g) {
  // custom logic
}
]]>
  </text>
</script>
```

---

## Toolbar — Entity chuẩn

```xml
&XMLStandardReportToolbar;
```

> Toolbar chuẩn báo cáo gồm: Freeze, Print (PDF/Excel), Export. **Không có nút Add/Edit/Delete.**
> Nếu cần toolbar đặc thù, prompt sẽ ghi rõ.

---

## Ví dụ hoàn chỉnh (báo cáo đơn giản)

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE grid [
  <!ENTITY XMLWhenReportLoading SYSTEM "..\Include\XML\WhenReportLoading.xml">
  <!ENTITY XMLWhenReportClosing SYSTEM "..\Include\XML\WhenReportClosing.xml">
  <!ENTITY XMLStandardReportToolbar SYSTEM "..\Include\XML\StandardReportToolbar.xml">
  <!ENTITY JavascriptReportInit SYSTEM "..\Include\Javascript\ReportInit.txt">
]>

<grid type="Report" valid="systotal = 1" xmlns="urn:schemas-fast-com:data-grid">
  <title v="Bảng kê chứng từ" e="Transaction List"/>
  <subTitle v="Từ ngày %d1 đến ngày %d2..." e="Date from %d1 to %d2..."/>
  <fields>
    <field name="ngay_ct" type="DateTime" dataFormatString="@datetimeFormat" width="80" allowSorting="true" allowFilter="true">
      <header v="Ngày ct" e="VC. Date"/>
    </field>
    <field name="so_ct" width="80" dataFormatString="@upperCaseFormat" allowSorting="true" allowFilter="true" align="right">
      <header v="Số ct" e="Voucher No."/>
    </field>
    <field name="dien_giai" width="300" allowSorting="true" allowFilter="true">
      <header v="Diễn giải" e="Description"/>
    </field>
    <field name="ps_no" type="Decimal" width="120" dataFormatString="@baseCurrencyAmountViewFormat" allowSorting="true" aggregate="Sum">
      <header v="Phát sinh nợ" e="Debit Amount"/>
    </field>
    <field name="ps_co" type="Decimal" width="120" dataFormatString="@baseCurrencyAmountViewFormat" allowSorting="true" aggregate="Sum">
      <header v="Phát sinh có" e="Credit Amount"/>
    </field>
    <field name="systotal" width="0" hidden="true">
      <header v="" e=""/>
    </field>
  </fields>
  <views>
    <view id="Grid">
      <field name="ngay_ct"/>
      <field name="so_ct"/>
      <field name="dien_giai"/>
      <field name="ps_no"/>
      <field name="ps_co"/>
      <field name="systotal"/>
    </view>
  </views>
  <commands>
    &XMLWhenReportLoading;
    &XMLWhenReportClosing;
  </commands>
  <script>
    <text>
      &JavascriptReportInit;
    </text>
  </script>
  &XMLStandardReportToolbar;
</grid>
```

---

## Lưu ý quan trọng

1. **`type="Report"` bắt buộc** — phân biệt với Grid danh mục
2. **`valid="systotal = 1"`** — mặc định dùng `systotal` để phân biệt dòng dữ liệu (=1) và dòng nhóm (=0)
3. **Không có `table`, `code`, `order`** — Grid báo cáo không bind trực tiếp vào bảng
4. **`&XMLStandardReportToolbar;`** — đặt cuối cùng, **sau** thẻ `</grid>` KHÔNG ĐƯỢC, phải **trước** thẻ `</grid>`
5. **Field `systotal` ẩn**: nên khai báo trong fields + views dù hidden, để `valid` hoạt động
6. **`aggregate="Sum"`**: chỉ đặt trên field số cần tính tổng cuối Grid
