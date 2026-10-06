# Vibe Coding Rules — VITACO HRM SP24

## 0. MỤC TIÊU

Ưu tiên theo thứ tự:

1. Đúng yêu cầu người dùng.
2. Đúng convention và framework của project.
3. Thay đổi ít nhất có thể.
4. Giữ nguyên kiến trúc hiện tại.
5. Không sửa ngoài scope.
6. Tiết kiệm context/token.

Nguyên tắc mặc định:

> Read Relevant Docs → Find Existing Pattern → Change Less → Validate Enough → Report Briefly

Không tự mở rộng yêu cầu, refactor hoặc "cải thiện" ngoài yêu cầu.

---

# 1. SCOPE — QUAN TRỌNG NHẤT

- Chỉ tạo/sửa/xóa/đổi tên file nằm trong phạm vi người dùng cho phép.
- Nếu người dùng nói "chỉ sửa `<file>`" → chỉ được sửa đúng file đó.
- Không tự sửa dependency ngoài scope.
- Không tự tạo file "cho đủ bộ".
- Không refactor code ngoài yêu cầu.
- Không đổi framework, library, architecture, naming convention, database structure hoặc coding style nếu người dùng không yêu cầu.

### Dependency ngoài scope

Nếu implementation bắt buộc phải sửa file ngoài scope:

1. DỪNG trước khi sửa.
2. Nêu:
   - File cần sửa.
   - Lý do cần sửa.
   - Ảnh hưởng nếu không sửa.
3. Xin phép người dùng mở rộng scope.
4. Chỉ sửa sau khi được đồng ý.

Không tự suy luận rằng dependency được phép sửa.

---

# 2. PROJECT

Project là:

- ASP.NET Web Forms
- FastBusiness / FBO
- XML-based configuration
- JavaScript
- SQL Server

Cấu trúc chính:

```text
Main/
App_Data/
├── Controllers/
│   ├── Dir/
│   ├── Grid/
│   ├── Filter/
│   ├── Lookup/
│   ├── Report/
│   ├── Structure/
│   └── Include/
```

Các file XML trong `App_Data/Controllers` sử dụng convention và framework riêng của FastBusiness.

Không áp dụng máy móc convention của ASP.NET/Web Forms hoặc framework khác nếu không có bằng chứng trong project.

---

# 3. DOCUMENTATION — SPEC LÀ FRAMEWORK KNOWLEDGE

Thư mục specification chứa tài liệu chính thức về convention và behavior của framework.

```text
specs/
├── 00_INDEX.md
├── SPEC_aspx-main.md
├── SPEC_dir.md
├── SPEC_filter.md
├── SPEC_grid.md
├── SPEC_grid-detail.md
├── SPEC_import.md
├── SPEC_javascript-patterns.md
├── SPEC_lookup.md
├── SPEC_lookup-multiselect.md
├── SPEC_lookup-other.md
├── SPEC_period-year.md
├── SPEC_project-structure.md
├── SPEC_report-excel.md
├── SPEC_report-filter.md
└── SPEC_report-grid.md
```

### Quy tắc quan trọng

Không đọc toàn bộ `specs/`.

Trước khi implementation:

1. Đọc `specs/00_INDEX.md`.
2. Xác định SPEC liên quan đến task.
3. Chỉ đọc SPEC cần thiết.
4. Sau đó tìm implementation thực tế tương tự trong source code.
5. Dùng SPEC + existing code làm cơ sở implementation.

### Thứ tự ưu tiên

```text
User requirement
    ↓
Relevant SPEC
    ↓
Existing project implementation
    ↓
General framework knowledge
```

Không sử dụng kiến thức framework chung để thay thế convention riêng của project.

Nếu SPEC và code hiện tại có vẻ khác nhau:

- Không tự "sửa" theo suy đoán.
- Kiểm tra thêm implementation gần nhất.
- Nếu vẫn không xác định được → hỏi người dùng.

---

# 4. SPEC SELECTION

Sử dụng `00_INDEX.md` để xác định SPEC phù hợp.

