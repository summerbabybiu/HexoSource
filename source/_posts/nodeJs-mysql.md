---
layout: post
title: nodeJs实现mysql事务
date: 2017-12-27 19:23:45
tags: mysql
---

嗷嗷，接了个老项目，没有成熟的ORM，emmm... 只能单独去实现。😊
<!--more-->

### 依赖
- [mysql](https://github.com/mysqljs/mysql)
- [async](https://github.com/caolan/async)

### 实现
```javascript
const mysql = require('mysql');
const async = require('async');

// 定义连接对象
const pool = mysql.createPool({
    host: 'XXXX',
    user: 'XXXx',
    password: 'XXX',
    database: 'databaseName',
    connectionLimit: 10,
    port: 3306,
    waitForConnections: false,
    charset: 'utf8mb4' // 如果不设置，将无法存储 emoji
    });

function execTrans(sqlparamsEntities, callback) {
    pool.getConnection(function(err, connection) {
        if(err) {
            return callback(err, null);
        }
        connection.beginTransaction(function(err) {
            if(err) {
                return callback(err, null);
            }
            console.log(`开始执行transaction，共执行${sqlparamsEntities.length}条数据`);
            const funcAry = [];
            sqlparamsEntities.forEach(function(sqlParam) {
                const temp = function(cb) {
                    const sql = sqlParam.sql;
                    const param = sqlParam.params;
                    connection.query(sql, param, function(tErr) {
                        if(tErr) {
                            return cb(tErr);
                        } else {
                            return cb(null, 'ok');
                        }
                    });
                };
                funcAry.push(temp);
            });

            async.series(funcAry, function(err, result) {
                console.log(`end,transaction error:${err}`);
                if(err) {
                    connection.rollback(function(err1) {
                        console.log(`transaction error:${err1}`);
                        connection.release();
                        return callback(err, null);
                    });
                } else {
                    connection.commit(function(err) {
                        console.log('transaction info:');
                        if(err) {
                            console.log(`执行事务失败，${err}`);
                            connection.rollback(function(err) {
                                console.log(`transaction error:${err}`);
                                connection.release();
                                return callback(err, null);
                            });
                        } else {
                            connection.release();
                            return callback(null, result);
                        }
                    });
                }
            });
        });
    });
}

```

### 调用
```javascript
const agree = [
    {
        sql: 'INSERT INTO `api_group_history` SET ?',
        params: historyInfo
    },
    {
        sql: 'DELETE FROM `api_group_api` where (`api_group_id` = ?) AND (`api_id` in (?))',
        params: [ apiGroup.id, apiGroupCheck.newApis ]
    },
    {
        sql: 'DELETE FROM `api_group_check` WHERE (`id` = ?) LIMIT 1',
        params: id
    }
];

 dbHelper.execTrans(agree, function(err, result) {
    if(err) {
        // 事务执行失败
    }
    // 事务执行成功
});

```
