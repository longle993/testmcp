# [SPEC_report-store] — Viết Store Procedure cho Báo cáo

> Dùng khi: viết store procedure xử lý dữ liệu cho báo cáo.
> Đọc `SPEC_report-overview.md` trước để hiểu luồng dữ liệu.
> File này hướng dẫn **cách viết SQL store** — phần XML gọi store xem `SPEC_report-filter.md` (event Processing).

---

## Tổng quan

Store báo cáo là nơi xử lý logic nghiệp vụ: tạo bảng tạm theo cấu trúc, lấy dữ liệu từ bảng phân kỳ, tính toán, nhóm, rồi trả dataset cho Grid hiển thị.

### Luồng gọi

```
Filter (Processing) → exec rs_rptXxx @param1, ..., @@language, @@userID, @@admin, 'Controller', '@@sysDatabaseName', '#$query'
                                           ↓
                                 Store xử lý → trả dataset (1 hoặc nhiều resultset)
                                           ↓
                                 Grid nhận dataset → hiển thị
```

---

## Quy ước đặt tên

| Loại | Prefix store | Ví dụ |
|---|---|---|
| **Báo cáo chuẩn** | `rs_rpt{TênController}` | `rs_rptStockSummary` |
| **Báo cáo customize** | `zc_{tên_controller}` | `zc_bctonkho` |
| **Báo cáo HRM** | `hs_rpt{TênController}` | `hs_rptSalary` |

> Controller `zcbctonkho` → store `zc_bctonkho`.

---

## Tham số

### Kiểu dữ liệu tham số

| Loại | Kiểu | Ví dụ |
|---|---|---|
| Ngôn ngữ | `CHAR(1)` | `@Language` |
| Ngày | `SMALLDATETIME` | `@DateFrom`, `@DateTo` |
| Mã đơn lẻ | `VARCHAR(32)` | `@Site`, `@Item`, `@Customer` |
| Danh sách / chuỗi | `VARCHAR(1024)` hoặc `NVARCHAR(1024)` | `@Unit`, `@Account`, `@VCs` |
| Loại (0/1/2...) | `CHAR(1)` hoặc `TINYINT` | `@Type`, `@DataType` |
| Kích thước | `TINYINT` | `@Size` |
| User ID | `INT` | `@UserID` |
| Admin | `BIT` | `@Admin` |
| Controller | `VARCHAR(128)` | `@Controller` (default `''`) |
| Tên bảng dynamic | `VARCHAR(32)` | `@TableName` (default `''`) |
| Database hệ thống | `VARCHAR(32)` hoặc `VARCHAR(64)` | `@SysDatabase` (default `''`) |

### Biến thường dùng (từ điều kiện lọc)

| Biến trên Filter | Biến trên Store | Kiểu |
|---|---|---|
| `tu_ngay` | `@DateFrom` | `SMALLDATETIME` |
| `den_ngay` | `@DateTo` | `SMALLDATETIME` |
| `ma_vt` | `@Item` | `VARCHAR(32)` |
| `tk` | `@Account` | `VARCHAR(1024)` |
| `ma_kh` | `@Customer` | `VARCHAR(32)` |
| `ma_dvcs` | `@Unit` | `VARCHAR(1024)` |
| `ma_kho` | `@Site` | `VARCHAR(32)` |
| `@@language` | `@Language` | `CHAR(1)` |
| `@@userID` | `@UserID` | `INT` |
| `@@admin` | `@Admin` | `BIT` |

### 3 tham số cuối (Dynamic — có default)

Store chuẩn thường có 3 tham số cuối với giá trị mặc định, store khi làm tính năng mới không cần, khi cần sẽ ghi rõ:

```sql
@Controller VARCHAR(128) = '',
@TableName VARCHAR(32) = '',   -- hoặc @DynamicKeyTable, @SysDatabase
@SysDatabase VARCHAR(32) = ''
```

---

## Quy tắc viết SQL

### Hoa/thường

| Ngữ cảnh | Quy tắc | Ví dụ |
|---|---|---|
| Từ khóa SQL viết trực tiếp | **IN HOA** | `SELECT`, `FROM`, `WHERE`, `INSERT INTO` |
| Từ khóa SQL trong chuỗi động | **in thường** | `'insert into #report select ...'` |
| Tên biến, tên bảng, tên cột | **in thường** | `@DateFrom`, `ma_vt`, `#report` |

