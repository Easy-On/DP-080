# Transact-SQL Cheat Sheet
By Roman Korecky

## [SELECT][1]

SELECT is the core command of SQL. It is used to query or read data.

### [Return values of expressions][2]

```SQL
SELECT 10 + 2, 10 - 2, 4 * 1.2, 12 / 1.1, 10 % 3
```

### [Query a table][3]

Select all columns (not recommended)

```SQL
SELECT * FROM SalesLT.Customer
```

Select specific columns

```SQL
SELECT CustomerID, CompanyName, SalesPerson, EmailAddress FROM SalesLT.Customer
```

### [Column aliases][2]

```SQL
SELECT
    CustomerID As Id, 
    CompanyName As Company, 
    SalesPerson As [Sales Person], 
    EmailAddress As [E-mail address]
FROM SalesLT.Customer
```

### [Sort results][4]

```SQL
SELECT CustomerID, CompanyName, ModifiedDate
FROM SalesLT.Customer
ORDER BY ModifiedDate DESC, Companyname
```

### [Limit number of results][2]

```SQL
SELECT TOP 10 ProductID, Name, ListPrice
FROM SalesLT.Product
ORDER BY ListPrice DESC
```

Always use the ```TOP``` together with ```ORDER BY```.

The clause ```WITH TIES``` may return more results than specified with ```TOP```, if there are records with values equal to the last record in the top results in the sorted columns.

```SQL
SELECT TOP 10 WITH TIES ProductID, Name, ListPrice
FROM SalesLT.Product
ORDER BY ListPrice DESC
```

The clause ```PERCENT``` returns the top n percent of all records.

```SQL
SELECT TOP 10 PERCENT ProductID, Name, ListPrice
FROM SalesLT.Product
ORDER BY ListPrice DESC
```

### [Page results][4]

```SQL
SELECT ProductID, Name, ListPrice
FROM SalesLT.Product
ORDER BY Name
OFFSET 0 ROWS
FETCH NEXT 10 ROWS ONLY
```

To query the next 10 records, run the same query with an offset of 10 * n.

```SQL
SELECT ProductID, Name, ListPrice
FROM SalesLT.Product
ORDER BY Name
OFFSET 10 ROWS
FETCH NEXT 10 ROWS ONLY
```

### [Remove duplicates][2]

```SQL
SELECT DISTINCT CountryRegion, StateProvince, City
FROM SalesLT.Address
ORDER BY CountryRegion, StateProvince, City
```

### [Filter data with predicates][5]

```SQL
SELECT ProductID, Name, ProductCategoryID
FROM SalesLT.Product
WHERE ProductCategoryID = 6
```

```SQL
SELECT ProductID, Name, ProductCategoryID
FROM SalesLT.Product
WHERE Color <> 'White'
````

#### [Handling NULLs][21]

```SQL
SELECT Name, ParentProductCategoryID
FROM SalesLT.ProductCategory
WHERE ParentProductCategoryID IS NULL
```

```SQL
SELECT Name, ParentProductCategoryID
FROM SalesLT.ProductCategory
WHERE ParentProductCategoryID IS NOT NULL
```

### [Use inner joins][3]

All examples return return equal results.

```SQL
SELECT p.Name, p.ProductCategoryID, c.Name
FROM SalesLT.Product AS p
INNER JOIN SalesLT.ProductCategory AS c
    ON p.ProductCategoryID = c.ProductCategoryID
```

```SQL
SELECT p.Name, p.ProductCategoryID, c.Name
FROM SalesLT.Product AS p
JOIN SalesLT.ProductCategory AS c
    ON p.ProductCategoryID = c.ProductCategoryID
```

This ANSI SQL-89 syntax is not recommended anymore:

```SQL
SELECT p.Name, p.ProductCategoryID, c.Name
FROM SalesLT.Product AS p, SalesLT.ProductCategory AS c
WHERE p.ProductCategoryID = c.ProductCategoryID
```

### [Use outer joins][3]

#### Left join

The following two examples return equal results.

```SQL
SELECT c.CustomerID, c.CompanyName, o.SalesOrderID
FROM SalesLT.Customer AS c
LEFT OUTER JOIN SalesLT.SalesOrderHeader AS o
    ON c.CustomerID = o.CustomerID
```

```SQL
SELECT c.CustomerID, c.CompanyName, o.SalesOrderID
FROM SalesLT.Customer AS c
LEFT JOIN SalesLT.SalesOrderHeader AS o
    ON c.CustomerID = o.CustomerID
