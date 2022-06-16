---
title: SQL 中使用 boolean logic 代替 CASE
date: 2021-10-30
tags: [sql]
---

# SQL WHERE clauses: Avoid CASE, use Boolean logic

## Why ?

- portable
- easier to read
- ❤ efficient

## How ?

First off, when I say “conditional logic”, I am talking about something like this: `IF A THEN B`

Where A and B are both conditions. For example, in a WHERE clause, you might want to implement a condition like this: `IF (@ReturnAll <>1) THEN (EmpID = @EmpID)`

In other words, if the @ReturnAll parameter is 1, then return all of the rows, but if @ReturnAll is not 1, then only return rows where EmpID is equal to the @EmpID parameter supplied.

To express this logic in the WHERE clause, many people might code it like this:

`WHERE EmpID = CASE WHEN @ReturnAll<>1 THEN @EmpID ELSE EmpID END`

However, this is kind of counter-intuitive (why should we check that EmpID = EmpID ?) and can be really tough to implement when the condition spans more than 1 column in the table (you need multiple CASE's). Also, if EmpID is null this will fail.

The alternative is to translate the condition into a regular boolean expression using only AND, OR and NOT. The logical translation of “IF A then B” is: `(Not A) or B`

If you work it out on paper, you will see it makes sense. To translate our WHERE clause requirement using the above logic, it becomes: `WHERE (@ReturnAll =1) OR (EmpID = @EmpID)`

## More examples

### A

Suppose we wish to say: `IF @Filter=1 THEN Date= @Date and Cust= @Cust and Emp= @Emp`

Expressing this in a CASE clause results in:

```sql
WHERE Date = CASE WHEN @Filter=1 THEN @Date ELSE Date END AND
Cust = CASE WHEN @Filter=1 THEN @Cust ELSE Cust END AND
Emp = CASE WHEN @Filter=1 THEN @Emp ELSE Emp END
```

A little hard to read and quite inefficient – all 3 case expressions must be evaluated for each row in the result set. Without CASE, we get: `WHERE @Filter<>1 OR (Date= @Date and Cust= @Cust and Emp= @Emp)`

Much easier to read and maintain, and faster – if @Filter <>1, the rest of the expression can be ignored by the optimizer. 

### B

Using a single variable to implement the optional filter. For example: `IF @CustID is not null THEN CustID = @CustID`

This is often implemented using ISNULL() or COALESCE() like this: `WHERE CustID = ISNULL(@CustID, CustID)`

This is basically the same as writing a CASE expression in that it will not use an index on our column and doesn't implement solid boolean logic. 

Converting that IF to a simple boolean expression results in a nice WHERE clause of: `WHERE (@CustID is null OR CustID = @CustID)`

Which, again, is the preferred way to implement this type of logic in SQL.  It is short, simple, portable, easy to read and maintain, and efficient.

## Finally,

To express: `IF A THEN B ELSE C`
You would write it as: `((Not A) or B) AND (A or C)`

A little harder, but it does the job! No need for CASE in the WHERE clause

## Reference

[SQL WHERE clauses: Avoid CASE, use Boolean logic](https://weblogs.sqlteam.com/jeffs/2003/11/14/513/)

