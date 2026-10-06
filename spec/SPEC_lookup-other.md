# [SPEC_lookup-other] — Lookup khác (Kiểm tra sau danh sách thường dùng trong SPEC_lookup)

> File này chứa các lookup **ít xuất hiện hơn**, chỉ dùng trong một số module nhất định.
> **Chỉ kiểm tra file này khi `SPEC_lookup.md` không có** controller cần tìm.

---

## Danh sách Lookup khác

| Controller | Bảng | Mã (`code`) | Tên (`name`) | Ghi chú |
|---|---|---|---|---|
| `SalesPerson` | dmnvbh | ma_nvbh | ten_nvbh%l | Nhân viên bán hàng |
| `Employee` | dmnv | ma_nv | ten_nv%l | Nhân viên |
| `Bank` | dmnh | ma_nh | ten_nh%l | Ngân hàng |
| `Discount` | dmck | ma_ck | ten_ck%l | Chiết khấu |
| `Tax` | dmthue | ma_thue | ten_thue%l | Thuế |
| `CustomerGroup` | dmnhkh | ma_nh | ten_nh%l | Nhóm khách hàng (lọc `loai_nh`) |
| `ItemGroup` | dmnhvt | ma_nh | ten_nh%l | Nhóm vật tư (lọc `loai_nh`) |
| `JobGroup` | dmnhvv | ma_nh | ten_nh%l | Nhóm vụ việc (lọc `loai_nh`) |
| `SalesPriceType` | dmloaigia2 | loai_gia | ten_loai%l | Loại giá bán |
| `CustomerPriceClass` | dmnhgia | ma_nhgia | ten_nhgia%l | Nhóm giá khách hàng |
| `UOMItemExtension` | dmdvt_vt | dvt | ten_dvt%l | ĐVT mở rộng theo vật tư |
| `PaymentTerm` | dmhttt | ma_httt | ten_httt%l | Hình thức thanh toán |
| `BankAccount` | dmtk_nh | stk | ten_stk%l | Tài khoản ngân hàng |
| `CostCenter` | dmttcp | ma_ttcp | ten_ttcp%l | Trung tâm chi phí |

---

## XML mẫu các Lookup khác

### SalesPerson

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmnvbh" code="ma_nvbh" name="ten_nvbh%l" order="ma_nvbh"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục nhân viên bán hàng" e="Salesperson List"></header>
  <fields>
    <field name="ma_nvbh" allowSorting="true" allowFilter="true">
      <header v="Mã NV" e="Salesperson ID"></header>
    </field>
    <field name="ten_nvbh%l" allowSorting="true" allowFilter="true">
      <header v="Tên nhân viên" e="Salesperson Name"></header>
    </field>
  </fields>
</lookup>
```

### Employee

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmnv" code="ma_nv" name="ten_nv%l" order="ma_nv"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục nhân viên" e="Employee List"></header>
  <fields>
    <field name="ma_nv" allowSorting="true" allowFilter="true">
      <header v="Mã NV" e="Employee ID"></header>
    </field>
    <field name="ten_nv%l" allowSorting="true" allowFilter="true">
      <header v="Tên nhân viên" e="Employee Name"></header>
    </field>
  </fields>
</lookup>
```

### Bank

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmnh" code="ma_nh" name="ten_nh%l" order="ma_nh"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục ngân hàng" e="Bank List"></header>
  <fields>
    <field name="ma_nh" allowSorting="true" allowFilter="true">
      <header v="Mã NH" e="Bank ID"></header>
    </field>
    <field name="ten_nh%l" allowSorting="true" allowFilter="true">
      <header v="Tên ngân hàng" e="Bank Name"></header>
    </field>
  </fields>
</lookup>
```

### Discount

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmck" code="ma_ck" name="ten_ck%l" order="ma_ck"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục chiết khấu" e="Discount List"></header>
  <fields>
    <field name="ma_ck" allowSorting="true" allowFilter="true">
      <header v="Mã CK" e="Discount ID"></header>
    </field>
    <field name="ten_ck%l" allowSorting="true" allowFilter="true">
      <header v="Tên chiết khấu" e="Discount Name"></header>
    </field>
  </fields>
</lookup>
```

### Tax

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmthue" code="ma_thue" name="ten_thue%l" order="ma_thue"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Danh mục thuế" e="Tax List"></header>
  <fields>
    <field name="ma_thue" allowSorting="true" allowFilter="true">
      <header v="Mã thuế" e="Tax ID"></header>
    </field>
    <field name="ten_thue%l" allowSorting="true" allowFilter="true">
      <header v="Tên thuế" e="Tax Name"></header>
    </field>
  </fields>
