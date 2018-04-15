## sql -`if else`

```sql
IF(condition, value_if_true, value_if_false)
```


## sql - `when case`

simple case 

```sql
SELECT CASE <variable> 
            WHEN <value>      THEN <returnvalue>
            WHEN <othervalue> THEN <returnthis>
            ELSE <returndefaultcase>
       END AS <newcolumnname>
FROM <table>
```

extended case

```sql
SELECT CASE 
            WHEN <test>      THEN <returnvalue>
            WHEN <othertest> THEN <returnthis>
            ELSE <returndefaultcase>
       END AS <newcolumnname>
FROM <table>
```