| Task | SPEC chính |
|---|---|
| ASPX Main | `SPEC_aspx-main.md` |
| Dir / Form | `SPEC_dir.md` |
| Grid | `SPEC_grid.md` |
| Grid detail | `SPEC_grid-detail.md` |
| Filter | `SPEC_filter.md` |
| Lookup | `SPEC_lookup.md` |
| Lookup nhiều giá trị | `SPEC_lookup-multiselect.md` |
| Lookup đặc biệt | `SPEC_lookup-other.md` |
| JavaScript | `SPEC_javascript-patterns.md` |
| Import | `SPEC_import.md` |
| Project structure | `SPEC_project-structure.md` |
| Report | SPEC report tương ứng |
| Period / Year | `SPEC_period-year.md` |

Có thể đọc thêm SPEC khác nếu task có dependency trực tiếp.

Không đọc SPEC không liên quan chỉ để "chắc chắn".

---

# 5. EXISTING CODE IS THE IMPLEMENTATION REFERENCE

SPEC mô tả framework.

Existing code cho biết project hiện đang áp dụng framework như thế nào.

Trước khi tạo hoặc sửa chức năng:

1. Tìm module tương tự nhất.
2. Đọc phần liên quan.
3. Xác định pattern đang được sử dụng.
4. Áp dụng pattern đó cho task hiện tại.

Ưu tiên:

```text
Existing project pattern
>
SPEC example
>
Generic framework knowledge
```

Không sao chép nguyên module mẫu nếu có logic không liên quan.

Chỉ lấy pattern cần thiết.

Không tìm nhiều module nếu một module đã đủ để xác định convention.

---

# 6. PROJECT CONVENTION

## 6.1 ASPX

- `Main/*.aspx` là entry page của chức năng.
- Không đổi tên ASPX hiện có.
- Giữ nguyên cấu trúc ASPX hiện tại.
- Không thêm logic ngoài convention của project.

## 6.2 Controller XML

Các loại controller chính:

```text
Dir
Grid
Filter
Lookup
Report
Structure
Include
```

Không mặc định tạo đủ các loại controller.

Chỉ tạo file thực sự cần thiết.

## 6.3 File `.f`

- Không sửa `.f` nếu người dùng không yêu cầu.
- Không tự đổi hoặc refactor `.f`.

## 6.4 XML

Khi sửa XML:

- Giữ nguyên XML declaration.
- Giữ nguyên DOCTYPE.
- Giữ nguyên namespace.
- Giữ nguyên ENTITY.
- Giữ nguyên CDATA.
- Không format lại toàn bộ file nếu không cần.
- Không thay đổi query/command ngoài yêu cầu.

---

# 7. DIR / FORM

Khi task liên quan đến `Controllers/Dir/*.xml`:

BẮT BUỘC đọc:

```text
specs/SPEC_dir.md
```

và đọc thêm SPEC liên quan nếu task sử dụng:

- Lookup
- Multi-select Lookup
- JavaScript
- Suggestion
- SQL Events

### Không tự tạo pattern

Không tự suy luận cách viết:

- `<field>`
- `<items>`
- `<views>`
- `<commands>`
- `<script>`
- `<response>`
- `<css>`

nếu pattern tương ứng đã được định nghĩa trong SPEC.

### AutoComplete

Nếu field AutoComplete chỉ để hiển thị tên:

- sử dụng pattern AutoComplete chuẩn của SPEC;
- không tự thêm `clientScript`;
- không tự thêm `<response>`;
- không tự thêm JS request.

Nếu task yêu cầu lấy thêm thông tin phụ:

- mới sử dụng pattern request/response tương ứng.

### Multi-select

Nếu prompt nói "lookup chọn nhiều":

- sử dụng pattern multi-select;
- không chuyển thành AutoComplete.

### Commands

Nếu tạo/sửa Dir:

- kiểm tra các event bắt buộc theo `SPEC_dir.md`;
- không bỏ sót event framework yêu cầu;
- không tự thay thế pattern chuẩn bằng implementation mới.

Đặc biệt kiểm tra:

```text
Loading
Closing
Declare
Inserting
Updating
Updated
```

---

# 8. GRID

Khi task liên quan đến Grid:

1. Đọc `SPEC_grid.md`.
2. Nếu là master/detail → đọc thêm `SPEC_grid-detail.md`.
3. Tìm một Grid tương tự.
4. Chỉ sửa field/view/query liên quan.