```

More practical example:

```SQL
-- Find customers without sales orders
SELECT c.CustomerID, c.CompanyName, o.SalesOrderID
FROM SalesLT.Customer AS c
LEFT JOIN SalesLT.SalesOrderHeader AS o
    ON c.CustomerID = o.CustomerID
WHERE o.SalesOrderID IS NULL
```

#### Right join

The examples return equal results as the left join examples.

```SQL
SELECT c.CustomerID, c.CompanyName, o.SalesOrderID
FROM SalesLT.SalesOrderHeader AS o
RIGHT OUTER JOIN SalesLT.Customer AS c
    ON o.CustomerID = c.CustomerID
```

```SQL
SELECT c.CustomerID, c.CompanyName, o.SalesOrderID
FROM SalesLT.SalesOrderHeader AS o
RIGHT JOIN SalesLT.Customer AS c
    ON o.CustomerID = c.CustomerID
```

### [Use cross joins][3]

```SQL
SELECT c.CompanyName, p.Name
FROM SalesLT.Customer AS c
CROSS JOIN SalesLT.Product AS p
```

This ANSI SQL-89 syntax is not recommended anymore:

```SQL
SELECT c.CompanyName, p.Name
FROM SalesLT.Customer AS c, SalesLT.Product AS p
```

### [Summarize data with GROUP BY][]

```SQL
SELECT CustomerID, COUNT(*) AS OrderCount
FROM SalesLT.SalesOrderHeader
GROUP BY CustomerID
```

#### [Filter groups with HAVING][54]

```SQL
SELECT
    CustomerID,
    COUNT(*) AS OrderCount
FROM SalesLT.SalesOrderHeader
GROUP BY CustomerID
HAVING COUNT(*) > 10;
```

### [Create a new table from query results][56]

```SQL
SELECT SalesOrderID, CustomerID, OrderDate, PurchaseOrderNumber, TotalDue
INTO SalesLT.Invoice
FROM SalesLT.SalesOrderHeader;
```

### [Logical processing order][1]

1. FROM → get source data
1. WHERE → filter rows
1. GROUP BY → create groups
1. HAVING → filter groups
1. SELECT → choose output columns
1. ORDER BY → sort results 

## [INSERT][55]

```SQL
INSERT INTO SalesLT.CallLog
VALUES  (
    '2015-01-01T12:30:00', 
    'adventure-works\pamela0',
    1,
    '245-555-0173',
    'Returning call re: enquiry about delivery'
)
```

```SQL
INSERT INTO SalesLT.CallLog (
    CallTime,
    SalesPerson,
    CustomerID,
    PhoneNumber,
    Notes
)
VALUES  (
    DEFAULT, 
    'adventure-works\david8', 
    2, 
    '170-555-0127', 
    NULL
)
```

```SQL
INSERT INTO SalesLT.CallLog (
    SalesPerson, 
    CustomerID, 
    PhoneNumber, 
    Notes
) 
SELECT 
    SalesPerson, 
    CustomerID, 
    Phone, 
    'Sales promotion call' 
FROM SalesLT.Customer 
WHERE CompanyName = 'Big-Time Bike Store'
```

### Retrieving an identity value

[Return the most recent identity value in the current scope for any table][57]:

```SQL
SELECT SCOPE_IDENTITY();
```

[Return the latest identity value in a specific table][58]:

```SQL
SELECT IDENT_CURRENT('SalesLT.CallLog')
```

### Allow custom identity value on insert

```SQL
SET IDENTITY_INSERT SalesLT.CallLog ON; 
```

### Disallow custom identity value on insert

```SQL
SET IDENTITY_INSERT SalesLT.CallLog OFF;
```

## SEQUENCE

[Create a sequence][59]

```SQL
    CREATE SEQUENCE SalesLT.InvoiceNumber AS INT
    START WITH 1000 INCREMENT BY 1;
```

[Next value of a sequence][60]

```SQL
NEXT VALUE FOR SalesLT.InvoiceNumber
```

[Sort sequence values by another column][60]

```SQL
SELECT 
    NEXT VALUE FOR SaleLT.InvoiceNumber OVER (
        ORDER BY InvoiceDate
    ) AS InvoiceNumber
FROM SalesLT.Invoice
```

## [UPDATE][61]

Basic update

```SQL
UPDATE SalesLT.CallLog 
SET Notes = 'No notes' 
WHERE Notes IS NULL
```

Update from a table

```SQL
UPDATE SalesLT.CallLog 
SET
    SalesPerson = c.SalesPerson,
    PhoneNumber = c.Phone 
