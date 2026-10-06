# [SPEC_voucher-overview] — Tổng quan lập trình Chứng từ (Voucher)

> Dùng khi: tạo chức năng chứng từ (nhập liệu dạng Master/Detail có tách kỳ hoặc không tách kỳ).
> Đọc file này **đầu tiên** khi prompt yêu cầu làm chứng từ.
> Sau đó đọc từng file SPEC chi tiết theo loại file cần tạo.

---

## Phân biệt Danh mục vs. Chứng từ

| Tiêu chí | Danh mục | Chứng từ |
|---|---|---|
| Kiểu form | Đơn form / Master-Detail tĩnh | **Luôn** Master-Detail (form + grid detail) |
| Bảng | 1 bảng chính, optional detail | 4 bảng (c/m/d/i) hoặc 3 bảng (master/detail/inquiry) |
| Tách kỳ | Không | Có hoặc không |
| type trên `<grid>` / `<dir>` | _(không có)_ | `type="Voucher"` |
| Khóa chính | Mã do user nhập (`ma_xxx`) | `stt_rec` — hệ thống tự tăng |
| Số chứng từ | Không | `so_ct` — có hệ thống đánh số tự động |
| Mã chứng từ | Không | `ma_ct` — mã định nghĩa nghiệp vụ (VD: `PND`, `HDA`, `SX1`) |
| Entity chung | Ít | Rất nhiều (xem mục Entity bên dưới) |
| File cần tạo | aspx, Grid, Dir, (Filter, Lookup) | Grid Browser, Grid Detail, Dir, Filter, Report, (Upload) |

---

## Hai loại chứng từ

### Loại 1: Có phân kỳ (Partitioned) — Tham khảo IR (Phiếu nhập kho)

Bảng lưu trữ tách theo tháng dựa trên `ngay_ct`:

| Bảng | Prefix | Ví dụ | Mô tả |
|---|---|---|---|
| Chung | `cXX$000000` | `c74$000000` | Bảng tổng hợp chung (không tách kỳ), dùng để kiểm tra trùng số CT |
| Master | `mXX$yyyymm` | `m74$202601` | Bảng master, tách theo tháng |
| Detail | `dXX$yyyymm` | `d74$202601` | Bảng detail, tách theo tháng |
| Inquiry | `iXX$yyyymm` | `i74$202601` | Bảng tìm kiếm, chứa chuỗi filter |
| Cấu trúc | `mXX$000000` | `m74$000000` | Bảng cấu trúc (đuôi $000000) |

**Khai báo partition trong XML:**
```xml
<partition table="c74$000000" prime="m74$" inquiry="i74$"
  field="ngay_ct"
  expression="convert(char(6), {0}, 112)"
  increase="dateadd(month, 1, {0})"
  default="000000"/>
```

| Thuộc tính | Mô tả |
|---|---|
| `table` | Bảng tổng hợp chung (cXX$000000) |
| `prime` | Prefix bảng master (mXX$) |
| `inquiry` | Prefix bảng inquiry (iXX$) |
| `field` | Trường dùng làm căn cứ phân kỳ |
| `expression` | Công thức tính suffix (yyyymm) |
| `increase` | Quy luật tăng kỳ |
| `default` | Giá trị mặc định (000000 = bảng cấu trúc) |

### Loại 2: Không phân kỳ (Non-partitioned) — Tham khảo MO (Lệnh sản xuất)

| Bảng | Tên | Ví dụ | Mô tả |
|---|---|---|---|
| Master | `phXX` hoặc `mXX` | `phsx` | Bảng master (không có $ tách kỳ) |
| Detail | `ctXX` hoặc `dXX` | `ctsx` | Bảng detail |
| Inquiry | `iXX` | `isx` | Bảng tìm kiếm |
| Chung | _(không dùng)_ | | Không cần vì không tách kỳ |

**Khai báo partition (dummy — không tách kỳ):**
```xml
<partition table="isx" prime="phsx" inquiry="isx"
  field="ngay_ct"
  expression="''"
  increase="{0}"
  default=""/>
```

> `expression="''"` và `default=""` = không phân kỳ. Framework nhận diện và bỏ qua tách bảng.

---

## Trường chung của chứng từ

