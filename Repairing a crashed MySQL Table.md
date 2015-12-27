You may be chugging along, doing your thing, logging on to MySQL, and
issuing something like this:

    mysql> select * from testdatabase.user;

Your smile may lose its luster when you're confronted with this:

    ERROR 145 (HY000): Table './testdatabase/user' is marked as crashed and should be repaired

Don't worry. There are a few things you can do to fix this.

### When your job is easy

Usually, issuing this should fix the issue:

    mysql> repair table testdatabase.user;

This, if all goes well, will produce output sort of like this:

    +----------------------+--------+----------+------------------------------------------+  
    | Table                | Op     | Msg_type | Msg_text                                 |  
    +----------------------+--------+----------+------------------------------------------+  
    | testdatabase.user    | repair | warning  | Number of rows changed from 0 to 1151697 |  
    | testdatabase.user    | repair | status   | OK                                       |  
    +----------------------+--------+----------+------------------------------------------+  
    2 rows in set (2 min 7.91 sec)

### When your job gets harder

It gets frustrating when you get something like this:

    |------------|--------|----------|-------------------------------------------------------------------------|
    |   Table    |   Op   | Msg_type |                                 Msg_text                                |
    |------------|--------|----------|-------------------------------------------------------------------------|
    | mysql.user | repair | Error    | Table './testdatabase/user' is marked as crashed and should be repaired |
    | mysql.user | repair | Error    | Table 'user' is marked as crashed and should be repaired                |
    | mysql.user | repair | status   | Table is already up to date                                             |
    |------------|--------|----------|-------------------------------------------------------------------------|
    3 rows in set (0.00 sec)

DON'T PANIC. This can be fixed by quitting MySQL, and doing this:

    cd /var/lib/mysql/testdatabase  
    myisamchk --safe-recover user

`user`, of course, is the table you wanted to recover. If the Gods have
mercy, you will see something like this:

    Warning: option 'key_buffer_size': unsigned value 18446744073709551615 adjusted to 4294963200  
    Warning: option 'read_buffer_size': unsigned value 18446744073709551615 adjusted to 4294967295  
    Warning: option 'write_buffer_size': unsigned value 18446744073709551615 adjusted to 4294967295  
    Warning: option 'sort_buffer_size': unsigned value 18446744073709551615 adjusted to 4294967295  
    - recovering (with keycache) MyISAM-table 'user'  
    Data records: 81  
    Duplicate key  1 for record at       7918 against new record at       3959  
    Duplicate key  1 for record at       8132 against new record at       3638  
    Duplicate key  1 for record at       8239 against new record at       1498  
    Duplicate key  1 for record at       8560 against new record at       4280  
    Data records: 77  
    myisamchk: warning: 4 records have been removed

Ta da! Assuming, of course, that your table's the default MyISAM type.
