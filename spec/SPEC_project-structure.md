# [SPEC_project-structure] — Cấu trúc cây thư mục Web/ (Project FBO)

> File này Claude đọc khi cần biết **một file XML / ASPX / Excel được sinh ra sẽ đặt vào đúng thư mục nào** trong hệ thống thực tế.
> Các SPEC khác (`SPEC_dir.md`, `SPEC_grid.md`, `SPEC_aspx-main.md`, …) chỉ ghi đường dẫn rút gọn (kiểu `Controllers/Grid/`). File này là bản đồ đầy đủ.
>
> ⚠️ **Cảnh báo discrepancy**: Một số SPEC ghi sai số nhiều/số ít so với cây thực tế (xem mục [Discrepancy cần biết](#discrepancy-cần-biết-spec--cây-thực-tế) bên dưới). Ưu tiên đường dẫn trong file này.

---

## Toàn cảnh Web/

```
Web/
├── AppHandler/              ← Các .ashx (Import.ashx, Voucher.ashx, View.ashx, ...) — handler runtime
├── AppService/              ← Các service backend (.asmx / .svc)
├── App_Data/
│   ├── Controllers/         ← ⭐ TRUNG TÂM — toàn bộ XML khai báo màn hình
│   ├── Download/            ← Output framework sinh ra cho user tải về
│   │   ├── PDF/
│   │   └── Zip/
│   └── Upload/              ← Buffer file user upload lên (trước khi xử lý)
│       ├── PDF/
│       └── XML/
├── ClientScript/            ← JavaScript chạy phía browser
│   ├── External/            ← Lib thứ ba (Calendar, Chart, PDF, QRCode, j0, j1)
│   └── Home/                ← Script riêng của Home/Dashboard
├── Css/                     ← Stylesheet
│   └── External/            ← CSS lib thứ ba (Calendar, c1)
├── Help/
│   └── Vie/                 ← File HTML hướng dẫn (tiếng Việt)
├── Images/                  ← Tài nguyên ảnh
│   ├── Flow/                ← Hình minh hoạ luồng nghiệp vụ
│   ├── Menu/                ← Icon menu
│   ├── Report/              ← Logo / ảnh dùng trong mẫu in
│   └── Tree/                ← Icon cây danh mục
├── Main/                    ← ⭐ Các file .aspx (trang web) + MasterPage.master
├── Upload/
│   └── Transfer/            ← Khu vực truyền file giữa các phân hệ
│       ├── Export/
│       └── Import/
└── bin/                     ← DLL biên dịch (FastBusiness.dll, ReportExtender.dll, …)
```

**Chỉ 4 thư mục mà Claude sinh file trực tiếp vào**: `Main/`, `App_Data/Controllers/<loại>/`, `App_Data/Controllers/Templates/Excel/`, và `App_Data/Controllers/Templates/Rpt/`. Mọi thư mục còn lại là hệ thống / framework / runtime.

---

## App_Data/Controllers/ — Trung tâm khai báo

Đây là nơi đặt **mọi XML** khai báo màn hình. Mỗi loại file có thư mục riêng.

```
App_Data/Controllers/
├── Allocation/              ← XML khai báo phân bổ (allocation rule)
├── BankHub/                 ← XML tích hợp ngân hàng điện tử
├── Chat/                    ← XML cho module chat / messaging
├── Config/
│   └── Export/              ← Cấu hình export
├── Dashboard/
│   ├── Objects/             ← Component dashboard (chart, KPI tile, ...)
│   └── Template/            ← Mẫu dashboard
├── Dir/                     ← ⭐ XML form nhập/sửa/xóa  (dir_*.xml)
│   ├── Config/
│   │   └── Fields/          ←   Field-level config dùng chung
│   └── View/                ←   Variant view của Dir
├── EInvoice/                ← XML hóa đơn điện tử
├── Filter/                  ← ⭐ XML khai báo bộ lọc  (filter_*.xml)
│   └── Config/
│       └── Fields/
├── Flow/                    ← XML workflow / luồng phê duyệt
├── Grid/                    ← ⭐ XML danh sách + grid chi tiết  (grid_*.xml)
│   └── Config/
│       ├── Arrangement/     ←   Sắp xếp / nhóm cột
│       ├── Detail/          ←   Cấu hình grid-detail
│       ├── Fields/          ←   Field-level config dùng chung
│       └── Include/
├── Include/                 ← ⭐⭐ ENTITY chia sẻ (Javascript, XML, Command, ...)
│   ├── A001.TT88/           ←   Tham chiếu thông tư cụ thể
│   ├── Circular/            ←   Theo thông tư (TT88, TT132)
│   │   ├── A001.TT88/
│   │   └── A002.TT132/
│   ├── Clipboard/
│   ├── Command/             ←   ENTITY các <command> dùng chung
│   ├── Config/
│   ├── Extra/               ←   ENTITY %Extra (mở rộng)
│   ├── Irregular/           ←   Logic không định kỳ
│   ├── Javascript/          ←   ⭐ ENTITY mã JS (Irregular.txt, Suggestion.txt, DownloadScript.txt, UploadScript.txt, …)
│   ├── Nested/              ←   ENTITY nested grid
│   ├── Standard/            ←   ENTITY chuẩn
│   │   └── XML/
│   └── XML/                 ←   ⭐ ENTITY XML snippet (Suggestion.xml, WhenFilterLoading.xml, UploadField.txt, …)
│       ├── Circular/
│       │   ├── A01119/  A0126/  A02151/  A03200/  A04195/  A0592/
│       │   ├── A06133/  A0728/  A08200/  A09123/  A10132/
│       │   ├── A1080/   A1115/
│       │   └── Config/
│       └── Config/
│           └── Fields/
├── List/                    ← XML list view (dạng đặc biệt của Grid)
├── Lookup/                  ← ⭐ XML lookup / autocomplete  (lookup_*.xml)
│   └── Config/
│       └── Include/
├── Media/                   ← XML media viewer
├── Notify/                  ← XML thông báo
├── Options/                 ← XML options dialog
│   └── Config/
│       └── Actions/
├── Post/                    ← XML hậu xử lý / hạch toán
│   └── Convert/             ←   Convert giữa các định dạng
├── Query/                   ← ⭐ XML query (drill-down, View.ashx)
│   ├── Template/
│   └── XML/
├── Report/                  ← ⭐ XML khai báo mẫu in báo cáo  (report_*.xml)
│   ├── Config/
│   ├── External/
│   └── Include/
├── Request/                 ← XML request handler
├── Structure/               ← Schema khai báo các loại XML (meta)
│   ├── App/
│   ├── Dir/                 ←   Schema validate file dir_*.xml
│   ├── Filter/              ←   Schema validate file filter_*.xml
│   ├── Grid/                ←   Schema validate file grid_*.xml
│   ├── Lookup/              ←   Schema validate file lookup_*.xml
│   └── Sys/
├── Templates/               ← ⭐ Template file in
│   ├── Clipboard/
│   │   └── Config/
│   ├── Excel/               ← ⭐⭐ File .xlsx template báo cáo  ({templateFile}.xlsx)
│   │   └── External/
│   ├── Html/                ←   Template HTML cho preview / email
│   │   └── External/
│   ├── Rpt/                 ← ⭐ File .rpt template (Crystal Report)
│   │   └── External/
│   ├── Upload/              ← ⭐ XML khai báo import Excel  (upload_*.xml)
│   │   ├── Config/
│   │   │   ├── Detail/
│   │   │   ├── Include/
│   │   │   │   ├── Detail/  Master/  Nested/
│   │   │   └── Master/
│   │   ├── Include/
│   │   │   └── Nested/
│   │   └── Invoice/         ←   Template import hóa đơn
│   └── Xml/
│       └── Config/
└── View/                    ← XML cấu hình View (drill-down)
```

---

## Bảng mapping: Claude sinh file gì → đặt ở đâu

| Loại file Claude sinh (prefix khi share) | Đường dẫn thực tế trong Web/ |
|---|---|
| `aspx_{Tên}.aspx` | `Web/Main/{Tên}.aspx` |
| `dir_{Controller}.xml` | `Web/App_Data/Controllers/Dir/{Controller}.xml` |
| `grid_{Controller}.xml` | `Web/App_Data/Controllers/Grid/{Controller}.xml` |
| `filter_{Controller}.xml` | `Web/App_Data/Controllers/Filter/{Controller}.xml` |
| `lookup_{Controller}.xml` | `Web/App_Data/Controllers/Lookup/{Controller}.xml` |
| `upload_{Controller}.xml` | `Web/App_Data/Controllers/Templates/Upload/{Controller}.xml` |
| `report_{Controller}.xml` | `Web/App_Data/Controllers/Report/{Controller}.xml` |
| `{templateFile}.xlsx` (mẫu in Excel) | `Web/App_Data/Controllers/Templates/Excel/{templateFile}.xlsx` |
| `{templateFile}.rpt` (mẫu in Crystal) | `Web/App_Data/Controllers/Templates/Rpt/{templateFile}.rpt` |

> Khi prompt nói "sinh đầy đủ bộ file cho danh mục X" → Claude tạo các file trên với prefix tương ứng vào `/mnt/user-data/outputs/`. Người dùng đổi tên (bỏ prefix) và copy vào thư mục thực ở cột phải.

---

## Đường dẫn tương đối trong ENTITY (DOCTYPE)

Các XML reference nhau qua ENTITY với **đường dẫn tương đối**. Tùy file XML đang nằm ở đâu mà số `..\` thay đổi.

### Từ Dir/ , Grid/ , Filter/ , Lookup/ , Report/  (1 cấp dưới Controllers/)

```xml
<!-- Một cấp lên: ..\ = App_Data/Controllers/ -->
<!ENTITY ScriptIrregular  SYSTEM "..\Include\Javascript\Irregular.txt">
<!ENTITY XMLSuggestion    SYSTEM "..\Include\XML\Suggestion.xml">
<!ENTITY ScriptSuggestion SYSTEM "..\Include\Javascript\Suggestion.txt">
<!ENTITY DowloadScript    SYSTEM "..\Include\Javascript\DownloadScript.txt">
<!ENTITY UploadCommand    SYSTEM "..\Include\Command\UploadCommand.txt">
<!ENTITY % ExportImportTemplate SYSTEM "..\Include\ExportImportTemplate.ent">
```

Giải mã: file ở `Controllers/Dir/X.xml` → `..\Include\Javascript\Irregular.txt` resolves thành `Controllers/Include/Javascript/Irregular.txt`. ✓

### Từ Templates/Upload/  (2 cấp dưới Controllers/)

```xml
<!-- Hai cấp lên: ..\..\ = App_Data/Controllers/ -->
<!ENTITY % ImportErrorMode      SYSTEM "..\..\Include\ImportErrorMode.ent">
<!ENTITY % ExportImportTemplate SYSTEM "..\..\Include\ExportImportTemplate.ent">
<!ENTITY % ListEditLog          SYSTEM "..\..\Include\ListEditLog.ent">
<!ENTITY % Tiny.External        SYSTEM "..\..\Include\Tiny.External.ent">

<!-- Một cấp lên: ..\ = Templates/ , hoặc cùng cấp: Include\ = Templates/Upload/Include/ -->
<!ENTITY IrregularValue SYSTEM "Include\Irregular.txt">
```

> **Quy tắc**: Đếm số cấp từ file hiện tại lên đến `App_Data/Controllers/`. Đó là số `..\` cần dùng để vào `Include/`.

---

## Discrepancy cần biết (SPEC ↔ cây thực tế)

Một vài SPEC đang viết đường dẫn **rút gọn / sai số nhiều-số ít**. Khi copy file vào hệ thống thực, dùng cột "Thực tế" bên dưới.

| Đang ghi trong SPEC | Cây thực tế (đúng) | File SPEC bị ảnh hưởng |
|---|---|---|
| `Pages/` | `Main/` | `00_INDEX.md` (bảng "Thành phần báo cáo"), `SPEC_aspx-main.md` (line 95) |
| `App_Data/Controller/Template/Excel/` | `App_Data/Controllers/Templates/Excel/` | `SPEC_report-excel.md`, `SPEC_report-print.md`, `00_INDEX.md` |
| `Controllers/Upload/` (cho file `upload_*.xml`) | `App_Data/Controllers/Templates/Upload/` | `00_INDEX.md`, `SPEC_import.md` |
| `Controllers/Grid/` | `App_Data/Controllers/Grid/` (thêm tiền tố `App_Data/`) | Mọi SPEC tham chiếu rút gọn |

> Master page tham chiếu là `~/Main/MasterPage.master` — đúng với thư mục `Main/`. Đây là bằng chứng `Main/` là nơi đặt aspx, không phải `Pages/`.

---

## Thư mục không sinh file trực tiếp (chỉ tham khảo)

| Thư mục | Vai trò | Có cần đụng tới? |
|---|---|---|
| `AppHandler/` | `.ashx` xử lý request (`Import.ashx`, `Voucher.ashx`, `View.ashx`) | KHÔNG — chỉ tham chiếu trong XML qua `~/AppHandler/{X}.ashx` |
| `AppService/` | `.asmx` / `.svc` backend | KHÔNG |
| `bin/` | DLL biên dịch | KHÔNG |
| `ClientScript/` | JS framework | KHÔNG (script custom thường nhúng trong XML, không tạo file riêng) |
| `Css/` | Stylesheet | KHÔNG |
| `Help/Vie/` | HTML hướng dẫn | Có thể bổ sung khi prompt yêu cầu trợ giúp |
| `Images/Report/` | Logo trong mẫu in | Có thể đặt logo khi làm Excel template |
| `Structure/` | Schema validate XML (meta) | KHÔNG — framework duy trì |
| `App_Data/Download/`, `App_Data/Upload/` | Buffer runtime | KHÔNG — runtime tự tạo |
| `Upload/Transfer/` | Buffer chuyển dữ liệu | KHÔNG |

---

## Quy trình tra cứu nhanh khi sinh file

1. **Nhận prompt** → xác định loại file cần tạo (dir / grid / filter / lookup / upload / report / aspx / xlsx).
2. **Tra bảng "Claude sinh file gì → đặt ở đâu"** ở trên → biết thư mục đích thực tế.
3. **Đếm `..\`** trong ENTITY: file đang ở mấy cấp dưới `App_Data/Controllers/`? → dùng đúng số `..\`.
4. **Sinh file** vào `/mnt/user-data/outputs/` với prefix (`dir_`, `grid_`, …) theo quy ước share.
5. Ghi chú trong response: "File này khi copy về hệ thống đặt tại `{đường_dẫn_thực_tế}`".

---

## Liên kết

- `00_INDEX.md` — file định hướng tổng
- `SPEC_aspx-main.md` — ASPX → `Main/`
- `SPEC_dir.md` , `SPEC_grid.md` , `SPEC_filter.md` , `SPEC_lookup.md` — XML khai báo → `Controllers/<loại>/`
- `SPEC_import.md` — upload XML → `Controllers/Templates/Upload/`
- `SPEC_report-excel.md` — file `.xlsx` → `Controllers/Templates/Excel/`
- `SPEC_report-print.md` — file `.rpt` / report XML → `Controllers/Report/` + `Controllers/Templates/Rpt/`
