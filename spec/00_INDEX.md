# [00_INDEX] — Knowledge Base: FBO Framework

> File này Codex đọc đầu tiên để định hướng dùng file nào.
> Hệ thống Fast Business Online (FBO) — Web-based ERP framework.
> **Ba nhóm chính:** Danh mục (catalog), Chứng từ (voucher), và Báo cáo (report). Khi prompt nói "làm chứng từ" → đọc `SPEC_voucher-*`. Khi nói "tạo báo cáo" → đọc `SPEC_report-*`.

---

## Tổng quan hệ thống

Framework FBO xây dựng màn hình qua khai báo XML. Mỗi chức năng danh mục gồm:

| Thành phần | Thư mục gốc | Mục đích |
|---|---|---|
| File ASPX (Main) | Main/ | Trang web, khai báo Controller |
| File XML Grid | Controllers/Grid/ | Hiển thị danh sách (Browse) |
| File XML Dir | Controllers/Dir/ | Form thêm/sửa/xóa |
| File XML Filter | Controllers/Filter/ | Điều kiện lọc (chỉ khi cần) |
| File XML Lookup | Controllers/Lookup/ | Popup tra cứu / AutoComplete |
| File XML Upload | Controllers/Templates/Upload/ | Khai báo template Excel + SQL xử lý import |
| File XML Grid Detail | Controllers/Grid/ | Grid nhập liệu nhiều dòng, nhúng trong Dir |
| File XML Report | Controllers/Report/ | Khai báo mẫu in (RPT + Excel) cho báo cáo |

---

## ⚠️ Encoding bắt buộc khi tạo file XML

> **BẮT BUỘC**: Tất cả file XML (grid, dir, filter, lookup, upload, report) phải được tạo với encoding **UTF-8 BOM**.
> Nếu thiếu BOM, FBO runtime sẽ cảnh báo "File không có UTF-8 BOM, runtime có thể đọc lỗi" và có thể không đọc được file.

**Khi dùng `create_file` (Python):**
```python
# SAI — tạo UTF-8 không BOM
with open(path, 'w', encoding='utf-8') as f:
    f.write(content)

# ĐÚNG — tạo UTF-8 với BOM (utf-8-sig)
with open(path, 'w', encoding='utf-8-sig') as f:
    f.write(content)
```

**Khi dùng bash:**
```bash
# Thêm BOM vào đầu file sau khi tạo
printf '\xEF\xBB\xBF' | cat - file.xml > /tmp/tmp.xml && mv /tmp/tmp.xml file.xml
```

**Kiểm tra nhanh bằng bash:**
```bash
hexdump -C file.xml | head -1
# Dòng đầu phải bắt đầu bằng: ef bb bf 3c 3f 78 6d 6c
# (ef bb bf = BOM, 3c 3f 78 6d 6c = <?xml)
```

> **Rule Claude**: Khi `create_file` tạo file XML → luôn dùng Python snippet với `encoding='utf-8-sig'` thay vì ghi trực tiếp bằng `create_file` tool (vì tool đó không hỗ trợ BOM). Hoặc dùng bash `printf '\xEF\xBB\xBF'` để prepend BOM sau khi tạo.

---

## ⚠️ Quy ước tên file khi upload / download

Trong hệ thống gốc, các file XML nằm ở **thư mục khác nhau** (Dir/, Grid/, Filter/, Lookup/) nên cùng tên `SalesPrice.xml` không nhầm. Nhưng khi **upload lên chat** hoặc **download xuống máy local** — không còn context thư mục — dễ nhầm lẫn.

**Quy ước đặt tên khi share file:**

| File gốc | Tên khi upload/download |
|---|---|
| `Controllers/Dir/SalesPrice.xml` | `dir_SalesPrice.xml` |
| `Controllers/Grid/SalesPrice.xml` | `grid_SalesPrice.xml` |
| `Controllers/Filter/SalesPrice.xml` | `filter_SalesPrice.xml` |
| `Controllers/Lookup/Customer.xml` | `lookup_Customer.xml` |
| `Controllers/Templates/Upload/Job.xml` | `upload_Job.xml` |

**Khi Claude sinh file mới**: luôn dùng prefix `dir_`, `grid_`, `filter_`, `lookup_`, `aspx_`. Khi copy về hệ thống thực, bạn bỏ prefix và đặt vào đúng thư mục.