FROM SalesLT.Customer AS c 
WHERE c.CustomerID = SalesLT.CallLog.CustomerID
```

or

```SQL
UPDATE SalesLT.CallLog 
SET
    SalesPerson = c.SalesPerson,
    PhoneNumber = c.Phone 
FROM SalesLT.Customer AS c 
INNER JOIN SalesLT.CallLog AS l
    ON c.CustomerID = l.CustomerID;
```

## [DELETE][62]

```SQL
DELETE FROM SalesLT.CallLog 
WHERE CallTime < DATEADD(dd, -7, GETDATE());
```

```SQL
DELETE SalesLT.CallLog 
WHERE CallTime < DATEADD(dd, -7, GETDATE());
```

### [Delete all rows][63]

```SQL
TRUNCATE TABLE SalesLT.CallLog
```

is more efficient than

```SQL
DELETE FROM SaleLT.CallLog
```

### [Synchronize tables (MERGE)][64]

```SQL
MERGE INTO Sales.Invoice as i
USING Sales.InvoiceStaging as s
ON i.SalesOrderID = s.SalesOrderID
WHEN MATCHED THEN
    UPDATE SET
        i.CustomerID = s.CustomerID,
        i.OrderDate = GETDATE(),
        i.PurchaseOrderNumber = s.PurchaseOrderNumber,
        i.TotalDue = s.TotalDue
WHEN NOT MATCHED BY TARGET THEN
    INSERT (
        SalesOrderID,
        CustomerID,
        OrderDate,
        PurchaseOrderNumber,
        TotalDue
    )
    VALUES (
        s.SalesOrderID,
        s.CustomerID,
        s.OrderDate,
        s.PurchaseOrderNumber,
        s.TotalDue
    )
WHEN NOT MATCHED BY SOURCE THEN
    DELETE
