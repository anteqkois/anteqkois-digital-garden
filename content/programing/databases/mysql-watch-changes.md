---
title: Watch MySQL database changes
tags:
  - dev
  - databse
  - sql
  - mysql
---

To watch for changes in MySQL database tables you can use my forked package [zongji](https://github.com/anteqkois/zongji) which use [binary logs](programing/databases/mysql-binary-log) to retrive changes.

## Simple TypeScript usage

```ts
const connectionConfig: mysql.PoolOptions = {
  port: process.env.PORT as unknown as number,
  host: process.env.HOST,
  user: process.env.USER,
  password: process.env.PASSWORD,
  database: process.env.DB_NAME,
  queueLimit: 0,
  enableKeepAlive: true,
  keepAliveInitialDelay: 0,
  connectionLimit: 5,
}

const altcoinWatcher = new ZongJi(connectionConfig)

const handleChangeInAltcoinsStream() {
  return async (event: ZongJiEvent) => {
    // bussines logic
  }
}

altcoinWatcher.on('binlog', handleChangeInAltcoinsStream())
```