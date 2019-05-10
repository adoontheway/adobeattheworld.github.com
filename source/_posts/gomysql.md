---
title: Golang 连接 Mysql
author: ado
date: 2019-05-07 19:17:29
description: 准备Golang的 Mysql 测试，由于第三方库文档不是很齐全，所以记录在这里
tags: [go, mysql]

---

# Preface

最近在做一系列的准备工作，包括 Golang 和很多软件以及工具的配合。

今天是测试连接之前搭建好的 mysql 容器。

由于 **go-sql-driver/mysql** 的文档比较晦涩，所以需要通过阅读源码配合文档来进行测试，所以才有此记录的必要。

# Dependency

## go-sql-driver

```sh
go get -u github.com/go-sql-driver/mysql
```

[go-sql-driver:Github](https://github.com/go-sql-driver/mysql)

[go-sql-driver:GoDoc](https://godoc.org/github.com/go-sql-driver/mysql)

# Test Code

```go
package test

import (
	"database/sql"
	"fmt"
	"github.com/go-sql-driver/mysql"
	"testing"
	"time"
)

func TestMysql(t *testing.T) {
	config := mysql.NewConfig()
	config.User = "root"
	config.Passwd = "12345"
	config.Net = "tcp"
	config.Addr = "192.168.2.127"
	config.ReadTimeout = time.Second * 5
	config.WriteTimeout = time.Second * 5
	config.Timeout = time.Second * 5
	//mysqldriver := &mysql.MySQLDriver{}
	dnsstr := config.FormatDSN()
	//mysqldriver.Open(dnsstr)

	db, err := sql.Open("mysql", dnsstr)
	if err != nil {
		fmt.Println("Error on conntecting : ", err)
		t.FailNow()
	}
	defer db.Close()

	err = db.Ping()
	if err != nil {
		fmt.Println("Error on ping : ", err)
		t.FailNow()
	}
}

```

# Result

```sh
=== RUN   TestMysql
--- PASS: TestMysql (0.00s)
PASS

Process finished with exit code 0
```

