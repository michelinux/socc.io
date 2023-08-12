---
layout: post
title:  "Spark and MySQL"
date:   2017-09-29 08:22:00 +0200
categories: spark
tags: spark mysql JDBC
---

## The MySQL Connector

Do you already have all the libraries to connect MySQL? Get the hands on your `spark-shell` and check with:

```scala
Class.forName("com.mysql.jdbc.Driver")
```

If you get an exception, then you are missing the MySQL connector.

Download it. There is a jar inside the tarball, that's all you need. Just tell `spark` where the file is:

```sh
spark-shell --jars mysql-connector-java-5.1.44-bin.jar
```

You won't get the exception anymore.


## Connecting and reading a table

Let's assume that you have a MySQL instance running, for example [using Docker on your machine]({ site.baseurl }}{% link _posts/2017-09-14-Run-MySQL-on-Mac-with-Docker.md %}).

You have to provide a few informations:
* **username** and **password**
* **hostname** and **port**, of the MySQL Server
* **Database** that you want to use

We use the **hostname**, **port** and **Database** to create a `jdbc` URL. The **username** and **password** will be passed to the reader via a `java.util.Properties` object.

We are going to use the `jdbc` method of the `DataFrameReader`. The `jdbc` is pretty general, so you will have to tell it which `driver` it should use to connect. Hint: for MySQL it's `com.mysql.jdbc.Driver`.

Now that we have all the information we need, let's read a table called `HEROES` from the database `MY_DATABASE` stored in our local MySQL:

```scala
val host = "localhost"
val port = 3306
val database = "MY_DATABASE"
val url = s"jdbc:mysql://${host}:${port}/${database}"

val props = new java.util.Properties
props.put("user", "root")
props.put("password", "password")

val heroes = spark.read.option("driver", "com.mysql.jdbc.Driver").jdbc(url, "HEROES", props)
heroes.show
```



```scala
val heros = spark.read.option("driver", "com.mysql.jdbc.Driver").jdbc(jdbcUrl, "HEROES", connectionProperties)
```

You can also create a new table in the database:

```scala
import java.sql.Date

val newData = List(
        ("Adolf Hitler", "Germany", Date.valueOf("1945-04-30")),
        ("Benito Mussolini", "Italy", Date.valueOf("1945-04-28")),
        ("Francisco Franco", "Spain", Date.valueOf("1975-11-20")))
  
val villains = newData.toDF("NAME", "COUNTRY", "DIED")

villains.write.option("driver", "com.mysql.jdbc.Driver").jdbc(url, "VILLAINS", props)
```

And you can access it from the MySQL client:

```mysql
use MY_DATABASE
select * from VILLAINS;
```

```
+------------------+---------+------------+
| NAME             | COUNTRY | DIED       |
+------------------+---------+------------+
| Benito Mussolini | Italy   | 1945-04-28 |
| Francisco Franco | Spain   | 1975-11-20 |
| Adolf Hitler     | Germany | 1945-04-30 |
+------------------+---------+------------+
```

## References

[https://docs.databricks.com/spark/latest/data-sources/sql-databases.html](https://docs.databricks.com/spark/latest/data-sources/sql-databases.html)

[https://dev.mysql.com/downloads/connector/j/](https://dev.mysql.com/downloads/connector/j/)

