List of Fields in Table/View
----------------------------

```sql
select * from all_tab_columns where table_name = 'TABLE' order by column_id
```

Limiting Resultset
------------------

Need to use the `ROWNUM` pseudo-column.

```sql
select * from (
    select * from tb_sample
) where ROWNUM <= 100
```

See [this](http://www.oracle.com/technetwork/issue-archive/2006/06-sep/o56asktom-086197.html) 
for `LIMIT` and `OFFSET` equivalent.
