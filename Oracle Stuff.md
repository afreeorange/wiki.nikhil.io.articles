List of Fields in Table/View
----------------------------

    select * from all_tab_columns where table_name = 'TABLE' order by column_id

[Limiting Resultset](http://www.oracle.com/technetwork/issue-archive/2006/06-sep/o56asktom-086197.html)
-------------------------------------------------------------------------------------------------------

Need to use the `ROWNUM` pseudo-column.

    select * from (
        select * from tb_sample
    ) where ROWNUM <= 100

See the article for LIMIT and OFFSET equivalent.

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Oracle](Category:_Oracle "wikilink")
