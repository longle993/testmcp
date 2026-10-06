# [SPEC_report-overview] — Tổng quan Báo cáo (Report)

> Đọc file này đầu tiên khi prompt yêu cầu "tạo báo cáo", "làm report".
> Báo cáo KHÁC với danh mục và chứng từ — đọc phần phân biệt bên dưới.

---

## Phân biệt Báo cáo vs Danh mục vs Chứng từ

| | Danh mục | Chứng từ | **Báo cáo** |
|---|---|---|---|
| **Mục đích** | Nhập liệu dữ liệu nền | Nhập liệu nghiệp vụ | **Hiển thị thông tin từ dữ liệu đã nhập** |
| **Dữ liệu** | Lưu vào DB | Lưu vào DB | **Chỉ đọc — không lưu DB** |
| **Dir (Form)** | Có — thêm/sửa/xóa | Có — thêm/sửa/xóa | **Không có** |
| **Filter** | Tùy chọn | Có | **Luôn có — hiện trước (FilterMode="true")** |
| **Grid** | Dữ liệu từ bảng/view | Dữ liệu từ bảng/view | **Dữ liệu từ Store (qua Processing)** |
| **Report (Mẫu in)** | Không | Có — query riêng khi in | **Có — dùng dataset sau khi lọc** |

---

## Thành phần báo cáo

| Thành phần | Thư mục | Tên file | Mô tả |
|---|---|---|---|
| **ASPX** | Pages/ | `{Controller}.aspx` | FilterMode="true" (mặc định) |
| **Filter** | Controllers/Filter/ | `{Controller}.xml` | Điều kiện lọc + Processing gọi Store |
| **Grid** | Controllers/Grid/ | `{Controller}.xml` | Hiển thị kết quả, type="Report" |
| **Report** | Controllers/Report/ | `{Controller}.xml` | Khai báo mẫu in (RPT + Excel) |

> **Không có file Dir** — báo cáo không có form thêm/sửa/xóa.

### Sơ đồ luồng dữ liệu

```
[Filter] → Processing (exec Store) → [Dataset trả về]
                                          ↓
                                    [Grid hiển thị]
                                          ↓
                                    [Report mẫu in] (dùng dataset đã có, không query lại)
```

---

## Quy ước đặt tên

### Controller
- Báo cáo: `rpt{TênBáoCáo}` — VD: `rptTransactionList`, `rptStockSummary`

### Store
- Dùng chung: `FastBusiness$Report${TênController}`
- Riêng: `rs_rpt{TênController}`
- HRM: `hs_rpt{TênController}`

### Tham số Store chuẩn
Store báo cáo luôn có 3 tham số hệ thống cuối: `@@language`, `@@userID`, `@@admin`.

### Quy ước prefix file share

| File gốc | Tên khi upload/download |
|---|---|
| `Controllers/Filter/rptXxx.xml` | `filter_rptXxx.xml` |
| `Controllers/Grid/rptXxx.xml` | `grid_rptXxx.xml` |
| `Controllers/Report/rptXxx.xml` | `report_rptXxx.xml` |
| `Pages/rptXxx.aspx` | `aspx_rptXxx.aspx` |

---

## Quy trình tạo báo cáo mới

1. Đọc file này (`SPEC_report-overview.md`) → hiểu tổng thể
2. Tạo ASPX: `SPEC_aspx-main.md` (template có Filter — FilterMode="true")
3. Đọc `SPEC_report-filter.md` → tạo `filter_rptXxx.xml`
4. Đọc `SPEC_report-grid.md` → tạo `grid_rptXxx.xml`
5. Đọc `SPEC_report-print.md` → tạo `report_rptXxx.xml`
6. Tra Lookup nếu cần: `SPEC_lookup.md` → `SPEC_lookup-other.md`
7. Đọc `SPEC_report-store.md` → viết Store procedure (hoặc tạo khung tạm)

---

## ASPX cho báo cáo

Báo cáo **luôn dùng** FilterMode="true" — hiện điều kiện lọc trước khi hiển thị Grid.

```aspx
<%@ Page AutoEventWireup="false" MasterPageFile="~/Main/MasterPage.master" Inherits="FastBusiness.ReportExtender.UI.Page" v="{Tên tiếng Việt}" e="{English Name}"%>
<asp:Content ID="headContent" ContentPlaceHolderID="head" runat="server"></asp:Content>
<asp:Content ID="mainContent" ContentPlaceHolderID="FastBusiness" runat="server">
    <div>
        <asp:Panel ID="panelReport" runat="server"/>
    </div>
    <FastBusiness:ReportExtender ID="MainReport" runat="server" TargetControlID="panelReport" ReadOnly="true" FilterMode="true" Controller="{Controller}"/>
</asp:Content>
```

---

## Store — Tổng quan