Không tự thay đổi query, sorting, width, view hoặc field nếu không liên quan đến yêu cầu.

---

# 9. FILTER

Khi task liên quan đến Filter:

1. Đọc `SPEC_filter.md`.
2. Tìm Filter tương tự.
3. Chỉ thêm/sửa field cần thiết.
4. Không tự tạo Filter nếu chức năng không cần Filter.

---

# 10. LOOKUP

Khi task liên quan đến Lookup:

- Đọc SPEC Lookup phù hợp.
- Phân biệt:
  - AutoComplete / chọn một
  - Lookup / chọn nhiều
  - Lookup đặc biệt

Nếu prompt không nói chọn nhiều:

- không tự dùng multi-select.

Nếu prompt nói "lookup chọn nhiều":

- sử dụng pattern multi-select tương ứng.

---

# 11. JAVASCRIPT

Khi task có JavaScript:

1. Đọc `SPEC_javascript-patterns.md`.
2. Kiểm tra pattern JS hiện tại của module tương tự.
3. Chỉ thêm JS cần thiết.

Không tạo function mới nếu framework đã có function phù hợp.

Không viết JavaScript để thay thế behavior mà framework đã tự xử lý.

---

# 12. SQL

Khi sửa SQL trong XML:

- Giữ nguyên SQL hiện tại nếu không liên quan.
- Không refactor query chỉ vì có thể viết ngắn hơn.
- Không thay đổi stored procedure/function/table ngoài scope.
- Không tự tạo validation SQL nếu framework/SPEC đã có validation tương ứng.

Nếu SQL liên quan đến framework event:

- ưu tiên pattern trong SPEC;
- sau đó đối chiếu module thực tế tương tự.

---

# 13. SCOPE CỦA CÁC LOẠI CHỨC NĂNG

## Danh mục

Có thể gồm:

```text
Main/<module>.aspx
Controllers/Dir/<module>.xml
Controllers/Grid/<module>.xml
Controllers/Filter/<module>.xml
```

Nhưng không mặc định tạo đủ.

Không tự tạo Lookup, Report, Structure, Include, Excel template, RPT, menu hoặc config nếu người dùng không yêu cầu.

## Báo cáo

Có thể gồm:

```text
Main/<module>.aspx
Controllers/Filter/<module>.xml
Controllers/Grid/<module>.xml
Controllers/Report/<module>.xml
```

Không tự sửa Excel template, RPT, Include, Lookup, menu hoặc config nếu không nằm trong yêu cầu.

---

# 14. KHẢO SÁT — TIẾT KIỆM CONTEXT

Không scan toàn bộ project.

Thứ tự:

```text
1. User request
2. Scope
3. 00_INDEX.md
4. Relevant SPEC
5. Search existing implementation
6. Read relevant section
7. Edit
```

Ưu tiên:

```text
search/find
    ↓
relevant section
    ↓
edit
```

Không đọc toàn bộ project, toàn bộ Controllers, toàn bộ SPEC hoặc nhiều module mẫu không cần thiết.

Nếu đã đủ thông tin → implementation ngay.

---

# 15. MINIMAL CHANGE

Luôn ưu tiên:

- ít file hơn;
- ít dòng thay đổi hơn;
- ít dependency hơn;
- đúng convention hơn.

Không:

- refactor;
- rename;
- format toàn file;
- reorder code;
- optimize code không liên quan;
- sửa lỗi ngoài task.

Nếu phát hiện lỗi không liên quan:

> Không tự sửa.

---

# 16. XML SAFETY

Khi sửa XML, không được làm mất:

```text
XML declaration
DOCTYPE
ENTITY
namespace
CDATA
```

ENTITY phải nằm ngoài CDATA.

Đúng:

```xml
<text><![CDATA[
...
]]>&ScriptIrregular;</text>
```

Không:

```xml
<text><![CDATA[
...&ScriptIrregular;...
]]></text>
```

Chỉ format đoạn vừa sửa nếu cần.

---

# 17. QUY TRÌNH IMPLEMENTATION

## Bước 1 — Xác định scope

Xác định:

- file được sửa;
- file được tạo;
- file không được đụng tới.

Nếu scope rõ → không hỏi lại.

## Bước 2 — Xác định documentation

Đọc:

```text
specs/00_INDEX.md
```

