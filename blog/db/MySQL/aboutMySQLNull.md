MySQL中的NULL问题

```sql
-- 查a字段是null的情况
select * from tableName where a = null;
select * from tableName where a is null;
```

为什么只有is null起作用？