### SET mở đầu / kết thúc

```sql
BEGIN
  SET NOCOUNT ON
  SET ANSI_NULLS OFF
  -- ... body ...
  SET NOCOUNT OFF
  SET ANSI_NULLS ON
END
```

### DECLARE biến trong Store

| Loại | Kiểu SQL |
|---|---|
| Số tiền, giá trị `NUMERIC(20, 5)` |
| Mã | `VARCHAR(33)` |
| Tên | `NVARCHAR(256)` |
| Chuỗi điều kiện / query | `NVARCHAR(4000)` |

---

## Cấu trúc Store — Từng bước

### Bước 1: Tạo bảng tạm bằng `SELECT TOP 0 ... INTO`

**Không dùng `CREATE TABLE`** — dùng `SELECT TOP 0 ... INTO #report` từ bảng mẫu (`wrkgl`, `wrkcolumns`, `cdvt`) để tạo đúng kiểu dữ liệu:

**Báo cáo sổ cái:**
```sql
SELECT TOP 0 5 AS sysorder, 1 AS sysprint, 1 AS systotal,
    a.stt_rec, a.ngay_ct, a.so_ct, a.so_ct0, a.tk, a.tk_du, a.ma_nt, a.ty_gia,
    a.ps_no, a.ps_no_nt, a.ps_co, a.ps_co_nt, a.dien_giai, a.ma_ct, a.ma_kh,
    a.nh_dk, a.line_nbr, b.stt_ct_nkc, b.ma_ct_in AS ma_ct0
  INTO #report
  FROM wrkgl a LEFT JOIN dmct b ON a.ma_ct = b.ma_ct
  ORDER BY 1
```

**Báo cáo kho (2 bước):**
```sql
-- Bước tạo bảng trung gian
SELECT TOP 0 a.ma_kho, a.ma_vt, ton00 AS so_luong, du00 AS tien, du_nt00 AS tien_nt
  INTO #t FROM wrkcolumns a, cdvt b

-- Bảng báo cáo chính
SELECT TOP 0 ma_vt, ma_kho, so_luong AS ton_dau, tien AS du_dau, tien_nt AS du_dau_nt,
    so_luong AS sl_nhap, tien AS tien_nhap, tien_nt AS tien_nt_n,
    so_luong AS sl_xuat, tien AS tien_xuat, tien_nt AS tien_nt_x
  INTO #report FROM #t
```

> Kỹ thuật alias: `ton00 AS so_luong` → tạo cột `so_luong` mang kiểu dữ liệu của `ton00`.

### Bước 2: Dynamic Fields (nếu cần)

Cho phép user tùy biến cột hiển thị trên báo cáo:

```sql
DECLARE @baseFields NVARCHAR(4000), @baseAliasFields NVARCHAR(4000),
  @totalFields NVARCHAR(4000), @sumTotalFields NVARCHAR(4000),
  @sumFields NVARCHAR(4000), @sumAliasFields NVARCHAR(4000),
  @queryJoin NVARCHAR(4000), @baseJoin NVARCHAR(4000), @filterKey NVARCHAR(4000)

EXEC FastBusiness$Report$GetDynamicFields '#report', @TableName, @Controller,
  @SysDatabase, @Language, 0,
  @baseFields OUTPUT, @baseAliasFields OUTPUT, @totalFields OUTPUT,
  @sumTotalFields OUTPUT, @sumFields OUTPUT, @sumAliasFields OUTPUT,
  @queryJoin OUTPUT, @baseJoin OUTPUT

SELECT @baseFields = REPLACE(@baseFields, '%a', 'a'),
  @baseAliasFields = REPLACE(@baseAliasFields, '%a', 'a'),
  @sumTotalFields = REPLACE(@sumTotalFields, '%a.', ''),
  @queryJoin = REPLACE(@queryJoin, '%a', 'a'),
  @baseJoin = REPLACE(@baseJoin, '%a', 'a')
```

> Không phải store nào cũng cần Dynamic Fields. Chỉ dùng khi báo cáo có hỗ trợ user tùy biến cột.

### Bước 3: Xây dựng chuỗi điều kiện @Key

#### Pattern chung

```sql
SELECT @Key = ''  -- hoặc khởi tạo với điều kiện status
```

#### Điều kiện khởi tạo

