# [SPEC_sql-events] — SQL Events trong commands

> Dùng khi: viết SQL cho các event trong Dir, Filter, Grid.

---

## Danh sách Events

| Event | Khi nào chạy | Dùng trong |
|---|---|---|
| `Loading` | Form/Grid vừa load | Dir, Grid, Filter |
| `Showing` | Form hiển thị (trước Loading) | Dir, Filter |
| `Closing` | Đóng form | Dir, Grid, Filter |
| `Declare` | Khai báo biến dùng chung | Dir |
| `InitExternalFields` | Gán giá trị cho field external | Dir |
| `Scattering` | Sau khi load dữ liệu vào form | Dir |
| `Inserting` | Trước khi INSERT (validate) | Dir, Filter |
| `Inserted` | Sau khi INSERT thành công | Dir |
| `Updating` | Trước khi UPDATE (validate) | Dir |
| `Updated` | Sau khi UPDATE thành công | Dir |
| `Deleting` | Trước khi DELETE (validate) | Dir |
| `Checking` | Validate tùy chỉnh | Dir |
| `Processing` | Xử lý/tính toán dữ liệu | Filter |

## Biến hệ thống

| Biến | Mô tả |
|---|---|
| `@@table` | Tên bảng (từ `table=` trên `<dir>`) |
| `@@userID` | ID người dùng |
| `@@language` | `'v'` = Việt |
| `@@admin` | `1` = admin |
| `@@sysDatabaseName` | Tên DB hệ thống |
| `@@appDatabaseName` | Tên DB ứng dụng |
| `@{field}` | Giá trị mới |
| `${field}.OldValue` | Giá trị cũ (chỉ trong Updating) |
| `@datetime0`, `@datetime2` | Datetime tạo / cập nhật |
| `@user_id0`, `@user_id2` | UserID tạo / cập nhật |

## Cách trả kết quả từ SQL

### Trả lỗi (form báo lỗi tại field)
```sql
select 'tên_field' as field, N'Thông báo lỗi' as message
return
```

### Gọi hàm JavaScript từ SQL
```sql
select 'activeFormDetail(this);' as message
return
```

### Trả kết quả + gọi JS
```sql
select '' as field, '' as message, 'myFunction();' as script
return
```

## Template Declare (composite key)

Dùng ENTITY để tái sử dụng điều kiện key phức tạp:
```xml
<!ENTITY k1 "ma_vt = @ma_vt AND ma_nt = @ma_nt AND ngay_ban = @ngay_ban">
<!ENTITY k2 "ma_vt = $ma_vt.OldValue AND ma_nt = $ma_nt.OldValue AND ngay_ban = $ngay_ban.OldValue">
```

## Template Inserting

```sql
if exists(select 1 from @@table where ]]>&k1;<![CDATA[) begin
  select 'field_pk' as field, @$exists as message
  return
end
select @datetime0 = getdate(), @datetime2 = getdate(), @user_id0 = @@userID, @user_id2 = @@userID
```

## Template Updated

```sql
update @@table set datetime2 = getdate(), user_id2 = @@userID
where ]]>&k1;
```
