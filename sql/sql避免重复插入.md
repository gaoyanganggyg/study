## `使用 IGNORE 关键字`

- 1、
    ```sql
    INSERT IGNORE INTO tab(col1, col2)
    VALUES(val1, val2)
    ```

    有重复就会忽略, 执行后返回0

- 2、
    ```sql
    INSERT IGNORE INTO tab(col)
    SELECT col FROM tab1
    ```
    这样复制表可以避免重复复制

## `使用 replace`

```sql
REPLACE INTO tab1(col1, ...) VALUES(val1, ...)
REPLACE INTO tab2(col1, ...) SELECT .....
REPLACE INTO tab3 SET col=val
```

REPLACE 的运行和 INSERT 很相似， 但是如果旧记录与新记录有想用的值， 则在新记录插入之前删除就记录`
当对于主键或者唯一索引出现重复而造成插入失败时：

    1、从表中删除含有重复记录的行
    2、再尝试把新的行插入到表中

## `ON DUPLACATE KEY UPDATE`

当插入导致PRIMARY KEY 或者 UNIQUE KEY 出现重复值，则执行UPDATE

```sql
INSERT INTO tab(col1, col2, col3, col4) VALUES(val1, val2, val3, val4)
ON DUPLICATE KEY UPDATE  val3=val3+1, val4=val4+1
``` 