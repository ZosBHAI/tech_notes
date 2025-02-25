---
date: 2025-02-08
---
# Stored Procedure

1. `GO` - It is like `END`, which marks the end of the batch.
2. You cannot change the database context within a stored procedure using `USE dbName`.  
   - This is because it would be a separate batch, and a stored procedure is a collection of only one batch of statements.
3. `@` is used to access variables, and `@@` is used to access system variables.
4. Stored procedure is not Transactional by default .It can be made transactional  by including `BEGIN TRANS` or `COMMIT` or `ROLLBACK`. [SQL SERVER - Stored Procedure and Transactions - SQL Authority with Pinal Dave](https://blog.sqlauthority.com/2010/06/02/sql-server-stored-procedure-and-transactions/)

---

## NVARCHAR vs VARCHAR

5. `NVARCHAR` stores **UNICODE** data. If you need to store UNICODE or multilingual data, `NVARCHAR` is the choice.  
   `VARCHAR` stores **ASCII** data and should be used for normal use.  
   - **Example**: A **Name** or **Comment** column should be `NVARCHAR` because you might need to store local names like `'ESKİ'`.
6. `NVARCHAR` values are always prefixed with `N''`.
7. `NVARCHAR` requires extra storage, whereas `VARCHAR` only uses **1 byte**.

---

## APPLY

- Used only when dealing with a **table-valued function** or **expression**.
- Works by evaluating the right expression **for each row** in the left table and joining the result with the left table row.

### CROSS APPLY

- Similar to **INNER JOIN**.
- The **execution plan** is also similar to `JOIN`.

#### Example:
```sql
SELECT t1.*, t2.*
FROM Table1 t1
CROSS APPLY MyFunction(t1.Column1) AS t2;
```
## When to Use CROSS APPLY?

`CROSS APPLY` is particularly useful when joining a table with a **function that returns a dynamic result set** based on input values.

It is commonly used with:
- **Built-in functions** like `STRING_SPLIT()`
- **User-defined table-valued functions (UDTFs)`
## OUTER APPLY

- Similar to **LEFT OUTER JOIN**.

---

## STRING_AGG

- Can be very handy, especially when you want to **concatenate values from multiple rows into a single string**.

---

## JSON in SQL Server

- **SQL Server (version 13 and above)** supports **JSON processing** to enable **NoSQL support**.
- **Storing JSON**: It can be stored as `VARCHAR` or `NVARCHAR`.  
  **Reference:** [YouTube Video](https://www.youtube.com/watch?v=6SuHvdCg1tk&list=PL5FkCIZQgbvMfdrxZO8tYJYc4wW3pA5bV&index=1&pp=iAQB).

### JSON Functions

- **`ISJSON` (Transact-SQL)**: Tests whether a string contains valid JSON.  
  **Querying Reference:** [YouTube Video](https://www.youtube.com/watch?v=YJ3Spzuz4gA&list=PL5FkCIZQgbvMfdrxZO8tYJYc4wW3pA5bV&index=2)
- **`JSON_VALUE` (Transact-SQL)**: Extracts a **scalar value** from a JSON string.
- **`JSON_QUERY` (Transact-SQL)**: Extracts an **object or an array** from a JSON string.
- **`JSON_MODIFY` (Transact-SQL)**: Changes a value in a JSON string.

#### Example:
```sql
UPDATE [controlTable].[MainControlTable_28u]
SET [DataLoadingBehaviorSettings] = JSON_MODIFY([SinkObjectSettings], '$.folderPath', 'dflowPartioned') 
WHERE Id = 3;
```
### Query to Check if a Column Contains Valid JSON:
- **Reference:** [Microsoft Docs](https://learn.microsoft.com/en-us/sql/t-sql/functions/isjson-transact-sql?view=sql-server-ver16)

### Converting JSON in SQL Server:
- **`FOR AUTO` / `FOR JSON`** - Converts rows to JSON.
- **`OPENJSON`** - Converts JSON into rows.  
  **Reference:** [Microsoft Docs](https://learn.microsoft.com/en-us/sql/relational-databases/json/use-openjson-with-the-default-schema-sql-server?view=sql-server-ver16)
	
# VIEWS vs Functions vs Stored Procedure

- All elements are used for **code reusability**.
- **Views** and **Functions** support only `SELECT`; however, **Views** **cannot** accept parameters.
- **Functions** are classified into three types:
  1. **Scalar Function** - Returns a **single value**.
  2. **Inline Table-Valued Function** - Contains only **a single SELECT statement** and returns a **table**.
  3. **Multi-Statement Table-Valued Function** - Supports **multiple SELECT statements** and requires defining the **structure of the output**.
- **Functions** or **Views** can be executed from a `SELECT` statement, whereas **Stored Procedures** must be executed using a **separate statement**.
- **Stored Procedures** support **INSERT, UPDATE, DELETE, SELECT**, and **transactional statements**.

📌 **Good Summary:** [YouTube Video](https://www.youtube.com/watch?v=TSCPXpXL4OI)

---

## Stored Procedure - IF ELSE

- When using `IF...ELSE` inside a **Stored Procedure**:
  - The use of `BEGIN...END` is necessary if the body contains more than one line.
  - If the body contains only **one line**, `BEGIN...END` can be omitted.
  - Wrapping the condition in **parentheses** improves code readability.

---

## CTAS vs SELECT INTO

(TBD - Add comparison details)

---

## Data Masking

📌 **Reference:** [Microsoft Docs](https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver16)

- SQL Server provides a feature to **mask selected columns** through **UI** or **SQL queries**.
- **Starting from SQL Server 2022**, you can **grant/revoke UNMASK permission** at:
  - **Database level**
  - **Schema level**
  - **Table level**
  - **Column level**
- **Limitations**:
  - Cannot mask **PolyBase external table columns**.
- **Important Considerations**:
  - Even if the data is **masked**, the column can still be **updated**.
  - `SELECT` queries **still work** on the column, making it possible to **identify records**.
  - **Brute force attacks** can be used to infer masked data.
  - If you need **complete obfuscation**, use **Always Encrypted**.
  
📌 **Reference:** [YouTube Video](https://www.youtube.com/watch?v=qEAVYtxtS2o)

### 🔹 **Production Use Case**
#### **1) SQL Authentication**
- **Create a ROLE** for unmasking.
- **Add a service user** to the role.
- **Grant `MASK` permission** to this role.
- **Power BI** can use this service user to connect to SQL Server.