- Sổ cái: `SELECT @Key = 'a.status = ''1'''`
- Kho: `SELECT @Key = ''` (không có status mặc định)

#### Điều kiện mã — dùng `like` + `REPLACE`

```sql
IF @Customer <> '' SET @Key = @Key + CASE WHEN @Key = '' THEN '' ELSE ' and ' END + 'a.ma_kh like ''' + REPLACE(RTRIM(@Customer), '''', '''''') + '%'''
```

> **Quan trọng**: Luôn dùng `REPLACE(RTRIM(@Var), '''', '''''')` để escape nháy đơn.
> Pattern cộng điều kiện: `CASE WHEN @Key = '' THEN '' ELSE ' and ' END + ...`

#### Điều kiện tài khoản — dùng `GetAccountFilter`

```sql
IF @Account <> '' SET @Key = @Key + ' and ' + dbo.FastBusiness$Function$System$GetAccountFilter('a.tk', 'inlist', @Account)
```

> `'inlist'` khi filter Lookup chọn nhiều, `'like'` khi filter AutoComplete đơn lẻ.

#### Điều kiện danh sách (Lookup chọn nhiều) — dùng `ff_inlist`

```sql
IF @VCs <> '' SET @Key = @Key + ' and dbo.ff_inlist(b.ma_ct_in, ''' + REPLACE(RTRIM(@VCs), '''', '''''') + ''') = 1'
```

#### Phân quyền đơn vị — dùng `GetUnitFilter` (function)

```sql
SET @UnitKey = dbo.FastBusiness$Function$System$GetUnitFilter('ma_dvcs', @Unit, @UserID, @Admin)
IF @UnitKey IS NOT NULL SET @Key = @Key + CASE WHEN @Key = '' THEN '' ELSE ' and ' END + @UnitKey
```

#### Phân quyền kho — dùng `GetSiteFilter` (exec)

```sql
EXEC FastBusiness$System$GetSiteFilter 'a.ma_kho', @Site, @UnitKey, @UserID, @Admin, @SiteKey OUTPUT
IF @SiteKey IS NOT NULL SET @Key = @Key + CASE WHEN @Key = '' THEN '' ELSE ' and ' END + @SiteKey
```

#### Dynamic Key (filter từ Grid)

```sql
EXEC FastBusiness$Report$GetDynamicKey @TableName, @filterKey OUTPUT
SET @filterKey = REPLACE(ISNULL(@filterKey, ''), '%[a]', 'a')
IF @filterKey <> '' SET @Key = @Key + CASE WHEN @Key <> '' THEN ' and ' ELSE '' END + @filterKey
```

#### Bước cuối: Check Key (BẮT BUỘC)

```sql
SET @Key = dbo.FastBusiness$Function$System$GetCheckKey(@Key)
```

> **Luôn gọi `GetCheckKey`** trên toàn bộ @Key trước khi dùng. Hàm này validate và sanitize chuỗi điều kiện.

### Bước 4: Lấy dữ liệu bảng phân kỳ

#### Cú pháp bảng phân kỳ: `bảng$%Partition`

Trong chuỗi query động, bảng phân kỳ viết dạng `{prefix}$%Partition`:

```sql
SET @q = 'insert into #report select 5 as sysorder, 1 as sysprint, 1 as systotal, ...'
SET @q = @q + ' from r00$%Partition a with(nolock) left join dmct b on a.ma_ct = b.ma_ct'
SET @q = @q + ' where %[' + @Key + ']%'
EXEC FastBusiness$Partition$Execute @q, @Unit, 'ngay_ct', @DateFrom, @DateTo, @UserID, @Admin
```

**Giải thích cú pháp:**
- `r00$%Partition`: framework thay `%Partition` thành suffix tháng (`202601`, `202602`...)
- `with(nolock)`: luôn có để tránh lock khi đọc
- `%[` + @Key + `]%`: framework thêm điều kiện ngày phân kỳ vào @Key tại đây

#### `FastBusiness$Partition$Execute`

```sql
EXEC FastBusiness$Partition$Execute @q, @Unit, 'ngay_ct', @DateFrom, @DateTo, @UserID, @Admin
```

