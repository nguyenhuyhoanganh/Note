# Chiến thuật đánh Index và Scale hệ thống lên hàng tỷ row

*Cách một Senior Database Engineer thật sự suy nghĩ và ra quyết định*

------------------------------------------------------------------------

# I. TƯ DUY GỐC -- ĐỪNG BAO GIỜ BẮT ĐẦU BẰNG "TẠO INDEX"

Một senior **không hỏi**:

> "Cột nào cần index?"

Mà hỏi:

## 1. Workload là gì?

**Workload** = đặc điểm và khối lượng truy cập hệ thống.

-   90% Read hay 90% Write?
    -   **Read** = thao tác đọc dữ liệu
    -   **Write** = thao tác ghi dữ liệu (INSERT, UPDATE, DELETE)
-   Read là:
    -   **Point lookup** = truy vấn đúng 1 giá trị cụ thể
    -   **Range scan** = truy vấn theo khoảng giá trị
-   Có:
    -   `JOIN`? → kết hợp nhiều bảng
    -   `ORDER BY`? → sắp xếp
    -   Pagination? → phân trang
    -   Filter nhiều cột?

## 2. Data Distribution thế nào?

**Data distribution** = cách dữ liệu phân bố trong cột.

-   **Selectivity** = tỷ lệ giá trị duy nhất
-   Có **skew** (lệch dữ liệu) không?

Ví dụ:

``` sql
WHERE state = 'CA'
```

Nếu `'CA'` chiếm 90% bảng → index gần như vô nghĩa.

## 3. Bảng sẽ lớn đến đâu?

-   1M rows
-   100M rows
-   1B rows
-   10B rows

Chiến thuật cho 1M **khác hoàn toàn** 1B.

------------------------------------------------------------------------

# II. CHIẾN THUẬT ĐÁNH INDEX THEO CASE CỤ THỂ

## CASE 1: Lookup theo Primary Key

``` sql
SELECT * FROM users WHERE id = ?
```

-   Dùng `BIGINT` auto increment
-   Tránh `UUID v4`
-   Distributed system → `UUID v7` hoặc `ULID`

Clustered index ordered → tránh page split.

------------------------------------------------------------------------

## CASE 2: Unique Business Key

``` sql
SELECT * FROM users WHERE email = ?
```

-   `UNIQUE INDEX`
-   PostgreSQL → `UNIQUE CONSTRAINT`
-   High write → giảm `fillfactor < 100`

------------------------------------------------------------------------

## CASE 3: Filter nhiều cột (AND)

``` sql
SELECT *
FROM orders
WHERE user_id = ?
AND status = 'PAID'
```

``` sql
CREATE INDEX idx_orders_user_status
ON orders (user_id, status);
```

------------------------------------------------------------------------

## CASE 4: ORDER BY

``` sql
SELECT *
FROM orders
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```

``` sql
CREATE INDEX idx_orders_user_created
ON orders (user_id, created_at DESC);
```

------------------------------------------------------------------------

## CASE 5: Pagination lớn

Sai:

``` sql
LIMIT 1000000 OFFSET 20;
```

Đúng:

``` sql
WHERE id > last_seen_id
ORDER BY id
LIMIT 20;
```

------------------------------------------------------------------------

## CASE 6: Range Scan

``` sql
WHERE created_at BETWEEN ...
```

``` sql
CREATE INDEX idx_orders_created
ON orders (created_at);
```

------------------------------------------------------------------------

## CASE 7: OR Condition

``` sql
WHERE a = ? OR b = ?
```

Rewrite:

``` sql
SELECT ... WHERE a = ?
UNION ALL
SELECT ... WHERE b = ?;
```

------------------------------------------------------------------------

## CASE 8: Covering Query

``` sql
SELECT id, grade
FROM students
WHERE grade > 90;
```

``` sql
CREATE INDEX idx_students_grade
ON students (grade) INCLUDE (id);
```

------------------------------------------------------------------------

## CASE 9: Partial Index

``` sql
CREATE INDEX idx_active_users
ON users (email)
WHERE deleted_at IS NULL;
```

------------------------------------------------------------------------

# III. WRITE HEAVY vs READ HEAVY

## Write Heavy

-   Chỉ primary key
-   Partition theo time
-   Tránh index trên cột update thường xuyên

## Read Heavy

-   Covering index
-   Composite index
-   Denormalize
-   Materialized View

------------------------------------------------------------------------

# IV. KHI BẢNG 100M → 1B ROW

## 0 → 10M

-   Index đơn giản

## 10M → 100M

-   Audit index
-   Nghĩ đến partition

## 100M → 1B

-   Partition theo time / tenant / region
-   Kiểm soát vacuum
-   Theo dõi bloat

------------------------------------------------------------------------

# V. KHI NÀO SHARD?

Shard khi:

-   CPU saturate
-   IO saturate
-   Storage limit
-   Partition không đủ

Chiến lược:

-   Hash user_id
-   Range time
-   Geo region

------------------------------------------------------------------------

# VI. UUID CHIẾN THUẬT

Tránh UUID v4 làm clustered key.

Dùng:

-   BIGINT sequence
-   UUID v7
-   ULID

------------------------------------------------------------------------

# VII. TỐI ƯU IO

-   Ordered insert
-   Sequential key
-   Time-based partition

> Index không làm query nhanh.\
> Index làm giảm IO.
