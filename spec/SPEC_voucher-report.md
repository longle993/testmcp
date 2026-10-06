# [SPEC_voucher-report] — Report XML cho Chứng từ (Mẫu in)

> Dùng khi: tạo file Report XML cho chức năng In chứng từ.
> Liên quan tới menu Print trên Grid Browser.
> Đọc `SPEC_voucher-overview.md` trước.

---

## Cấu trúc tổng thể

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE report [
  <!-- Entity chung cho Report -->
  <!ENTITY b SYSTEM ".\Include\BaseCurrency.xml">
  <!ENTITY f SYSTEM ".\Include\ForeignCurrency.xml">
  <!ENTITY s SYSTEM ".\Include\Separate.xml">
  <!ENTITY p "../images/pdf.gif">
  <!ENTITY e "../images/excel.gif">
  <!ENTITY bi "../images/bilingual.png">
  <!ENTITY be "../images/combine.png">
  <!ENTITY GLTranReport SYSTEM ".\Include\GLTranReportBI.xml">
  <!ENTITY GLTranReportSql SYSTEM ".\Include\GLTranReportSql.txt">

  <!-- Entity riêng từng chứng từ -->
  <!ENTITY Controller "{Controller}">
  <!ENTITY % Profile SYSTEM ".\Config\Profile.ent">
  %Profile;
  <!ENTITY % External SYSTEM ".\Config\{Controller}.ent">
  %External;
  <!ENTITY externalMasterDetail ", '&Master.Select;', '&Master.Join;', '&Detail.Select;', '&Detail.Join;'">
  <!ENTITY externalDetail ", '&Detail.Select;', '&Detail.Join;'">

  <!-- VisibleField -->
  <!ENTITY VisibleFieldController "{Controller}Print">
  <!ENTITY % VoucherVisibleField SYSTEM "..\Include\VoucherVisibleField.ent">
  %VoucherVisibleField;
]>

<report xmlns="urn:schemas-fast-com:data-report">
  <query>...</query>
  <forms>...</forms>
  <categories>...</categories>  <!-- nếu có nhóm mẫu -->
  <fields>...</fields>           <!-- parameter fields cho mẫu -->
