# SQL For Back-end Developers
Start thinking of your data as sets, not row by row.  

Back-end developers are typically accustomed to working with data by the row, and accessing data is done thru an object-relational mapping (ORM) framework, like EF Core for .NET developers, removing the necessity of writing SQL statements in the daily work.

It is a great idea to take some time to learn just the very basics of SQL and what kind of heavy lifting your ORM is doing for you.

There's also a high chance that you'll need to access the database and review data at some point, so, why not get very familiar with SQL?

## Selecting data
I'll cover the basics of selecting data first. My guess is you're more likely to spend time retrieving data than creating schemas, tables, and so forth.   

You're likely already familiar with `SELECT` or at least is going to be in a very short time.

At a basic level, `SELECT` is easy to understand. But, you're likely to encounter some of the most crazy expresions, containing IF-ELSE, ternary statements, data transformations and whatnot.

One thing to notice though, is the order of execution.
```sql
SELECT -- 5
FROM  -- 1
WHERE -- 2
GROUP BY -- 3
HAVING -- 4
ORDER BY -- 6
OFFSET - FETCH -- 7
```
The source data set is prepared first and rows are filtered based on a predicate, then rows are combined into groups. Next, groups can be filtered by the HAVING clause, which is essentially a WHERE at group level. At this point, we're ready to select what to return to the client. The rows may then be ordered and the returned result. 

When you're selecting data, you might also want to apply some kind of logic or transformation to the individual columns, or, make
computed columns that are not represented in the table.

```sql
SELECT Id,
       SeniorityInYears,
       CASE
         WHEN condition THEN 'Junior'
         WHEN other condition THEN 'Senior'
       END AS Seniority  
       IIF(Name IS NULL, 'Unknown', Name) AS [Name] -- ternary
FROM [Employees]
```
The `AS` keyword is used as aliasing. Sometimes you want to show different column names than the actual table's column names.

## Data integrity and transactions.
```sql
BEGIN TRANSACTION optional_name
-- statements
COMMIT TRANSACTION
```
A transaction is a single, logical unit of work that may contain one or more statements. A transaction begins when the first statement is executed, and ends with a transaction commit or rollback. All statements wrapped in a transaction must complete successfully in order to commit the changes, otherwise, the whole transaction is rolledback.

You may also rollback transactions yourself even if the statements completed successfully.  
You can even make save points allowing you to revert back to a point of the transaction if something fails.

Transactions are carried out in different modes.
- Auto commit is the default. Every statement is treated as a transaction.
- Explicit transactions are started with `BEGIN TRANSACTION` and ends with a `COMMIT|ROLLBACK TRANSACTION`.
- Implicit transactions starts after each `COMMIT|ROLLBACK`.

While a transaction is taking place, a lock is placed on the records that are being modified. This effectively means you cannot query those records. First when the transaction is committed or rollback the lock is released and you can query against the records again.  

You can however overcome this by using the `NOLOCK` hint while executing select statemments. But you do run the risk of retrieving data that's out of sync.

```sql
SELECT *
FROM [Employees]
WITH (NOLOCK) -- <- this
WHERE Id = 'GUID'
```

### Isolation levels
Transactions work with levels of isolation.
- Pessimistic: assume conflicts will take place and take preventive measures to avoid conflicts.
- Optimistic: assume conflicts are rare but requires validity check before committing.

### Rollback
Let's take a moment to revist this keyword. Rollingback a transaction means to return data to a previous state, that is, back to the start of the current transaction.

```sql
BEGIN TRANSACTION
    INSERT INTO [Employees] (Name, Level)
    VALUES ('Niels Bohr', 'Staff Scientist')

    SELECT * FROM [Employees] -- <- you will see the new, uncomitted employee record
ROLLBACK
```

Rollbacks like these are great when you want to try out something and see what happens, without actually committing the state to the database.