| Trường | Bảng | Mô tả |
|---|---|---|
| `stt_rec` | m, c, d, i | Khóa chính, char(13), hệ thống tự tăng |
| `stt_rec0` | d | Khóa thứ 2 của detail, char(3), đánh lại khi lưu |
| `line_nbr` | d | Số thứ tự dòng, đánh lại khi lưu |
| `ma_dvcs` | m, c, i | Mã đơn vị cơ sở |
| `ma_ct` | m, c, d, i | Mã chứng từ (VD: `PND`, `HDA`) |
| `so_ct` | m, c | Số chứng từ |
| `ngay_ct` | m, c, d | Ngày chứng từ (dùng tách kỳ) |
| `ngay_lct` | m | Ngày lập (mặc định = ngày hiện tại) |
| `ma_nt` | m | Mã ngoại tệ |
| `ty_gia` | m | Tỷ giá |
| `status` | m, c, i | Trạng thái chứng từ |
| `ma_gd` | m | Mã giao dịch (định nghĩa nghiệp vụ) |
| `dien_giai` | m | Diễn giải |
| Audit fields | m | `datetime0`, `datetime2`, `user_id0`, `user_id2` |

---

## Quy ước đặt tên Controller

| Thành phần | Quy tắc | Ví dụ IR | Ví dụ MO |
|---|---|---|---|
| Grid Browser | `{ma_ct}Tran` | `IRTran` | `MOTran` |
| Grid Detail | `{ma_ct}Detail` | `IRDetail` | `MODetail` |
| Dir | `{ma_ct}Tran` | `IRTran` | `MOTran` |
| Filter | `{ma_ct}Tran` | `IRTran` | `MOTran` |
| Report | `{ma_ct}Tran` | `IRTran` | _(nếu có)_ |
| Upload | `{ma_ct}Master` | `IRMaster` | _(nếu có)_ |
| ASPX | `zcct{ma_ct}` | `zcctIR` | `zcctMO` |

> Khi prompt sẽ cung cấp `ma_ct` cụ thể, Claude tự suy ra tên controller.

---

## Biến hệ thống (Session Variables)

Có sẵn trong SQL commands, dùng prefix `@@`:

| Biến | Mô tả |
|---|---|
| `@@id` | Mã chứng từ (VD: `PND`) |
| `@@unit` | Mã đơn vị hiện tại |
| `@@userID` | ID người dùng |
| `@@admin` | 1 = admin, 0 = user |
| `@@language` | `V` hoặc `E` |
| `@@action` | `New`, `Edit`, `View` |
| `@@master` | Tên bảng master (bảng cXX$000000) |
| `@@prime$partition$current` | Bảng master kỳ hiện tại (VD: `m74$202601`) |
| `@@prime$partition$previous` | Bảng master kỳ trước |
| `@@inquiry$partition$current` | Bảng inquiry kỳ hiện tại |
| `@@partition` | Thông tin partition |
| `@@expression` | Biểu thức partition |
| `@@sysDatabaseName` | Tên DB hệ thống |

---

## Biến SQL đặc biệt trong chứng từ

| Biến/Pattern | Mô tả |
|---|---|
| `$partition$current` | Suffix kỳ hiện tại (VD: `202601`) — dùng trong ENTITY |
| `$partition$previous` | Suffix kỳ trước |
| `#IF ... #THEN ... #ELSE ... #END` | Conditional SQL (framework xử lý trước khi exec) |
| `$dXX.NewValue`, `$dXX.OldValue` | So sánh detail grid có thay đổi không |
| `@d74` (table variable) | Biến bảng chứa dữ liệu detail grid gửi lên |

---

## Entity chung cho tất cả chứng từ

### Dir — Phải có (Entity bắt buộc)