</report>
```

---

## `<query>` — SQL lấy dữ liệu cho mẫu in

```xml
<query>
  <text>
    &Conditional.Unit.Profile.Query.Declare;
    &Conditional.Unit.Profile.Query.Select;
    <![CDATA[
if dbo.FastBusiness$Function$CheckSQLInjection(@stt_rec) = 0 return
]]>
    &VisibleFieldPrinting;
    &GLTranReportSql;<![CDATA[
else begin
    declare @m_ma_nt0 varchar(10)
    select @m_ma_nt0 = val from options where name = 'm_ma_nt0'
    -- Query lấy dữ liệu master
    select ...
      from @@prime$partition$current a with(nolock)
        left join {detail}$$partition$current b with(nolock) on ...
        -- join thêm bảng danh mục
      where a.stt_rec = @stt_rec

    -- Query lấy dữ liệu detail
    declare @key varchar(128)
    select @key = 'a.stt_rec = ''' + @stt_rec + ''''
    exec rs_Print{Controller} @@language, @key, '{detail}$$partition$current', @@id
end
]]>
    &Conditional.Unit.Profile.Query.Result;
  </text>
</query>
```

---

## `<forms>` — Khai báo mẫu in

### Các thuộc tính form

| Thuộc tính | Mô tả |
|---|---|
| `id` | Mã mẫu: `010`/`020` (Rpt), `110`/`120` (Excel) |
| `reportFile` | Tên file RPT (không có extension) |
| `templateFile` | Tên file Excel template (không có extension) |
| `commandArgument` | `"Pdf"` hoặc `"Excel"` |
| `urlImage` | Icon: `&p;` (PDF), `&e;` (Excel), `&bi;` (song ngữ), `&be;` (kết hợp) |
| `languageType` | `"0"` cho mẫu song ngữ |
| `controller` | Controller khác (nếu liên kết mẫu in từ chứng từ khác) |
| `externalID` | ID mẫu liên kết |

### Khai báo mẫu mặc định

Khi tạo Report mới, mặc định khai báo:

```xml
<forms>
  <!-- Mẫu RPT cơ bản (Pdf) -->
  <form id="010" reportFile="{Controller}_01" templateFile=""commandArgument="Pdf" urlImage="&p;">
    <header v="{Tên chứng từ}" e="{Voucher Name}"/>
    <download>
      <header v="{Tên chứng từ}" e="{Voucher Name}"/>
    </download>&b;
  </form>

  <!-- Mẫu RPT song ngữ (Pdf) -->
  <form id="020" reportFile="{Controller}_01BI" templateFile="" languageType="0" commandArgument="Pdf" urlImage="&bi;">
    <header v="{Tên} dạng song ngữ" e="{Name} - Bilingual Form"/>
    <download>
      <header v="{Tên} dạng song ngữ" e="{Name} - Bilingual Form"/>
    </download>
  </form>

  <!-- Mẫu Excel cơ bản -->
  <form id="110" templateFile="{Controller}" commandArgument="Excel" urlImage="&e;">
    <header v="{Tên chứng từ}" e="{Voucher Name}"/>
    <download>
      <header v="{Tên chứng từ}" e="{Voucher Name}"/>
    </download>&b;
  </form>

  <!-- Mẫu Excel song ngữ -->
  <form id="120" templateFile="{Controller}BI" languageType="0" commandArgument="Excel" urlImage="&be;">
    <header v="{Tên} dạng song ngữ" e="{Name} - Bilingual Form"/>
    <download>
      <header v="{Tên} dạng song ngữ" e="{Name} - Bilingual Form"/>
    </download>
  </form>

  &s;           <!-- Separate line -->
  &GLTranReport; <!-- Chứng từ hạch toán (nếu có post GL) -->
</forms>
```

### Form có parameter fields

```xml
<form id="012" reportFile="{Controller}_03" ...>
  <header v="..." e="..."/>
  <download><header v="..." e="..."/></download>
  <fields>
    <field name="isFC" type="Boolean">
      <header v="False" e="False"/>
    </field>
    <!-- Thêm parameter khác nếu cần -->
  </fields>
</form>
```

---

## `<categories>` — Nhóm mẫu in

```xml
<categories>
  <category index="12" length="9">
    <header v="Chứng từ hạch toán" e="General Ledger Voucher"/>
  </category>
</categories>
```

---

## `<fields>` — Parameter text cố định

Các trường text hiển thị trên mẫu in (header, label):

```xml
<fields>
  <field name="title" type="String">
    <header v="{TIÊU ĐỀ TIẾNG VIỆT}" e="{ENGLISH TITLE}"/>
  </field>
  <field name="h_nguoi_giao_hang" type="String">
    <header v="Họ và tên người giao hàng:" e="Deliverer's Full Name:"/>
  </field>
  <field name="h_stt" type="String">
    <header v="Stt" e="No."/>
  </field>
  <field name="h_ma_vt" type="String">
    <header v="Mã vật tư" e="Item Code"/>
  </field>
  <field name="h_ten_vt" type="String">
    <header v="Tên vật tư" e="Item Name"/>
  </field>
  <field name="h_dvt" type="String">
    <header v="Đvt" e="UOM"/>
  </field>
  <field name="h_so_luong" type="String">
    <header v="Số lượng" e="Quantity"/>
  </field>
  <field name="h_gia" type="String">
    <header v="Giá" e="Price"/>
  </field>
  <field name="h_tien" type="String">
    <header v="Tiền" e="Amount"/>
  </field>
  <field name="h_xac_nhan" type="String">
    <header v="Số tiền (viết bằng chữ):" e="Amount (in Words):"/>
  </field>
  <!-- Thêm fields theo layout mẫu in -->
</fields>
```

---

## Mẫu in Excel — Cấu trúc file template

File Excel template (`{Controller}.xlsx`) dùng các ký hiệu:

| Ký hiệu | Mô tả |
|---|---|
| `?field_name` | Lấy giá trị từ `<fields>` parameter (VD: `?title`) |
| `!1.column_name` | Lấy từ dataset master (row 1) (VD: `!1.so_ct`) |
| `!2.column_name` | Lấy từ dataset detail |
| `#?text` | Chuỗi cố định |

VD dòng đầu Excel: `?Entity_Line1` ... `!1.h_line1`

---

## Lưu ý khi tạo Report

1. File Report nằm trong thư mục `Controllers/Report/` (không phải Grid/ hay Dir/)
2. Entity `Controller` = tên controller Grid Browser (VD: `IRTran`)
3. File `.\Config\{Controller}.ent` chứa khai báo Master.Select, Master.Join, Detail.Select, Detail.Join cho customize
4. Stored procedure `rs_Print{Controller}` do nghiệp vụ riêng, Claude chỉ tạo khung gọi
5. Khi prompt không yêu cầu chi tiết mẫu in, tạo khung Report với 4 form mặc định (010, 020, 110, 120)