```

Instead of ```WHEN NOT MATCHED BY TARGET THEN```, you can write ```WHEN NOT MATCHED THEN```.

The ```WHEN NOT MATCHED BY SOURCE THEN``` clause is optional.

## [Expressions][22]

Expressions can be column names or calculations.

### [Arithmetic operators][23]

| Operator           | Meaning                                     |
|--------------------|---------------------------------------------|
| [+ (Add)][24]      | Addition                                    |
| [- (Subtract)][25] | Subtraction                                 |
| [* (Multiply)][26] | Muliplication                               |
| [/ (Divide)][27]   | Division                                    |
| [% (Modulo)][28]   | Returns the integer remainder of a division |

### [Comparison operators][6]

| Operator          | Description              | Example               |
|-------------------|--------------------------|-----------------------|
| [=][7]            | Equals                   | ProductCategoryID = 6 |
| [<>][8] or != [9] | Not eqals                | Color <> 'White'.     |
| [>][10]           | Greater than.            | ListPrice > 1000      |
| [>=][11]          | Greater than or equal to | ListPrice >= 1000     |
| [<][12]           | Less than                | ListPrice < 1000      |
| [<=][13]          | Less than or equal to    | ListPrice <= 1000     |

### [Logical Operators][14]

| Operator                                  | Description                                                                            | Example                                                                                                                 |
|-------------------------------------------|----------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| [AND][15]                                 | TRUE if both expressions are TRUE                                                      | ProductCategoryID = 6 AND ListPrice > 1000                                                                              |
| [OR][16]                                  | TRUE if either expression is TRUE                                                      | ProductCategoryID = 6 OR ListPrice > 1000                                                                               |
| [NOT][17]                                 | Reverses the value of the expression                                                   | NOT ProductCategoryID = 6 -- same result as ProductCategoryID <> 6                                                      |
| [IN][18]                                  | TRUE if expression is contained in set; set is a comma-separated list of expressions   | ProductCategoryID IN (2, 4, 6)                                                                                          |
| [BETWEEN lower_bound AND upper_bound][19] | TRUE if expression is greater or equal to lower_bound and less or equal to upper_bound | ListPrice BETWEEN 10 AND 100                                                                                            |
| [LIKE][20]                                | Wildcard comparison of string                                                          | ProductNumber LIKE 'FR-[^0-9][0-9][0-9]_%'                                                                              |
| |[EXISTS][41]                             | TRUE if the subquery contains one or more rows                                         | EXISTS (SELECT * FROM SalesLT.SalesOrderHeader WHERE SalesLT.SalesOrderHeader.CustomerID = SalesLT.Customer.CustomerID) |

#### [LIKE patterns][20]

| Wildcard character | Description                                                                  | Example                          |
|--------------------|------------------------------------------------------------------------------|----------------------------------|
| %	                 | Any string of zero or more characters.                                       | WHERE title LIKE '%computer%'    |
| _ (underscore)     | Any single character.                                                        | WHERE au_fname LIKE '_ean'       |
| [ ]                | Any single character within the specified range [a-f] or set [abcdef].       | WHERE au_lname LIKE '[C-P]arsen' |
| [^]                | Any single character not within the specified range [^a-f] or set [^abcdef]. | WHERE au_lname LIKE 'de[^l]%'    |

### [Functions][42]

[Date & time][43]
[Mathematical][44]

#### [Data type conversion][29]

[Data types (Transact-SQL)][30]

| Function    | Descripton                                                                                                                                                                    | Behavior on invalid values | Example                                   |
|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------|-------------------------------------------|
| [CAST][31]        | Converts from one data type to another.                                                                                                                                       | Throws error               | CAST(CustomerID AS varchar)               |
| [CONVERT][31]     | Allows to specify a style for localization.                                                                                                                                   | Throws error               | CONVERT(nvarchar(30), SellStartDate, 126) |
| [TRY_CAST][32]    | Converts from one data type to another.                                                                                                                                       | Returns NULL               | TRY_CAST(Size AS Integer)                 |
| [TRY_CONVERT][33] | Allows to specify a style for localization.                                                                                                                                   | Returns NULL               |                                           |
| [PARSE][34]       | Converts a string to a numeric, date, or time data type.                                                                                                                      | Throws error               | PARSE('01/01/2021' AS date)               |
| [TRY_PARSE][35]   | Converts a string to a numeric, date, or time data type.                                                                                                                      | Returns NULL               |                                           |
| [STR][36]         | Converts a float to a string. The second parameter is the total length of the resulting string. The third paramter is the number of digits to the right of the decimal point. |                            | STR(ListPrice, 10, 2)                     |

#### Handle NULLs

| Function | Description                                                                                                      | Example                                                |
|----------|------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------|
| [ISNULL][37]   | Returns the result of the first parameter, if it is not NULL, otherwise the result of the second parameter | ISNULL(ParentCategoryID, 0)                            |
| [COALESCE][38] | Return the result of the first parameter which is not NULL.                                                | COALESCE(SellEndDate, DiscontinuedDate, SellStartDate) |
| [NULLIF][39]   | Returns NULL if results of both parameters are equal.                                                      | NULLIF(Color, 'Multi')                                 |

#### [Aggregate functions][46]

| Function Name   | Syntax                                                 | Description                                       |
|-----------------|--------------------------------------------------------|---------------------------------------------------|
| [SUM][47]       | SUM([DISCTINCT] expr)                                  | Returns the sum of non-NULL values.               |
| [AVG][48]       | AVG([DISCTINCT] expr)                                  | Returns the average of non-NULL values.           |
| [MIN][49]       | MIN([DISCTINCT] expr)                                  | Returns the lowest non-NULL value.                |
| [MAX][50]       | MAX([DISCTINCT] expr)                                  | Returns the highest non-NULL value.               |
| [COUNT][51]     | COUNT([DISCTINCT] *) or COUNT([DISCTINCT] expr)        | Counts rows or non-NULL values in the expression. |
| [COUNT_BIG][52] | COUNT_BIG([DISCTINCT] *) or COUNT_BIG([DISTINCT] expr) | Counts rows and returns bigint instead of int.    |

#### [RANK][45]

```SQL
SELECT
    Name, 
    ListPrice,
    ProductCategoryID,
    RANK() OVER(ListPrice DESC) AS PriceRank
FROM SalesLT.Product
```

```SQL
SELECT
    Name, 
    ListPrice,
    ProductCategoryID,
    RANK() OVER(PARTITION BY ProductCategoryID ORDER BY ListPrice DESC) AS PriceRank
FROM SalesLT.Product
```

### [Return one value from multiple conditions][40]

```SQL
CASE Size
    WHEN '46' THEN 'S'
    WHEN '48' THEN 'M'
    WHEN '50' THEN 'L'
    WHEN '52' THEN 'XL'
    ELSE 'XXL'
END
```

The ```ELSE``` clause is optional.

```SQL
CASE
    WHEN Size = '46' THEN 'S'
    WHEN Size = '48' THEN 'M'
    WHEN Size = '50' THEN 'L'
    WHEN Size = '52' THEN 'XL'
    WHEN Size LIKE '5_' THEN 'XXL'
    ELSE 'Invalid'