> Chi tiết viết Store báo cáo: xem **`SPEC_report-store.md`** ⭐

### Quy ước đặt tên Store

| Loại | Prefix | Ví dụ |
|---|---|---|
| Báo cáo chuẩn | `rs_rpt{Controller}` | `rs_rptStockSummary` |
| Customize | `zc_{controller}` | `zc_bctonkho` |
| HRM | `hs_rpt{Controller}` | `hs_rptSalary` |

### Dataset trả về

Store báo cáo thường trả về nhiều bảng (resultset):

| Bảng | Nội dung | Ghi chú |
|---|---|---|
| **Bảng 1** (index 0) | Thông tin điều kiện lọc | 1 dòng: `date_from`, `date_to`, ... |
| **Bảng 2** (index 1) | Dữ liệu báo cáo chính | Nhiều dòng, chứa `systotal`, `sysprint`, `sysorder` |
| Bảng 3+ | Dữ liệu phụ (nếu có) | Pivot, chi tiết phụ, ... |

### Các cột hệ thống trong bảng dữ liệu chính

| Cột | Kiểu | Mô tả |
|---|---|---|
| `sysorder` | int | Thứ tự sắp xếp. 0=đầu kỳ, 1=PS cộng/blank, 2=cuối kỳ, 3=blank, **5=dòng chi tiết** |
| `sysprint` | int | 1 = in, 0 = không in (dùng cho rowFilter mẫu Excel). Grid browse vẫn hiện |
| `systotal` | int | **1 = dòng dữ liệu** (tính tổng), 0 = dòng nhóm/header/tổng |

### Khung Store tối thiểu (khi chưa có logic)

```sql
CREATE PROCEDURE rs_rpt{TênController}
  @DateFrom SMALLDATETIME,
  @DateTo SMALLDATETIME,
  -- ... các tham số từ điều kiện lọc ...
  @Language CHAR(1),
  @UserID INT,
  @Admin BIT,
  @Controller VARCHAR(128) = '',
  @TableName VARCHAR(32) = '',
  @SysDatabase VARCHAR(32) = ''
AS
BEGIN
  SET NOCOUNT ON
  SET ANSI_NULLS OFF
  -- Bảng 1: thông tin điều kiện lọc
  SELECT @DateFrom AS date_from, @DateTo AS date_to
  -- Bảng 2: dữ liệu chính (tạm rỗng)
  SELECT 5 AS sysorder, 1 AS sysprint, 1 AS systotal,
    '' AS ma_field1, N'' AS ten_field1
  WHERE 1 = 0
  SET NOCOUNT OFF
  SET ANSI_NULLS ON
END
```

### Cần viết Store đầy đủ?

Đọc **`SPEC_report-store.md`** — bao gồm:
- Quy tắc đặt biến, kiểu dữ liệu, hoa/thường
- Xây dựng chuỗi điều kiện (`@Key`, `GetCheckKey`, phân quyền đơn vị)
- Lấy dữ liệu bảng phân kỳ (`%[bảng$]%`, `FastBusiness$Partition$Execute`)
- Xử lý kho hóa đơn vs kho thực tế
- Tính tồn/dư tại thời điểm bất kỳ (hàm `FastBusiness$Balance$*`)
- Thông tin các bảng post chính (r70$, r90$, r00$, cdvt, cdtk, cdkh)

---

## Phân loại báo cáo

| Loại | Đặc điểm | Tham khảo |
|---|---|---|
| **Đơn giản** | Lọc → hiển thị → in | Bảng kê chứng từ |
| **Có chi tiết (drill-down)** | Click dòng → xem chi tiết | Tổng hợp NXT |
| **Có chọn nhiều (check)** | Grid có checkbox, in nhiều dòng | In nhiều chứng từ |
| **Xoay (pivot)** | Dữ liệu xoay cột động | Tồn theo kho |

> Mặc định Claude tạo **báo cáo đơn giản**. Các loại đặc thù sẽ nêu rõ trong prompt.

---

## Lưu ý quan trọng

1. **Báo cáo không có Dir** — chỉ có Filter, Grid, Report
2. **FilterMode="true" là bắt buộc** — người dùng luôn thấy điều kiện lọc trước
3. **Mẫu in dùng dataset có sẵn** — Report XML không query lại DB (trừ trường hợp đặc thù)
4. **Store tạm**: khi prompt không cung cấp logic nghiệp vụ, tạo Store khung chỉ select rỗng
5. **Tab chữ ký + canh chỉnh**: mặc định **luôn có** (qua entity `&ReportSign...`, `&ReportMargin...`). Nếu không cần, prompt sẽ ghi rõ "không cần chữ ký"
6. **Viết Store đầy đủ**: xem `SPEC_report-store.md` — quy tắc SQL, phân kỳ, tồn/dư, bảng post