| Tham số | Mô tả |
|---|---|
| `@q` | Chuỗi SQL chứa `%Partition` và `%[...]%` |
| Tham số 2 | Đơn vị cơ sở (truyền `@Unit` hoặc `NULL`) |
| Tham số 3 | Tên cột ngày phân kỳ (`'ngay_ct'` hoặc `'a.ngay_ct'`) |
| `@DateFrom` | Từ ngày |
| `@DateTo` | Đến ngày |
| `@UserID` | User ID |
| `@Admin` | Admin flag |

**Cách hoạt động:**
- Hàm gen ra nhiều câu `INSERT ... SELECT ... FROM r00$yyyyMM` cho từng tháng trong khoảng lọc.
- Tháng đầu/cuối nếu lọc lưng chừng sẽ có thêm điều kiện ngày chính xác.

#### Ví dụ sổ cái

```sql
SET @q = 'insert into #report select 5 as sysorder, 1 as sysprint, 1 as systotal,'
SET @q = @q + ' a.stt_rec, a.ngay_ct, a.so_ct, a.tk, a.tk_du, a.ma_nt, a.ty_gia,'
SET @q = @q + ' a.ps_no, a.ps_no_nt, a.ps_co, a.ps_co_nt, a.dien_giai, a.ma_ct, a.ma_kh'
SET @q = @q + ' from r00$%Partition a with(nolock) left join dmct b on a.ma_ct = b.ma_ct'
SET @q = @q + ' where %[' + @Key + ']%'
EXEC FastBusiness$Partition$Execute @q, @Unit, 'ngay_ct', @DateFrom, @DateTo, @UserID, @Admin
```

#### Ví dụ kho (r90$ + r70$)

```sql
-- Kho thực tế
IF @DataType = 1 BEGIN
  SET @q = 'insert into #report select a.ma_vt, max(a.ma_kho), 0, 0, 0,'
  SET @q = @q + ' sum(sl_nhap), sum(tien_nhap), sum(tien_nt_n), sum(sl_xuat), sum(tien_xuat), sum(tien_nt_x)'
  SET @q = @q + ' from r90$%Partition a with(nolock)' + @Join
  SET @q = @q + ' where %[' + @Key + ']%'
  SET @q = @q + ' group by a.ma_vt'
  EXEC FastBusiness$Partition$Execute @q, NULL, 'ngay_ct', @DateFrom, @DateTo, @UserID, @Admin
END

-- Kho hóa đơn (hoặc cả hai nếu không tách)
IF @DataType <> 1 OR NOT EXISTS(SELECT 1 FROM options WHERE name = 'm_instock_split' AND val = '1') BEGIN
  SET @q = 'insert into #report select a.ma_vt, max(a.ma_kho), 0, 0, 0,'
  SET @q = @q + ' sum(sl_nhap), sum(tien_nhap), sum(tien_nt_n), sum(sl_xuat), sum(tien_xuat), sum(tien_nt_x)'
  SET @q = @q + ' from r70$%Partition a with(nolock)' + @Join
  SET @q = @q + ' where %[' + @Key + ']%'
  SET @q = @q + ' group by a.ma_vt'
  EXEC FastBusiness$Partition$Execute @q, NULL, 'ngay_ct', @DateFrom, @DateTo, @UserID, @Admin
END
```

### Bước 5: Tính số dư / tồn đầu kỳ

#### Tồn kho đầu kỳ — `FastBusiness$Balance$Item`

```sql
INSERT INTO #t EXEC FastBusiness$Balance$Item @DateFrom, @Unit, @Site, @Item, 1, 2, @DataType, @UserID, @Admin, @Join, @Key

INSERT INTO #report SELECT ma_vt, ma_kho, so_luong AS ton_dau, tien AS du_dau, tien_nt AS du_dau_nt, 0, 0, 0, 0, 0, 0  -- sl_nhap, tien_nhap... khởi tạo = 0FROM #t
```

#### Số dư tài khoản đầu kỳ — `FastBusiness$Balance$Account`

```sql
CREATE TABLE #t (du NUMERIC(19, 2), du_nt NUMERIC(19, 2))
INSERT #t EXEC FastBusiness$Balance$Account @DateFrom, @Unit, @Account, 1, 1, @UserID, @Admin
SELECT @nDu_dk = Du, @nDu_dk_nt = Du_nt FROM #t
DROP TABLE #t

IF @nDu_dk IS NULL SET @nDu_dk = 0
IF @nDu_dk_nt IS NULL SET @nDu_dk_nt = 0
```