```xml
<!-- XML requests chung -->
<!ENTITY XMLWhenVoucherInit SYSTEM "..\Include\XML\WhenVoucherInit.xml">
<!ENTITY XMLWhenVoucherNavigating SYSTEM "..\Include\XML\WhenVoucherNavigating.xml">
<!ENTITY XMLWhenVoucherCopying SYSTEM "..\Include\XML\WhenVoucherCopying.xml">
<!ENTITY XMLWhenVoucherClosing SYSTEM "..\Include\XML\WhenVoucherClosing.xml">
<!ENTITY XMLGetVoucherNumber SYSTEM "..\Include\XML\GetVoucherNumber.xml">
<!ENTITY XMLVoucherBookAndNumberFields SYSTEM "..\Include\XML\VoucherBookAndNumberFields.txt">

<!-- Commands chung -->
<!ENTITY CommandWhenVoucherLoading SYSTEM "..\Include\Command\WhenVoucherLoading.txt">
<!ENTITY CommandWhenVoucherBeforeEdit SYSTEM "..\Include\Command\WhenVoucherBeforeEdit.txt">
<!ENTITY CommandWhenVoucherBeforeDelete SYSTEM "..\Include\Command\WhenVoucherBeforeDelete.txt">
<!ENTITY CommandRecordHasBeenChanged SYSTEM "..\Include\Command\RecordHasBeenChanged.txt">
<!ENTITY CommandCheckVoucherHandleBeforeSave SYSTEM "..\Include\Command\CheckVoucherHandleBeforeSave.txt">
<!ENTITY CommandCheckVoucherHandleBeforeEdit SYSTEM "..\Include\Command\CheckVoucherHandleBeforeEdit.txt">
<!ENTITY CommandCheckVoucherHandleBeforeDelete SYSTEM "..\Include\Command\CheckVoucherHandleBeforeDelete.txt">
<!ENTITY CommandCheckLockedDate SYSTEM "..\Include\Command\CheckLockedDate.txt">
<!ENTITY CommandGetIdentityNumber SYSTEM "..\Include\Command\GetIdentityNumber.txt">
<!ENTITY CommandGetVoucherNumber SYSTEM "..\Include\Command\GetVoucherNumber.txt">
<!ENTITY CommandSetVoucherNumber SYSTEM "..\Include\Command\SetVoucherNumber.txt">
<!ENTITY CommandShowWarningMessage SYSTEM "..\Include\Command\ShowWarningMessage.txt">
<!ENTITY CommandQueryVoucherNumber SYSTEM "..\Include\Command\QueryVoucherNumber.txt">
<!ENTITY CommandScatterVoucherNumber SYSTEM "..\Include\Command\ScatterVoucherNumber.txt">

<!-- External fields (dùng cho InitExternalFields) -->
<!ENTITY CommandExternalFieldDeclare SYSTEM "..\Include\Command\ExternalFieldDeclare.txt">
<!ENTITY CommandExternalFieldSelect SYSTEM "..\Include\Command\ExternalFieldSelect.txt">
<!ENTITY CommandExternalFieldSet SYSTEM "..\Include\Command\ExternalVoucherFieldAssign.txt">
<!ENTITY CommandExternalFieldQuery SYSTEM "..\Include\Command\ExternalVoucherFieldQuery.txt">

<!-- Javascript chung -->
<!ENTITY ScriptVoucherInit SYSTEM "..\Include\Javascript\VoucherInit.txt">
<!ENTITY ScriptVoucherNumber SYSTEM "..\Include\Javascript\VoucherNumber.txt">
<!ENTITY VoucherNumberLoading SYSTEM "..\Include\Javascript\WhenVoucherNumberLoading.txt">
<!ENTITY VoucherNumberScattering SYSTEM "..\Include\Javascript\WhenVoucherNumberScattering.txt">
<!ENTITY VoucherNumberReading SYSTEM "..\Include\Javascript\WhenVoucherNumberReading.txt">
<!ENTITY ScriptActiveVoucher SYSTEM "..\Include\Javascript\ActiveVoucherDate.txt">
<!ENTITY ScriptScatterVoucher SYSTEM "..\Include\Javascript\ScatterVoucher.txt">
<!ENTITY ScriptCloseVoucher SYSTEM "..\Include\Javascript\CloseVoucher.txt">
```

### Dir — Entity tùy chọn

```xml
<!-- Ngoại tệ (nếu chứng từ có tiền tệ) -->
<!ENTITY XMLGetExchangeRate SYSTEM "..\Include\XML\GetExchangeRate.xml">
<!ENTITY ScriptCurrency SYSTEM "..\Include\Javascript\Currency.txt">
<!ENTITY CurrencyDateChanged SYSTEM "..\Include\Javascript\WhenCurrencyDateChanged.txt">
<!ENTITY CurrencyResponse SYSTEM "..\Include\Javascript\WhenCurrencyResponse.txt">

<!-- Kiểm tra phiếu nhập trước khi sửa/xóa (chứng từ kho) -->
<!ENTITY CommandCheckReceiptBeforeEdit SYSTEM "..\Include\Command\CheckReceiptBeforeEdit.txt">
<!ENTITY CommandCheckReceiptBeforeDelete SYSTEM "..\Include\Command\CheckReceiptBeforeDelete.txt">

<!-- Log sửa/xóa -->
<!ENTITY % VoucherEditLog SYSTEM "..\Include\VoucherEditLog.ent">
%VoucherEditLog;
<!ENTITY % VoucherDeleteLog SYSTEM "..\Include\VoucherDeleteLog.ent">
%VoucherDeleteLog;

<!-- Đính kèm file / Comment -->
<!ENTITY % Extender SYSTEM "..\Include\Extender.ent">
%Extender;
%Extender.Include.{Controller};
%Extender.Ignore;

<!-- VoucherEndUpdated & HandleVoucherNumber -->
<!ENTITY % VoucherEndUpdated SYSTEM "..\Include\VoucherEndUpdated.ent">
%VoucherEndUpdated;
<!ENTITY % HandleVoucherNumber SYSTEM "..\Include\HandleVoucherNumber.ent">
%HandleVoucherNumber;

<!-- VisibleField — ẩn hiện field -->
<!ENTITY VisibleFieldController "{Controller}">
<!ENTITY % VoucherVisibleField SYSTEM "..\Include\VoucherVisibleField.ent">
%VoucherVisibleField;
```

