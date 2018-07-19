# Where Clauses

| Base | AND | OR |
| --- | --- | --- |
| where\(\) | andWhere\(\) | orWhere\(\) |
| whereBetween\(\) | andWhereBetween\(\) | orWhereBetween\(\) |
| whereColumn\(\) | andWhereColumn\(\) | orWhereColumn\(\) |
| whereExists\(\) | andWhereExists\(\) | orWhereExists\(\) |
| whereIn\(\) | andWhereIn\(\) | orWhereIn\(\) |
| whereNotBetween\(\) | andWhereNotBetween\(\) | orWhereNotBetween\(\) |
| whereNotExists\(\) | andWhereNotExists\(\) | orWhereNotExists\(\) |
| whereNotIn\(\) | andWhereNotIn\(\) | orWhereNotIn\(\) |
| whereNotNull\(\) | andWhereNotNull\(\) | orWhereNotNull\(\) |
| whereNull\(\) | andWhereNull\(\) | orWhereNull\(\) |
| whereRaw\(\) | andWhereRaw\(\) | orWhereRaw\(\) |

## Parameters

By default, the correct sql type will be inferred from your parameters. If you pass a number, `CF_SQL_NUMERIC` will be used. If it is a date, `CF_SQL_TIMESTAMP`, and so forth. If you need more control, you can always pass a struct with the parameters you would pass to `cfqueryparam`.

```text
//qb
var getResults = query.from('users')
    .where('age','>=', { value = 18, cfsqltype = "CF_SQL_INTEGER" })
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `age` >= 18
```

### Simple Where Clauses

You may use the `where` method on a query builder instance to add `where` clauses to the query. The most basic call to `where` requires three arguments. The first argument is the name of the column. The second argument is an operator, which can be any of the database's supported operators. Finally, the third argument is the value to evaluate against the column.

For example, here is a query that verifies the value of the "age" column is greater than or equal to 18:

```text
//qb
var getResults = query.from('users')
    .where('age','>=','18')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `age` >= 18
```

For convenience, if you simply want to verify that a column is **equal** to a given value, you may pass the value directly as the second argument to the `where` method:

```text
//qb
var getResults = query.from('users')
    .where('age','18')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `age` = 18
```

Of course, you may use a variety of other operators when writing a `where` clause:

```text
//qb
var getResults = query.from('users')
    .where('age','>=','18')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `age` >= 18

//qb
var getResults = query.from('users')
    .where('age','<>','18')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `age` <> 18


//qb
var getResults = query.from('users')
    .where('name','like','A%')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `name` LIKE 'A%'
```

### Or Statements

You may chain where constraints together as well as add `or` clauses to the query. The `orWhere` method accepts the same arguments as the `where` method:

```text
//qb
var getResults = query.from('users')
    .where('name','like','A%')
    .orWhere('age','>','30')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `name` LIKE 'A%' OR `age` > 30
```

### Additional Where Clauses

**whereBetween** / **whereNotBetween**

The `whereBetween` method verifies that a column's value is between two values:

```text
//qb
var getResults = query.from('users')
    .whereBetween('age','18','21')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `age` BETWEEN 18 AND 21
```

The `whereNotBetween` method verifies that a column's value lies outside of two values:

```text
//qb
var getResults = query.from('users')
    .whereNotBetween('age','18','21')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `age` NOT BETWEEN 18 AND 21
```

**whereIn / whereNotIn** (sub-queries)

The `whereIn` method verifies that a given column's value is contained within the provided array or QueryBuilder object:

Array:

```text
var getResults = query.from('users')
    .whereIn('age',[ 17, 18, 19, 20, 21 ])
    .get();
writeDump(getResults);
```

QueryBuilder (fetch all users whose age is in the all_ages table with a value between 17 and 21):

```text
var getResults = query.from('users')
       .whereIn('age', function ( subQuery ) {
        return subQuery.select( "age" )
        .from( "all_ages" )
        .whereBetween("age","17","21");
    })
    .get();
writeDump(getResults);
```

The `whereNotIn` method verifies that the given column's value is **not** contained in the provided array of QueryBuilder object:

```text
var getResults = query.from('users')
    .whereNotIn('age',[ 17, 18, 19, 20, 21 ])
    .get();
writeDump(getResults);
```

**whereNull / whereNotNull**

The `whereNull` method verifies that the value of the given column is `NULL`:

```text
//qb
var getResults = query.from('users')
    .whereNull('modifiedDate')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `modifiedDate` IS NULL
```

The `whereNotNull` method verifies that the column's value is not `NULL`:

```text
//qb
var getResults = query.from('users')
    .whereNotNull('modifiedDate')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `modifiedDate` IS NOT NULL
```

**whereExists / whereNotExists**

The `whereExists` method:

```text
//qb
var getResults = query.from('users')
    .whereExists( function( q ) {
        q.select( q.raw( 1 ) ).from( "departments" )
            .where( "departments.ID", "=", q.raw( '"users.FK_departmentID"' ) );
    })
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE EXISTS (SELECT 1 FROM `departments` WHERE `departments`.`ID` = "users.FK_departmentID")
```

**whereColumn**

The `whereColumn` method may be used to verify that two columns are equal:

```text
//qb
var getResults = query.from('users')
    .whereColumn('modifiedDate','createdDate')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `modifiedDate` = `createdDate`
```

You may also pass a comparison operator to the method:

```text
//qb
var getResults = query.from('users')
    .whereColumn('modifiedDate','<>','createdDate')
    .get();
writeDump(getResults);

//sql
SELECT * FROM `users` WHERE `modifiedDate` <> `createdDate`
```

**WHERE (a = ? OR b = ?) AND c = ?**

Here is an example of how to strategically place parentheses with `OR` using closures.

```text
//qb
var getResults = query.from('users')
    .where( function( q ) {
        q.where( "a", 1 ).orWhere( "b", 2 );
    } )
    .where( "c", 3 );
writeDump(getResults);

// sql
SELECT * FROM `users` WHERE (a = ? OR b = ?) AND c = ?
```
