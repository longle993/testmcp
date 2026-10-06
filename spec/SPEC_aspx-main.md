# [SPEC_aspx-main] — File ASPX (Main Page)

> Dùng khi: tạo trang ASPX cho danh mục / báo cáo / chứng từ.
> File gốc: `Web/Main/{Tên}.aspx`
> Khi share/download: `aspx_{Tên}.aspx`

---

## Quy tắc quan trọng

1. **UTF-8 BOM**: File ASPX phải lưu **UTF-8 with BOM** (có byte `EF BB BF` ở đầu file).
2. **CRLF**: Line ending phải là **CRLF** (`\r\n`, Windows-style). Tool `create_file` chạy trên Linux sẽ tạo LF — **không dùng `create_file`** cho file ASPX.
3. **Single-line**: Cả thẻ `<%@ Page ... %>` và `<FastBusiness:ReportExtender ... />` phải viết **trên 1 dòng duy nhất**, không xuống dòng hay indent.
4. **Thẻ đóng `</asp:Content>`** cuối file viết liền — không thêm dòng trống.

---

## Cách tạo file ASPX đúng chuẩn

**KHÔNG dùng `create_file`** — phải dùng `bash_tool` + Python:

```python
content = (
    '<%@ Page AutoEventWireup="false" MasterPageFile="~/Main/MasterPage.master" '
    'Inherits="FastBusiness.ReportExtender.UI.Page" '
    'v="{TênViệt}" e="{EnglishName}"%>\r\n'
    '\r\n'
    '<asp:Content ID="headContent" ContentPlaceHolderID="head" runat="server"></asp:Content>\r\n'
    '<asp:Content ID="mainContent" ContentPlaceHolderID="FastBusiness" runat="server">\r\n'
    '    <div>\r\n'
    '        <asp:Panel ID="panelReport" runat="server"/>\r\n'
    '    </div>\r\n'
    '    <FastBusiness:ReportExtender ID="MainReport" runat="server" '
    'TargetControlID="panelReport" ReadOnly="true" '
    'Controller="{TênController}" FilterMode="true"/>\r\n'
    '</asp:Content>'
)

with open(output_path, 'w', encoding='utf-8-sig', newline='') as f:
    f.write(content)
```

Giải thích:
- `encoding='utf-8-sig'` → tự thêm BOM (`EF BB BF`) ở đầu file
- `newline=''` → Python không tự chuyển đổi line ending, giữ nguyên `\r\n` trong content
- `\r\n` viết tường minh trong content string → đảm bảo CRLF

---

## Template danh mục đơn giản (không có Filter)

```aspx
<%@ Page AutoEventWireup="false" MasterPageFile="~/Main/MasterPage.master" Inherits="FastBusiness.ReportExtender.UI.Page" v="Tên tiếng Việt" e="English Name"%>

<asp:Content ID="headContent" ContentPlaceHolderID="head" runat="server"></asp:Content>
<asp:Content ID="mainContent" ContentPlaceHolderID="FastBusiness" runat="server">
    <div>
        <asp:Panel ID="panelReport" runat="server"/>
    </div>
    <FastBusiness:ReportExtender ID="MainReport" runat="server" TargetControlID="panelReport" ReadOnly="true" Controller="{TênController}"/>
</asp:Content>
```

Bỏ `FilterMode="true"` khi không có file Filter.

## Template danh mục / báo cáo có Filter

```aspx
<%@ Page AutoEventWireup="false" MasterPageFile="~/Main/MasterPage.master" Inherits="FastBusiness.ReportExtender.UI.Page" v="Tên tiếng Việt" e="English Name"%>

<asp:Content ID="headContent" ContentPlaceHolderID="head" runat="server"></asp:Content>
<asp:Content ID="mainContent" ContentPlaceHolderID="FastBusiness" runat="server">
    <div>
        <asp:Panel ID="panelReport" runat="server"/>
    </div>
    <FastBusiness:ReportExtender ID="MainReport" runat="server" TargetControlID="panelReport" ReadOnly="true" Controller="{TênController}" FilterMode="true"/>
</asp:Content>
```

## Thuộc tính quan trọng

| Thuộc tính | Giá trị | Mô tả |
|---|---|---|
| `v` | string | Tên tiếng Việt hiển thị trên tab |
| `e` | string | Tên tiếng Anh |
| `Controller` | string | Tên file XML (không có .xml) |
| `ReadOnly` | `true` | Luôn true cho danh mục / báo cáo |
| `FilterMode` | `true` | Thêm khi có file Filter |

## Thứ tự thuộc tính trong ReportExtender

Viết theo đúng thứ tự: `ID` → `runat` → `TargetControlID` → `ReadOnly` → `Controller` → `FilterMode`

## Lưu ý

- Controller phải trùng tên file XML trong 3 thư mục Grid/, Dir/, Filter/
- File ASPX đặt trong thư mục `Web/Main/` (cùng cấp với `MasterPage.master`)