### Bước 6: Tính số dư lũy kế (Running Balance)

Kỹ thuật running update với CLUSTERED INDEX:

```sql
-- Tạo bảng sắp xếp với IDENTITY
SELECT id, ps_no, ps_co, ps_no_nt, ps_co_nt, du_no, du_no_nt, IDENTITY(INT, 1,1) AS stt
INTO #bal FROM #report a JOIN dmct b ON a.ma_ct = b.ma_ct
  ORDER BY a.sysorder, a.ngay_ct, b.stt_ct_nkc, b.ma_ct_in, a.so_ct, a.stt_rec, a.line_nbr
CREATE CLUSTERED INDEX i ON #bal(stt)

-- Running update
SELECT @du_no = @nDu_dk, @du_no_nt = @nDu_dk_nt
UPDATE #bal SET
  @du_no = @du_no + ps_no - ps_co,
  @du_no_nt = @du_no_nt + ps_no_nt - ps_co_nt,
  du_no = ISNULL(@du_no, 0),
  du_no_nt = ISNULL(@du_no_nt, 0)
FROM #bal WITH(INDEX(i))

-- Gán lại vào bảng chính
UPDATE #report SET du_no = b.du_no, du_no_nt = b.du_no_nt
FROM #report a JOIN #bal b ON a.id = b.id

-- Tách dư nợ / dư có
UPDATE #report SET du_co = - du_no, du_no = 0 WHERE du_no < 0
UPDATE #report SET du_co_nt = - du_no_nt, du_no_nt = 0 WHERE du_no_nt < 0
```

### Bước 7: Dòng hệ thống (Đầu kỳ / PS cộng / Cuối kỳ / Blank)

```sql
-- Lấy nhãn từ bảng reports
SELECT @s1 = CASE WHEN @Language = 'V' THEN cname ELSE cname2 END FROM reports WHERE ccode = 'OPBAL'    -- Dư đầu kỳ
SELECT @s2 = CASE WHEN @Language = 'V' THEN cname ELSE cname2 END FROM reports WHERE ccode = 'PRAMOUNT' -- Phát sinh
SELECT @s3 = CASE WHEN @Language = 'V' THEN cname ELSE cname2 END FROM reports WHERE ccode = 'CLBAL'    -- Dư cuối kỳ

-- Dòng dư đầu kỳ (sysorder = 0)
INSERT INTO #report (sysorder, sysprint, systotal, dien_giai, ps_no, ps_no_nt, ps_co, ps_co_nt)
  VALUES (0, 0, 0, @s1,
    CASE WHEN @nDu_dk >= 0 THEN @nDu_dk ELSE 0 END,
    CASE WHEN @nDu_dk_nt >= 0 THEN @nDu_dk_nt ELSE 0 END,
    CASE WHEN @nDu_dk < 0 THEN -@nDu_dk ELSE 0 END,
    CASE WHEN @nDu_dk_nt < 0 THEN -@nDu_dk_nt ELSE 0 END)

-- Dòng phát sinh cộng (sysorder = 1)
INSERT INTO #report (sysorder, sysprint, systotal, dien_giai, ps_no, ps_no_nt, ps_co, ps_co_nt)
  VALUES (1, 0, 0, @s2, @nPs_no, @nPs_no_nt, @nPs_co, @nPs_co_nt)

-- Dòng dư cuối kỳ (sysorder = 2)
INSERT INTO #report (sysorder, sysprint, systotal, dien_giai, ps_no, ps_no_nt, ps_co, ps_co_nt)
  VALUES (2, 0, 0, @s3,
    CASE WHEN @nDu_ck >= 0 THEN @nDu_ck ELSE 0 END,
    CASE WHEN @nDu_ck_nt >= 0 THEN @nDu_ck_nt ELSE 0 END,
    CASE WHEN @nDu_ck < 0 THEN -@nDu_ck ELSE 0 END,
    CASE WHEN @nDu_ck_nt < 0 THEN -@nDu_ck_nt ELSE 0 END)

-- Dòng blank (sysorder = 3)
INSERT INTO #report (sysorder, sysprint, systotal) VALUES (3, 0, 0)
```

#### Giá trị sysorder