END
```

[1]: https://learn.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql "SELECT (Transact-SQL) - SQL Server | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/sql/t-sql/queries/select-clause-transact-sql "SELECT clause (Transact-SQL) - SQL Server | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql "FROM clause plus JOIN, APPLY, PIVOT (Transact SQL) - SQL Server | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql "SELECT - ORDER BY clause (Transact-SQL) - SQL Server | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/sql/t-sql/queries/where-transact-sql "WHERE (Transact-SQL) - SQL Server | Microsoft Learn"
[6]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/comparison-operators-transact-sql "Comparison Operators (Transact-SQL) - SQL Server | Microsoft Learn"
[7]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/equals-transact-sql "= (Equals) (Transact-SQL) - SQL Server | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/not-equal-to-transact-sql-traditional "Not equal to (Transact-SQL) - traditional - SQL Server | Microsoft Learn"
[9]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/not-equal-to-transact-sql-exclamation "Not equal to (Transact-SQL) - exclamation - SQL Server | Microsoft Learn"
[10]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/greater-than-transact-sql "> (Greater Than) (Transact-SQL) - SQL Server | Microsoft Learn"
[11]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/greater-than-or-equal-to-transact-sql ">= (Greater Than or Equal To) (Transact-SQL) - SQL Server | Microsoft Learn"
[12]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/less-than-transact-sql "< (Less Than) (Transact-SQL) - SQL Server | Microsoft Learn"
[13]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/less-than-or-equal-to-transact-sql "<= (Less Than or Equal To) (Transact-SQL) - SQL Server | Microsoft Learn"
[14]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/logical-operators-transact-sql "Logical Operators (Transact-SQL) - SQL Server | Microsoft Learn"
[15]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/and-transact-sql "AND (Transact-SQL) - SQL Server | Microsoft Learn"
[16]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/or-transact-sql "OR (Transact-SQL) - SQL Server | Microsoft Learn"
[17]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/not-transact-sql "NOT (Transact-SQL) - SQL Server | Microsoft Learn"
[18]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/in-transact-sql "IN (Transact-SQL) - SQL Server | Microsoft Learn"
[19]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql "BETWEEN (Transact-SQL) - SQL Server | Microsoft Learn"
[20]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql "LIKE (Transact-SQL) - SQL Server | Microsoft Learn"
[21]: https://learn.microsoft.com/en-us/sql/t-sql/queries/is-null-transact-sql "IS NULL (Transact-SQL) - SQL Server | Microsoft Learn"
[22]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/expressions-transact-sql "Expressions (Transact-SQL) - SQL Server | Microsoft Learn"
[23]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/arithmetic-operators-transact-sql "Arithmetic operators (Transact-SQL) - SQL Server | Microsoft Learn"
[24]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/add-transact-sql "+ (Addition) (Transact-SQL) - SQL Server | Microsoft Learn"
[25]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/subtract-transact-sql "- (Subtraction) (Transact-SQL) - SQL Server | Microsoft Learn"
[26]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/multiply-transact-sql "* (Multiplication) (Transact-SQL) - SQL Server | Microsoft Learn"
[27]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/divide-transact-sql "/ (Division) (Transact-SQL) - SQL Server | Microsoft Learn"
[28]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/modulo-transact-sql "% (Modulus) (Transact-SQL) - SQL Server | Microsoft Learn"
[29]: https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-conversion-database-engine "Data type conversion (Database Engine) - SQL Server | Microsoft Learn"
[30]: https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql "Data types (Transact-SQL) - SQL Server | Microsoft Learn"
[31]: https://learn.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql "CAST and CONVERT (Transact-SQL) - SQL Server | Microsoft Learn"
[32]: https://learn.microsoft.com/en-us/sql/t-sql/functions/try-cast-transact-sql "TRY_CAST (Transact-SQL) - SQL Server | Microsoft Learn"
[33]: https://learn.microsoft.com/en-us/sql/t-sql/functions/try-convert-transact-sql "TRY_CONVERT (Transact-SQL) - SQL Server | Microsoft Learn"
[34]: https://learn.microsoft.com/en-us/sql/t-sql/functions/parse-transact-sql "PARSE (Transact-SQL) - SQL Server | Microsoft Learn"
[35]: https://learn.microsoft.com/en-us/sql/t-sql/functions/try-parse-transact-sql "TRY_PARSE (Transact-SQL) - SQL Server | Microsoft Learn"
[36]: https://learn.microsoft.com/de-de/sql/t-sql/functions/str-transact-sql "STR (Transact-SQL) - SQL Server | Microsoft Learn"
[37]: https://learn.microsoft.com/en-us/sql/t-sql/functions/isnull-transact-sql "ISNULL (Transact-SQL) - SQL Server | Microsoft Learn"
[38]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/coalesce-transact-sql "COALESCE (Transact-SQL) - SQL Server | Microsoft Learn"
[39]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/nullif-transact-sql "NULLIF (Transact-SQL) - SQL Server | Microsoft Learn"
[40]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql "CASE (Transact-SQL) - SQL Server | Microsoft Learn"
[41]: https://learn.microsoft.com/en-us/sql/t-sql/language-elements/exists-transact-sql "EXISTS (Transact-SQL) - SQL Server | Microsoft Learn"
[42]: https://learn.microsoft.com/en-us/sql/t-sql/functions/functions "What are the SQL database functions? - SQL Server | Microsoft Learn"
[43]: https://learn.microsoft.com/en-us/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql "Date and Time Data Types and Functions - SQL Server (Transact-SQL)| Microsoft Learn"
[44]: https://learn.microsoft.com/en-us/sql/t-sql/functions/mathematical-functions-transact-sql "Mathematical Functions (Transact-SQL) - SQL Server | Microsoft Learn"
[45]: https://learn.microsoft.com/en-us/sql/t-sql/functions/rank-transact-sql "RANK (Transact-SQL) - SQL Server | Microsoft Learn"
[46]: https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql "Aggregate functions (Transact-SQL) - SQL Server | Microsoft Learn"
[47]: https://learn.microsoft.com/en-us/sql/t-sql/functions/sum-transact-sql "SUM (Transact-SQL) - SQL Server | Microsoft Learn"
[48]: https://learn.microsoft.com/en-us/sql/t-sql/functions/avg-transact-sql "AVG (Transact-SQL) - SQL Server | Microsoft Learn"
[49]: https://learn.microsoft.com/en-us/sql/t-sql/functions/min-transact-sql "MIN (Transact-SQL) - SQL Server | Microsoft Learn"
[50]: https://learn.microsoft.com/en-us/sql/t-sql/functions/max-transact-sql "MAX (Transact-SQL) - SQL Server | Microsoft Learn"
[51]: https://learn.microsoft.com/en-us/sql/t-sql/functions/count-transact-sql "COUNT (Transact-SQL) - SQL Server | Microsoft Learn"
[52]: https://learn.microsoft.com/en-us/sql/t-sql/functions/count-big-transact-sql "COUNT_BIG (Transact-SQL) - SQL Server | Microsoft Learn"
[53]: https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql "GROUP BY (Transact-SQL) - SQL Server | Microsoft Learn"
[54]: https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql "HAVING (Transact-SQL) - SQL Server | Microsoft Learn"
[55]: https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql "INSERT (Transact-SQL) - SQL Server | Microsoft Learn"
[56]: https://learn.microsoft.com/en-us/sql/t-sql/queries/select-into-clause-transact-sql "SELECT - INTO clause (Transact-SQL) - SQL Server | Microsoft Learn"
[57]: https://learn.microsoft.com/en-us/sql/t-sql/functions/scope-identity-transact-sql "SCOPE_IDENTITY (Transact-SQL)"
[58]: https://learn.microsoft.com/en-us/sql/t-sql/functions/ident-current-transact-sql "IDENT_CURRENT (Transact-SQL) - SQL Server | Microsoft Learn"
[59]: https://learn.microsoft.com/en-us/sql/t-sql/statements/create-sequence-transact-sql "CREATE SEQUENCE (Transact-SQL) - SQL Server | Microsoft Learn"
[60]: https://learn.microsoft.com/en-us/sql/t-sql/functions/next-value-for-transact-sql "NEXT VALUE FOR (Transact-SQL) - SQL Server | Microsoft Learn"
[61]: https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql "UPDATE (Transact-SQL) - SQL Server | Microsoft Learn"
[62]: https://learn.microsoft.com/en-us/sql/t-sql/statements/delete-transact-sql "DELETE (Transact-SQL) - SQL Server | Microsoft Learn"
[63]: https://learn.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql "TRUNCATE TABLE (Transact-SQL)  - SQL Server | Microsoft Learn"
[64]: https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql "MERGE (Transact-SQL)   - SQL Server | Microsoft Learn"