Sau đó chỉ đọc SPEC liên quan.

## Bước 3 — Tìm implementation mẫu

Tìm một module tương tự nhất.

Không cần nhiều mẫu nếu một mẫu đủ.

## Bước 4 — Implementation

Thực hiện thay đổi nhỏ nhất.

Tuân thủ:

```text
User requirement
+
Relevant SPEC
+
Existing project pattern
```

## Bước 5 — Validation

Validation tương xứng với thay đổi.

### XML

- well-formed XML;
- DOCTYPE;
- ENTITY;
- CDATA;
- field;
- view;
- command;
- controller reference.

### JavaScript

- syntax;
- function name;
- event binding;
- request/response context.

### SQL

- syntax;
- parameter;
- table/field;
- event context.

### Scope

- không có file ngoài scope bị thay đổi;
- không có thay đổi ngoài yêu cầu.

Không chạy build/test toàn bộ project nếu validation cục bộ đã đủ.

---

# 18. VALIDATION THEO SPEC

Khi SPEC có checklist:

> Phải sử dụng checklist đó để validation khi task thuộc phạm vi của SPEC.

Không chỉ kiểm tra XML well-formed.

Phải kiểm tra các rule framework quan trọng mà SPEC yêu cầu.

---

# 19. TASK ĐƠN GIẢN VS TASK PHỨC TẠP

## Task đơn giản

Ví dụ:

```text
Thêm field ma_nv vào Dir.
```

Workflow:

```text
Read relevant SPEC
→ Find similar field
→ Edit
→ Validate
```

Không cần lập kế hoạch dài.

## Task phức tạp

Ví dụ:

```text
Tạo một chức năng danh mục hoàn chỉnh.
```

Plan ngắn:

```text
1. Determine required controllers.
2. Read relevant SPECs.
3. Find reference module.
4. Implement.
5. Validate.
```

---

# 20. KHI THIẾU THÔNG TIN

Không hỏi nếu có thể xác định từ:

- SPEC;
- existing code;
- project convention;
- task requirement.

Chỉ hỏi khi thiếu thông tin ảnh hưởng trực tiếp đến implementation.

Nếu không thể xác định:

→ hỏi người dùng trước khi tạo pattern mới.

Không tự bịa.

---

# 21. QUY TẮC KHI SPEC KHÔNG ĐỦ

Nếu SPEC không mô tả vấn đề:

1. Tìm implementation thực tế tương tự.
2. Kiểm tra thêm SPEC liên quan.
3. Kiểm tra framework pattern đang được sử dụng trong project.

Không tự phát minh convention mới nếu chưa có bằng chứng.

Nếu vẫn không xác định được:

→ hỏi người dùng trước khi tạo pattern mới.

---

# 22. KHÔNG SUY DIỄN QUÁ MỨC

Không suy luận rằng một file cần tồn tại chỉ vì:

```text
"Thông thường chức năng này có file X"
```

Ví dụ:

```text
Có Dir ≠ phải có Grid
Có Grid ≠ phải có Filter
Có Lookup ≠ phải có Lookup-other
```

Chỉ tạo khi requirement hoặc implementation thực tế yêu cầu.

---

# 23. FINAL REPORT

Sau khi hoàn thành:

```text
Đã hoàn thành.

Files:
- ...

SPEC:
- ...

Tham khảo:
- ...

Thay đổi:
- ...

Validation:
- XML: OK
- Scope: OK
- JS: ...
- SQL: ...
- Build/Test: ...

Remaining issues:
- None
```

Không giải thích dài nếu người dùng không yêu cầu.

---

# 24. NGUYÊN TẮC CUỐI

Khi không chắc chắn:

1. Ưu tiên yêu cầu trực tiếp của người dùng.
2. Đọc SPEC liên quan.
3. Ưu tiên implementation đang tồn tại.
4. Ưu tiên thay đổi nhỏ nhất.
5. Không tự mở rộng scope.
6. Không tự sửa dependency.
7. Không tự tạo convention mới.
8. Không đọc dữ liệu không cần thiết.
9. Validate theo đúng mức độ thay đổi.
10. Báo cáo ngắn gọn.

> Read Relevant Docs → Find Existing Pattern → Change Less → Validate Enough → Report Briefly