| sysorder | Loại dòng | systotal | sysprint | Hiển thị |
|---|---|---|---|---|
| `0` | Dư đầu kỳ | `0` | `0` | Trên Grid, không in trên mẫu in |
| `1` | Phát sinh cộng / Blank | `0` | `0` | Trên Grid |
| `2` | Dư cuối kỳ | `0` | `0` | Trên Grid |
| `3` | Blank / Nhóm | `0` | `0` | Phân cách |
| **`5`** | **Dòng chi tiết** | **`1`** | **`1`** | **Grid + In** |

> Dòng chi tiết (systotal=1) là dòng duy nhất Grid tính aggregate (Sum).

### Bước 8: Nhóm dữ liệu (báo cáo kho)

```sql
-- Nhóm lại và tính tồn cuối
SELECT 5 AS sysorder, 1 AS sysprint, 1 AS systotal,
    0 AS stt, a.ma_vt,
    CASE WHEN @Language = 'V' THEN MAX(b.ten_vt) ELSE '' END AS ten_vt,
    CASE WHEN @Language <> 'V' THEN MAX(b.ten_vt2) ELSE '' END AS ten_vt2,
    MAX(b.dvt) AS dvt,
    SUM(a.ton_dau) AS ton_dau, SUM(a.du_dau) AS du_dau, SUM(a.du_dau_nt) AS du_dau_nt,
    SUM(a.sl_nhap) AS sl_nhap, SUM(a.tien_nhap) AS tien_nhap, SUM(a.tien_nt_n) AS tien_nt_n,
    SUM(a.sl_xuat) AS sl_xuat, SUM(a.tien_xuat) AS tien_xuat, SUM(a.tien_nt_x) AS tien_nt_x,
    MAX(a.ton_dau * 0) AS ton_cuoi, MAX(a.du_dau * 0) AS du_cuoi, MAX(a.du_dau_nt * 0) AS du_cuoi_nt
  INTO #incd1
  FROM #report a LEFT JOIN dmvt b WITH(INDEX(PK_dmvt)) ON a.ma_vt = b.ma_vt
  GROUP BY a.ma_vt

-- Tính tồn cuối
UPDATE #incd1 SET ton_cuoi = ton_dau + sl_nhap - sl_xuat,
  du_cuoi = du_dau + tien_nhap - tien_xuat,
  du_cuoi_nt = du_dau_nt + tien_nt_n - tien_nt_x
```

> `CASE WHEN @Language = 'V' THEN ... ELSE '' END`: lấy tên theo ngôn ngữ hiện tại.

### Bước 9: Dòng Tổng cộng

**Báo cáo kho:**
```sql
DECLARE @s1 AS NVARCHAR(511)
SELECT @s1 = cname FROM reports WHERE ccode = 'TOTAL'

INSERT INTO #incd1 (sysorder, sysprint, systotal, ten_vt, ten_vt2,
    ton_dau, du_dau, du_dau_nt, sl_nhap, tien_nhap, tien_nt_n,
    sl_xuat, tien_xuat, tien_nt_x, ton_cuoi, du_cuoi, du_cuoi_nt)
  SELECT 0, 0, 0, @s1, @s12,
    SUM(ton_dau), SUM(du_dau), ..., SUM(ton_cuoi), SUM(du_cuoi), SUM(du_cuoi_nt)
  FROM #incd1 WHERE systotal = 1
```

**Báo cáo sổ cái (có Dynamic Fields):**
```sql
SET @q = 'insert into #report (sysorder, sysprint, systotal, dien_giai, ps_no, ps_no_nt, ps_co, ps_co_nt%sumFields%)'
SET @q = @q + 'select 0, 0, 0, N''' + REPLACE(@s, '''', '''''') + ''', sum(ps_no), sum(ps_no_nt), sum(ps_co), sum(ps_co_nt)%sumValues% from #report'
SET @q = REPLACE(REPLACE(@q,
  '%sumFields%', CASE WHEN @totalFields <> '' THEN ',' + @totalFields ELSE '' END),
  '%sumValues%', CASE WHEN @sumTotalFields <> '' THEN ',' + @sumTotalFields ELSE '' END)
EXEC sp_executesql @q
```

### Bước 10: SELECT kết quả cuối

