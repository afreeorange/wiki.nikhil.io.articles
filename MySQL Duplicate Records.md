```sql
SELECT      column, COUNT(column) AS column_count 
FROM        table
GROUP BY    column
HAVING      (COUNT(column) > 1)
```
