---
title: MySQL Binary Log
tags:
  - dev
  - databse
  - sql
  - mysql
---

MySQL Binary Log is often used to append `watchers` on the table changes. I used it to react on new insert on the separate service (it can be also achieved using event pattern)


## Storage

One of drawback is storage. Day by day it get a much space and from some point server can start thrownig error that there is no space on disk. So we must find sollution to clear periodically old Binary Log entries.

You can check logs using `SHOW BINARY LOGS`

MySQL statamenet to delete logs

```
PURGE BINARY LOGS {
    TO 'log_name'
  | BEFORE datetime_expr
}
```

Usage example
```
PURGE BINARY LOGS TO 'mysql-bin.010';
PURGE BINARY LOGS BEFORE '2019-04-02 22:46:26';
```

Don't delete raw files form using linux commands etc:

> PURGE BINARY LOGS TO and PURGE BINARY LOGS BEFORE both fail with an error when binary log files listed in the .index file had been removed from the system by some other means (such as using rm on Linux). (Bug #18199, Bug #18453) To handle such errors, edit the .index file (which is a simple text file) manually to ensure that it lists only the binary log files that are actually present, then run again the PURGE BINARY LOGS statement that failed.

## Delete logs

You can use set `binlog_expire_logs_seconds` system veriabel, default is 30 days (2592000).

When thee are deleted:
> After their expiration period ends, binary log files can be automatically removed. Possible removals happen at startup and when the binary log is flushed

When the logs are flushing:
The server creates a new file (for binary log) in the series each time any of the following events occurs:
- The server is started or restarted
- **The server flushes the logs.**
- The size of the current log file reaches max_binlog_size.

When the logs are flusing:
> Log flushing occurs when you issue a FLUSH LOGS statement; execute mysqladmin with a flush-logs or refresh argument; or execute mysqldump with a --flush-logs option [...]. In addition, the binary log is flushed when its size reaches the value of the max_binlog_size system variable.


So change the `binlog_expire_logs_seconds` and it will do the job. You can check current veriable value using `SHOW VARIABLES LIKE 'binlog_expire_logs_seconds'`. Here is command to change veriable value: `SET GLOBAL binlog_expire_logs_seconds = 259200;`