**Trực tiếp (không Dynamic):**
```sql
SELECT sysorder, systotal, sysprint,
    a.ngay_ct, a.so_ct, a.ma_kh, a.dien_giai, a.tk_du, a.ma_ct, a.stt_rec,
    a.ps_no, a.ps_co, a.ps_no_nt, a.ps_co_nt,
    du_no, du_co, du_no_nt, du_co_nt,
    b.ten_kh, d.ma_ct_in AS ma_ct0
  FROM #report a
    LEFT JOIN dmkh b ON a.ma_kh = b.ma_kh
    LEFT JOIN dmct d ON a.ma_ct = d.ma_ct
  ORDER BY a.sysorder, a.ngay_ct, d.stt_ct_nkc, d.ma_ct_in, a.so_ct, a.stt_rec, a.line_nbr
```

**Qua chuỗi động (có Dynamic Fields):**
```sql
SET @q = 'select sysorder, sysprint, systotal, a.stt_rec, a.ma_ct, ...'
SET @q = @q + CASE WHEN @baseFields <> '' THEN ',' ELSE '' END + @baseFields
SET @q = @q + ' from #report a left join dmkh b on a.ma_kh = b.ma_kh' + @baseJoin
SET @q = @q + ' order by sysorder, ngay_ct, ...'
EXEC sp_executesql @q
```

---

## Báo cáo kho — Kho hóa đơn vs Kho thực tế

| `@DataType` | Ý nghĩa | Bảng |
|---|---|---|
| `0` hoặc khác `1` | Kho hóa đơn | `r70$` |
| `1` | Kho thực tế | `r90$` + một phần `r70$` nếu không tách |

Tham số `m_instock_split` (bảng options):
- `= '1'`: tách kho hóa đơn và thực tế riêng
- Khác: một số chứng từ lưu chung `r70$`

> Mặc định giữ xử lý theo mẫu `rs_rptStockSummary`.

---

## Nhãn từ bảng `reports`

| ccode | Ý nghĩa | cname (V) | cname2 (E) |
|---|---|---|---|
| `OPBAL` | Số dư đầu kỳ | Dư đầu kỳ | Opening Balance |
| `PRAMOUNT` | Phát sinh | Phát sinh | Arising Amount |
| `CLBAL` | Số dư cuối kỳ | Dư cuối kỳ | Closing Balance |
| `TOTAL` | Tổng cộng | Tổng cộng | Total |

```sql
SELECT @s = CASE WHEN @Language = 'V' THEN cname ELSE cname2 END
FROM reports WHERE ccode = 'TOTAL'
```

---

## Store tham khảo

| Store | Báo cáo | Đặc điểm chính |
|---|---|---|
| `rs_rptTransactionList` | Bảng kê chứng từ sổ cái | Dynamic Fields, running balance, dư đầu/cuối kỳ |
| `rs_rptStockSummary` | Tổng hợp NXT | Tồn đầu kỳ (`Balance$Item`), kho HĐ/TT, nhóm, order |
| `rs_rptAccountActivity` | Sổ cái tài khoản | Dư đầu kỳ (`Balance$Account`), running balance |

---

## Checklist khi viết Store báo cáo

- [ ] `SET NOCOUNT ON` + `SET ANSI_NULLS OFF` mở đầu
- [ ] Tạo bảng tạm bằng `SELECT TOP 0 ... INTO` (không `CREATE TABLE`)
- [ ] Xây dựng @Key: escape nháy đơn bằng `REPLACE(RTRIM(@var), '''', '''''')`, pattern `CASE WHEN @Key = '' THEN '' ELSE ' and ' END`
- [ ] **BẮT BUỘC** gọi `dbo.FastBusiness$Function$System$GetCheckKey(@Key)` trước khi dùng
- [ ] Bảng phân kỳ: cú pháp `bảng$%Partition` + `where %[key]%`
- [ ] Gọi `FastBusiness$Partition$Execute` để thực thi
- [ ] Tính tồn/dư: dùng `FastBusiness$Balance$Item` hoặc `FastBusiness$Balance$Account`
- [ ] Dòng hệ thống: sysorder `0/1/2/3` cho đầu kỳ/PS/cuối kỳ/blank, `5` cho chi tiết
- [ ] Dòng Tổng: `systotal = 0`, `sysprint = 0`, lấy nhãn từ bảng `reports`
- [ ] SELECT cuối: JOIN danh mục (dmkh, dmct, dmvt...) lấy tên, ORDER BY `sysorder` + ngày + chứng từ
- [ ] `SET NOCOUNT OFF` + `SET ANSI_NULLS ON` kết thúc
