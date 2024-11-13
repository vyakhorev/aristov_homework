# Transaction isolation levels

## Start two wsl consoles

`psql -U postgres` in both wsl consoles - A and B


## read committed

`console A`

```
BEGIN;
CREATE TABLE ideas(id serial, priority numeric, summary text);
INSERT INTO ideas(priority, summary) VALUES (0, 'wheat has corrupted humanity');
INSERT INTO ideas(priority, summary) VALUES (1, 'the weather is fine');
COMMIT;
```

`console B`

Looks as expected
```
postgres=# select * from ideas;
 id | priority |           summary
----+----------+------------------------------
  1 |        0 | wheat has corrupted humanity
  2 |        1 | the weather is fine
```

`show transaction isolation level;` -> `read committed`

Now both `console A` and `console B`

```
begin
```

`console A`

```
INSERT INTO ideas(priority, summary) VALUES (42, 'the cake is a lie');
```

`console B`

We cannot see the result. Cause with `read committed` we won't read data that is not committed by another transaction.

```
postgres=*# select * from ideas;
 id | priority |           summary
----+----------+------------------------------
  1 |        0 | wheat has corrupted humanity
  2 |        1 | the weather is fine
(2 rows)
```

`console A`

```
commit
```

`console B`

Well, now since the transaction A is committed we can see the new record. So with `read committed` we can see data that's been committed by another transaction during our transaction.

```
postgres=*# select * from ideas;
 id | priority |           summary
----+----------+------------------------------
  1 |        0 | wheat has corrupted humanity
  2 |        1 | the weather is fine
  3 |       42 | the cake is a lie
```

`console B`

```
commit
```

## repeatable read

Both `console A` and `console B`


```
BEGIN transaction isolation level repeatable read;
```

`console A`

```
INSERT INTO ideas(priority, summary) VALUES (100500, 'cats are funny');
```

`console B`

We cannot see the result. Cause with `repeatable read` we still do not see dirty records from another transaction. 

```
postgres=*# select * from ideas;
 id | priority |           summary
----+----------+------------------------------
  1 |        0 | wheat has corrupted humanity
  2 |        1 | the weather is fine
  3 |       42 | the cake is a lie
(3 rows)
```

`console A`

close the repeatable read transaction

```
commit
```

`console B`

We still cannot see the record since with `repeatable read` we do not want to see new records while we're performing our transaction

```
postgres=*# select * from ideas;
 id | priority |           summary
----+----------+------------------------------
  1 |        0 | wheat has corrupted humanity
  2 |        1 | the weather is fine
  3 |       42 | the cake is a lie
(3 rows)
```

```
commit
```

Now after we closed our transaction we finally see the record in a default read committed transaction

```
postgres=# select * from ideas;
 id | priority |           summary
----+----------+------------------------------
  1 |        0 | wheat has corrupted humanity
  2 |        1 | the weather is fine
  3 |       42 | the cake is a lie
  4 |   100500 | cats are funny
```

