# Postgresql

## Working with data

Say you have a colum with a value like this: `'123,321'`. It's a comma-separated list of numbers. You can:
- Convert the string to an array: `string_to_array('123,321', ',')`. Let's refer to the result as `array`.
- Count the number of values in the array: `cardinality(array)`
- Unnest the array, so each value becomes a row of its own: `unnest(array)`.
  

## Protecting data
Use the DCL (Data Control Language) to `grant` or `revoke` permissions to database objects.

A DCL goes like this:
```postgresql
GRANT -- statement
SELECT -- Permission
ON TABLE -- Object
TO CURRENT_USER -- Role
;

REVOKE -- statement
SELECT -- Permission
ON TABLE -- Object
FROM CURRENT_USER -- Role
;
```

## Use aggregate functions in HAVING
Remember that you can use aggregate functions in HAVING clauses.

```postgresql
SELECT *
FROM some_table
GROUP BY value
HAVING COUNT(*) = 1
```
