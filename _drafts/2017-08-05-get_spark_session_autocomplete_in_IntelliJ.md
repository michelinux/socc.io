---
layout: post
title:  Get a `spark` session autocomplete in IntelliJ
date:   2017-08-05 12:03:00 +0200
categories: spark IntelliJ scala
tags: spark spark-shell Intellij scala SparkSession autocomplete
---

I really like to play with the `spark-shell`. It's a quick way to try something. Sometimes, however, one would like to write those snippets in a proper file, and have some synthax checks and highlight.

I personally use IntelliJ, but you should able to adapt this example.

Once you created your script file, you can add the following at the beginning of your file:

```scala
import org.apache.spark.sql.SparkSession
val spark = SparkSession.builder()
  .appName("Spark shell")
  .config("spark.sql.warehouse.dir", "file:/tmp/spark-warehouse")
  .master("local")
  .getOrCreate()
import spark.implicits._
```

<!-- readmore -->

The first two lines are used to create a `spark` session object. The `spark-shell` is creating it for you but you don't have it (yet) when writing in IntelliJ. It basically create this object so that IntelliJ does not complain too much about the syntax and you have all the autocompletions that a good `CTRL+Space` can give you. On the other hand, when you use this file in the `spark-shell` the `spark` object you created will be kindly ignored in favor of the local instance (it's not exactly like this, but the effect is pretty much the same).

![spark autocomplete in IntelliJ](/images/2017-08-05-spark-autocomplete-intellij.png)

The `import spark.implicits._` allows you to use in IntelliJ all the implicit conversions provided by `spark` on top of the native types of `scala` (thanks to the `scala` Implicits. For example, you can write:

```scala
val myList = "Michele" :: "Maggie" :: "Vincenzo" :: "Luca" :: Nil
val myDF = myList.toDF
```
despite the fact that the `List` type in `scala` has no `toDF` method.