---

## Dùng file nào cho việc gì

| Nhiệm vụ | File cần đọc |
|---|---|
| **Cần biết file XML nào đặt vào folder nào trong Web/** | `SPEC_project-structure.md` ⭐ Bản đồ cây thư mục thực tế |
| Tạo danh mục đơn giản (mã + tên + status) | `SPEC_aspx-main.md` + `SPEC_grid.md` + `SPEC_dir.md` |
| Tạo danh mục có nhiều field → cần Tab | + `SPEC_dir.md` (phần Tabs) |
| Tạo danh mục có điều kiện lọc | + `SPEC_filter.md` |
| Danh mục có field kỳ (tháng/kỳ) và năm | + `SPEC_period-year.md` |
| Tạo Lookup / AutoComplete mới | `SPEC_lookup.md` → `SPEC_lookup-other.md` |
| Field tra cứu chọn nhiều (Lookup multi-select) | `SPEC_lookup-multiselect.md` |
| Xử lý ẩn/hiện field động | `SPEC_show-hide.md` |
| Viết SQL commands / response | `SPEC_sql-events.md` |
| Thêm tính năng Import Excel (upload hàng loạt) | `SPEC_import.md` |
| Danh mục / khai báo có Grid Detail (master-detail) | `SPEC_grid-detail.md` ⭐ Khi có grid nhập liệu |

| **Field kỳ/năm** | `SPEC_period-year.md` ⭐ Khi có field kỳ tháng + năm |
| **Tạo báo cáo** (tổng quan) | `SPEC_report-overview.md` ⭐ Đọc đầu tiên khi làm báo cáo |
| **Filter báo cáo** | `SPEC_report-filter.md` |
| **Grid báo cáo** | `SPEC_report-grid.md` |
| **Mẫu in báo cáo** (Report XML) | `SPEC_report-print.md` |
| **Mẫu in Excel báo cáo** (file .xlsx template) | `SPEC_report-excel.md` ⭐ |
| **Store báo cáo** (SQL, phân kỳ, tồn/dư) | `SPEC_report-store.md` ⭐ |

---

## ⭐ Quy trình tra cứu Lookup (tiết kiệm token)

Khi gặp field AutoComplete cần lookup, Claude kiểm tra **theo thứ tự**:

1. **`SPEC_lookup.md`** (đọc trước) — Cấu trúc XML + 9 lookup dùng nhiều nhất: Customer, Item, Site, Currency, UOM, Job, Account, Department, Unit
2. **`SPEC_lookup-other.md`** (chỉ đọc khi bước 1 không có) — Lookup ít dùng: SalesPerson, Bank, Tax, Discount, các nhóm danh mục (CustomerGroup, ItemGroup, JobGroup...), SalesPriceType, PaymentTerm...
3. **Custom Lookup từ prompt** — Lookup đặc thù dự án: prompt sẽ cung cấp controller, bảng, cột mã, cột tên → Claude tạo theo template trong `SPEC_lookup.md`

> Xem chi tiết cấu trúc XML và hướng dẫn tạo custom lookup tại `SPEC_lookup.md`.

## Phân loại mức độ phức tạp của danh mục

| Mức | Đặc điểm | File tham khảo |
|---|---|---|
| **Cơ bản** | < 10 field, không tab, không filter | `SPEC_dir.md` (pattern Bank) |
| **Trung bình** | 10-25 field, có tab, không filter | `SPEC_dir.md` (pattern Job — có tab) |
| **Có Import** | Có filter import, upload template, SQL processing | `SPEC_dir.md` + `SPEC_import.md` |
| **Có Grid Detail** | Form master + grid nhập liệu nhiều dòng, khóa tự nhập hoặc tự tăng | SPEC_grid-detail.md |

---

## Quy tắc đặt tên DB

### Bảng
- Danh mục: `dm{module}` — `dmkh`, `dmvt`, `dmnh`, `dmvv`
- View hiển thị: `v{tên_bảng}` — `vdmgia2`
- Chứng từ: `ct{xx}` — `ct00`, `ct70`
- Chi tiết danh mục: `ct{bảng_master}` hoặc `{bảng_master}{số}` — `ctdmku`, `dmpb1`

### Cột
- Mã: `ma_{module}` — `ma_kh`, `ma_vt`, `ma_nh`
- Tên: `ten_{module}` — `ten_kh`, `ten_nh`
- Tên hiển thị song ngữ: `ten_{module}%l` (`%l` = ngôn ngữ hiện hành)
- Tên khác: `ten_{module}2`
- Nhóm: `nh_{module}N` — `nh_kh1`, `nh_vv1`
- Flag: `phan_loai`, `status`, `vv_sd_pslk`
- Ngày: `ngay_{mô_tả}` — `ngay_vv`, `ngay_vv1`, `ngay_vv2`
- Tiền: `tien`, `tien_nt`, `gia_nt2`
- Audit (framework tự quản lý): `datetime0`, `datetime2`, `user_id0`, `user_id2`

---

## Quy trình tạo danh mục mới

> **⚠️ Lưu ý encoding**: Mọi file XML phải tạo bằng `encoding='utf-8-sig'` (UTF-8 với BOM). Xem chi tiết tại mục "Encoding bắt buộc khi tạo file XML" ở trên.

1. Xác định mức độ phức tạp (xem bảng phân loại ở trên)
2. Đọc `SPEC_aspx-main.md` → tạo file `aspx_{Controller}.aspx`
3. Đọc `SPEC_grid.md` → tạo file `grid_{Controller}.xml`
4. Đọc `SPEC_dir.md` → tạo file `dir_{Controller}.xml`
5. (Nếu cần) Đọc `SPEC_filter.md` → tạo `filter_{Controller}.xml`
6. (Nếu Grid cần tên hiển thị) Tạo SQL View `v{bảng}`
7. (Nếu Lookup chưa có) Tra `SPEC_lookup.md` → `SPEC_lookup-other.md` → nếu vẫn không có, tạo custom lookup theo `SPEC_lookup.md`
8. (Nếu có Import) Đọc `SPEC_import.md` → tạo `filter_{Controller}Import.xml` + `upload_{Controller}.xml` + bổ sung toolbar/script vào Grid
9. (Nếu có Grid Detail) Đọc `SPEC_grid-detail.md` → tạo `grid_{GridDetailController}.xml` + bổ sung field grid + SQL trong Dir

---

## 🔶 Chứng từ (Voucher) — Nhập liệu Master/Detail

> Khi prompt nói "làm chứng từ", "tạo phiếu nhập", "tạo hóa đơn", "voucher" → đọc các file `SPEC_voucher-*`.

### Thành phần chứng từ

| Thành phần | Thư mục | File cần tạo |
|---|---|---|
| Grid Browser | Controllers/Grid/ | `grid_{ma_ct}Tran.xml` |
| Grid Detail | Controllers/Grid/ | `grid_{ma_ct}Detail.xml` |
| Dir (Form) | Controllers/Dir/ | `dir_{ma_ct}Tran.xml` |
| Filter | Controllers/Filter/ | `filter_{ma_ct}Tran.xml` |
| Report | Controllers/Report/ | `report_{ma_ct}Tran.xml` |
| Upload (Import) | Controllers/Templates/Upload/ | `upload_{ma_ct}Master.xml` |

### Dùng file nào cho việc gì (Chứng từ)

| Nhiệm vụ | File cần đọc |
|---|---|
| Hiểu tổng quan chứng từ, bảng, entity, phân kỳ | `SPEC_voucher-overview.md` ⭐ Đọc đầu tiên |
| Tạo Grid Browser + Filter | `SPEC_voucher-grid.md` |
| Tạo Grid Detail (nhập liệu nhiều dòng) | `SPEC_voucher-detail.md` |
| Tạo Dir (form chứng từ — phức tạp nhất) | `SPEC_voucher-dir.md` |
| Tạo Report (mẫu in) | `SPEC_voucher-report.md` |
| Tạo Upload (import Excel) | `SPEC_import.md` (dùng chung) |
| Tra Lookup cho field | `SPEC_lookup.md` → `SPEC_lookup-other.md` |
| Xử lý ẩn/hiện field | `SPEC_show-hide.md` |

### Phân loại chứng từ

| Loại | Đặc điểm | Tham khảo |
|---|---|---|
| **Phân kỳ** | Bảng tách theo tháng (mXX$yyyymm), có bảng chung (cXX$) | IR (Phiếu nhập kho) |
| **Không phân kỳ** | Bảng cố định (phXX/ctXX), không có bảng chung | MO (Lệnh sản xuất) |

### Quy trình tạo chứng từ mới

1. Đọc `SPEC_voucher-overview.md` → xác định loại (phân kỳ / không), nhận biết entity
2. Đọc `SPEC_voucher-grid.md` → tạo Grid Browser (`grid_{ma_ct}Tran.xml`) + Filter (`filter_{ma_ct}Tran.xml`)
3. Đọc `SPEC_voucher-detail.md` → tạo Grid Detail (`grid_{ma_ct}Detail.xml`)
4. Đọc `SPEC_voucher-dir.md` → tạo Dir (`dir_{ma_ct}Tran.xml`) — file chính, phức tạp nhất
5. Đọc `SPEC_voucher-report.md` → tạo Report (`report_{ma_ct}Tran.xml`)
6. (Nếu có Import) Đọc `SPEC_import.md` → tạo Upload
7. Tra Lookup nếu cần: `SPEC_lookup.md` → `SPEC_lookup-other.md` → custom

---

## 🔷 Báo cáo (Report) — Hiển thị thông tin từ dữ liệu đã nhập

> Khi prompt nói "tạo báo cáo", "làm report" → đọc các file `SPEC_report-*`.
> Báo cáo KHÁC danh mục/chứng từ: **chỉ đọc, không nhập liệu, không có Dir**.

### Thành phần báo cáo

| Thành phần | Thư mục | File cần tạo |
|---|---|---|
| ASPX | Main/ | `aspx_{Tên}.aspx` |
| Filter | Controllers/Filter/ | `filter_{Tên}.xml` |
| Grid | Controllers/Grid/ | `grid_{Tên}.xml` |
| Report (Mẫu in) | Controllers/Report/ | `report_{Tên}.xml` |
| Excel Template | App_Data/Controllers/Templates/Excel/ | `{templateFile}.xlsx` |

### Dùng file nào cho việc gì (Báo cáo)

| Nhiệm vụ | File cần đọc |
|---|---|
| Hiểu tổng quan báo cáo, phân biệt với danh mục/chứng từ | `SPEC_report-overview.md` ⭐ Đọc đầu tiên |
| Tạo ASPX (FilterMode="true") | `SPEC_aspx-main.md` (template có Filter) |
| Tạo Filter (điều kiện lọc + Processing gọi Store) | `SPEC_report-filter.md` |
| Tạo Grid (type="Report", hiển thị kết quả) | `SPEC_report-grid.md` |
| Tạo Report XML (mẫu in RPT + Excel) | `SPEC_report-print.md` |
| Tạo file Excel template (.xlsx) cho mẫu in | `SPEC_report-excel.md` ⭐ |
| Viết Store procedure báo cáo (SQL, phân kỳ, tồn/dư) | `SPEC_report-store.md` ⭐ |
| Tra Lookup cho field lọc | `SPEC_lookup.md` → `SPEC_lookup-other.md` |

### Phân loại báo cáo

| Loại | Đặc điểm | Tham khảo |
|---|---|---|
| **Đơn giản** | Lọc → hiển thị → in | rptTransactionList |
| **Có chi tiết (drill-down)** | Click dòng → xem chi tiết | Đặc thù — prompt chỉ rõ |
| **Có chọn nhiều (check)** | Grid có checkbox, in nhiều | Đặc thù — prompt chỉ rõ |
| **Xoay (pivot)** | Cột xoay động | Đặc thù — prompt chỉ rõ |

### Quy trình tạo báo cáo mới

1. Đọc `SPEC_report-overview.md` → hiểu tổng thể
2. Tạo ASPX: `SPEC_aspx-main.md` (template có Filter — FilterMode="true")
3. Đọc `SPEC_report-filter.md` → tạo Filter (điều kiện lọc + Processing)
4. Đọc `SPEC_report-grid.md` → tạo Grid (type="Report")
5. Đọc `SPEC_report-print.md` → tạo Report (mẫu in)
5b. Đọc `SPEC_report-excel.md` → tạo file Excel template (.xlsx)
6. Tra Lookup: `SPEC_lookup.md` → `SPEC_lookup-other.md`
7. Đọc `SPEC_report-store.md` → viết Store procedure (hoặc tạo khung tạm)