</lookup>
```

### CustomerGroup (nhóm khách hàng — lọc theo loai_nh)

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmnhkh" code="ma_nh" name="ten_nh%l" order="ma_nh"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Nhóm khách hàng" e="Customer Group List"></header>
  <fields>
    <field name="ma_nh" allowSorting="true" allowFilter="true">
      <header v="Mã nhóm" e="Group ID"></header>
    </field>
    <field name="ten_nh%l" allowSorting="true" allowFilter="true">
      <header v="Tên nhóm" e="Group Name"></header>
    </field>
  </fields>
</lookup>
```

> **Cách dùng trong Dir** — lọc theo `loai_nh`:
> ```xml
> <items style="AutoComplete" controller="CustomerGroup" reference="ten_nh%l"
>   key="loai_nh = 1 and status = '1'" check="1 = 1"
>   information="ma_nh$dmnhkh.ten_nh%l" new="Default"/>
> ```
> Thay `loai_nh = 1` thành `loai_nh = 2`, `3`,... tùy field.

### ItemGroup (nhóm vật tư — lọc theo loai_nh)

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmnhvt" code="ma_nh" name="ten_nh%l" order="ma_nh"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Nhóm vật tư" e="Item Group List"></header>
  <fields>
    <field name="ma_nh" allowSorting="true" allowFilter="true">
      <header v="Mã nhóm" e="Group ID"></header>
    </field>
    <field name="ten_nh%l" allowSorting="true" allowFilter="true">
      <header v="Tên nhóm" e="Group Name"></header>
    </field>
  </fields>
</lookup>
```

### JobGroup (nhóm vụ việc — lọc theo loai_nh)

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmnhvv" code="ma_nh" name="ten_nh%l" order="ma_nh"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Nhóm vụ việc" e="Job Group List"></header>
  <fields>
    <field name="ma_nh" allowSorting="true" allowFilter="true">
      <header v="Mã nhóm" e="Group ID"></header>
    </field>
    <field name="ten_nh%l" allowSorting="true" allowFilter="true">
      <header v="Tên nhóm" e="Group Name"></header>
    </field>
  </fields>
</lookup>
```

### SalesPriceType

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmloaigia2" code="loai_gia" name="ten_loai%l" order="loai_gia"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Loại giá bán" e="Sales Price Type List"></header>
  <fields>
    <field name="loai_gia" allowSorting="true" allowFilter="true">
      <header v="Loại giá" e="Price Type"></header>
    </field>
    <field name="ten_loai%l" allowSorting="true" allowFilter="true">
      <header v="Tên loại giá" e="Price Type Name"></header>
    </field>
  </fields>
</lookup>
```

### PaymentTerm

```xml
<?xml version="1.0" encoding="utf-8"?>
<lookup table="dmhttt" code="ma_httt" name="ten_httt%l" order="ma_httt"
        xmlns="urn:schemas-fast-com:data-lookup">
  <header v="Hình thức thanh toán" e="Payment Term List"></header>
  <fields>
    <field name="ma_httt" allowSorting="true" allowFilter="true">
      <header v="Mã HTTT" e="Payment Term ID"></header>
    </field>
    <field name="ten_httt%l" allowSorting="true" allowFilter="true">
      <header v="Tên HTTT" e="Payment Term Name"></header>
    </field>
  </fields>
</lookup>
```

---

## Lookup nhóm danh mục — Pattern chung (loai_nh)

Các bảng nhóm (`dmnhkh`, `dmnhvt`, `dmnhvv`...) dùng chung cấu trúc:

| Cột | Ý nghĩa |
|---|---|
| `loai_nh` | Loại nhóm (1, 2, 3...) — mỗi giá trị là 1 nhóm riêng |
| `ma_nh` | Mã nhóm |
| `ten_nh` / `ten_nh2` | Tên nhóm |

**Cùng 1 file lookup** — phân biệt bằng `key` trên `<items>`:

```xml
<!-- Nhóm KH loại 1 -->
<items style="AutoComplete" controller="CustomerGroup" reference="ten_nh%l"
  key="loai_nh = 1 and status = '1'" .../>

<!-- Nhóm KH loại 2 -->
<items style="AutoComplete" controller="CustomerGroup" reference="ten_nh%l"
  key="loai_nh = 2 and status = '1'" .../>

<!-- Nhóm KH loại 3 -->
<items style="AutoComplete" controller="CustomerGroup" reference="ten_nh%l"
  key="loai_nh = 3 and status = '1'" .../>
```

Tương tự cho `ItemGroup`, `JobGroup`, và các bảng nhóm khác.