### Dir — Entity riêng từng chứng từ (thay đổi theo nghiệp vụ)

```xml
<!-- Biến detail (tên bảng detail variable và table) -->
<!ENTITY DetailVariable "@dXX">
<!ENTITY DetailTable "dXX$$partition$current">

<!-- AfterUpdate — cập nhật inquiry + grand + general -->
<!ENTITY AfterUpdate "
exec FastBusiness$App$Voucher$UpdateInquiryTable @@id, '@@inquiry$partition$current', '@@prime$partition$current', 'dXX$$partition$current', 'stt_rec', @stt_rec, @@operation
exec FastBusiness$App$Voucher$UpdateGrandTable @@id, '@@master', '@@prime$partition$current', 'stt_rec', @stt_rec, 1, &HandleVoucherNumberUpdateGrandTable;
exec FastBusiness$App$Voucher$UpdateGeneral @@id, 'mXX$$partition$current', 'dXX$$partition$current', '@@inquiry$partition$current', '@@prime$partition$current', @stt_rec">

<!-- Post, Stock, Delete — đặc thù nghiệp vụ -->
<!ENTITY Post "...">
<!ENTITY Stock "...">
<!ENTITY Delete "...">
```

### Grid Browser — Entity chung

```xml
<!ENTITY CommandWhenWhenVoucherBeforeInit SYSTEM "..\Include\Command\WhenVoucherBeforeInit.txt">
<!ENTITY CommandWhenWhenVoucherBeforeAddNew SYSTEM "..\Include\Command\WhenVoucherBeforeAddNew.txt">
<!ENTITY CommandWhenWhenVoucherAfterInit SYSTEM "..\Include\Command\WhenVoucherAfterInit.txt">
<!ENTITY XMLStandardVoucherToolbar SYSTEM "..\Include\XML\ExternalVoucherToolbar.xml">
<!ENTITY % PrintRight SYSTEM "..\Include\PrintRightGrid.ent">
%PrintRight;
<!ENTITY % Control.Filter SYSTEM "..\Include\Filter.ent">
%Control.Filter;
```

### Grid Detail — Entity chung

```xml
<!ENTITY % GridInitialize SYSTEM "..\Include\Grid.ent">
%GridInitialize;
<!ENTITY XMLGetUOMConversion SYSTEM "..\Include\XML\GetUOMConversion.xml">
<!ENTITY ScriptCheckGridAction SYSTEM "..\Include\Javascript\CheckGridAction.txt">
<!ENTITY ScriptEmptyExternalField SYSTEM "..\Include\Javascript\EmptyExternalField.txt">
```

### Filter — Entity chung

```xml
<!ENTITY XMLWhenFilterInit SYSTEM "..\Include\XML\WhenFilterInit.xml">
<!ENTITY XMLWhenFilterLoading SYSTEM "..\Include\XML\WhenFilterLoading.xml">
<!ENTITY XMLWhenFilterClosing SYSTEM "..\Include\XML\WhenFilterClosing.xml">
<!ENTITY ScriptFilterInit SYSTEM "..\Include\Javascript\FilterInit.txt">
```

### Report — Entity chung

```xml
<!ENTITY b SYSTEM ".\Include\BaseCurrency.xml">
<!ENTITY f SYSTEM ".\Include\ForeignCurrency.xml">
<!ENTITY s SYSTEM ".\Include\Separate.xml">
<!ENTITY p "../images/pdf.gif">
<!ENTITY e "../images/excel.gif">
<!ENTITY bi "../images/bilingual.png">
<!ENTITY be "../images/combine.png">
<!ENTITY GLTranReport SYSTEM ".\Include\GLTranReportBI.xml">
<!ENTITY GLTranReportSql SYSTEM ".\Include\GLTranReportSql.txt">
```

---

## Quy trình tạo chứng từ mới

1. Xác định loại: phân kỳ hay không phân kỳ
2. Nhận thông tin: `ma_ct`, số bảng (XX), tên chứng từ, các trường nghiệp vụ
3. Đọc `SPEC_voucher-grid.md` → tạo Grid Browser + Filter
4. Đọc `SPEC_voucher-detail.md` → tạo Grid Detail
5. Đọc `SPEC_voucher-dir.md` → tạo Dir (phức tạp nhất)
6. Đọc `SPEC_voucher-report.md` → tạo Report
7. (Nếu có Import) Tạo Upload — tham khảo `SPEC_import.md`
8. Tra Lookup theo quy trình: `SPEC_lookup.md` → `SPEC_lookup-other.md` → custom
