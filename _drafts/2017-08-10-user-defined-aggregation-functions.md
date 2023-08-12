---
layout: post
title:  "User-Defined Aggregation Functions"
date:   2017-08-12 09:18:05 +0200
categories: spark UDAF scala
---

As I’m learning Spark (and Scala) some of the topics are a little bit too difficult for me.
One of them is the **User-Defined Aggregation Functions**, a topic discussed in Chapter 5 of the *Spark: The definitive guide, 1st Edition* by by Bill Chambers and Matei Zaharia (btw, I great book that I highly recommend).
I find the example in the book a bit _meagre_. It doesn't explain what's going on, so I decided to have a better look at those. What I wrote here is what I learned, the way I understood it. Woudl you find any mistake, please ping me on twitter, or do a pull request on this same blog.

Let's say that you want to calculate the percentile of a column. Just to give you an example, if you had a table with all the income for each USA citizen plus some other data, you could use it to calculate:

* Under which amount the poorest 10% of the population lives by state
  ```df.groupBy("state").agg(percentile("income", 0.1))```

* What is the minumum income for the top 1% by age
  ```df.groupBy("age").agg(percentile("income", 0.99))```

And so on.

We are going to use the data provided with the *Spark: The Definitive Guide, 1st Edition* which are [available on GitHub](https://github.com/databricks/Spark-The-Definitive-Guide).

## Easy things first. Returning a Random Int

Before creating the `Percentile` UDAF, we will create a much simpler one, that I'll call `RandomAgg`. It will simply produce a random number everytime you call it on aggregated items.

We start with
```scala
import org.apache.spark.sql.Row
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{DataType, IntegerType, StructType}

import scala.util.Random

class RandomAgg extends UserDefinedAggregateFunction {
  override def inputSchema: StructType = ???

  override def bufferSchema: StructType = ???

  override def dataType: DataType = ???

  override def deterministic: Boolean = ???

  override def initialize(buffer: MutableAggregationBuffer): Unit = ???

  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = ???

  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = ???

  override def evaluate(buffer: Row): Any = ???
}
```

The first things to implement are `deterministic`, which tells you whether you will always get the same output given that you provide the same input. Since it's a Random function, this will be a `False`.

The second easy one is the `dataType`, which is the Spark type we are returning.
```scala
  override def dataType: DataType = IntegerType

  override def deterministic: Boolean = False
```

### `inputSchema`

The `inputSchema` method has to return a `StructType` that *represents data types of input arguments of this aggregate function*. A practical way of understanding what `inputSchema` should return is to take a sample `DataFrame`, `select` the columns on which you want to apply your UDAF and have a look at the resulting `schema`. For example if I want to operate on two columns called `"Ticket Price"` and `"Extra Charges"`:

```scala
> sampleDF.select("Ticket Price", "Extra Charges").schema
StructType(StructField(Ticket Price,IntegerType,true), StructField(Extra Charges,IntegerType,true))
```

The `StructType.apply` method actually requires a `Seq` or an `Array`, and the name of the column needs to be passed as a `String`. So you would get

```scala
StructType(
  StructField("Ticket Price",IntegerType,true) :: 
  StructField("Extra Charges",IntegerType,true) :: 
  Nil)
```

In the case of the `RandomAgg` UDAF let's say that we want to operate on one column, holding an `int`. Let's use a slightly different trick: we create a *virgin* `DataFrame` with the structure that we have in mind, and use the `.schema` to have a look at how the schema is defined:

```scala
> spark.range(1).toDF.withColumn("value", lit(10)).select("value").schema
StructType(StructField(value,IntegerType,false))
```

In this case, because of the `lit(10)` the `nullable` value (which is the third argument of the `StructField` is `False`. In most of the case one would prefer `true`. If omitted, it will be `true` by default.

So in our code it will become

```scala
override def inputSchema: StructType = StructType(StructField("value",IntegerType) :: Nil)
```


### `bufferSchema`

Let's talk about the `bufferSchema` method. This method answers to the question: *what kind of data are we going to keep while been fed with more input?* It returns a `StructType`. It called by Spark to know how the data should be stored during the computation. For now we can put the `bufferSchema` aside, as to generate a random number we don't need to store anything, just let's imagine we want to store an `Integer`:

```scala
override def bufferSchema: StructType = StructType(StructField("result",IntegerType) :: Nil)
```

### `initialize`, `update` and `merge`

Those three are the methods that `spark` will call to operate on the data. We don't need them in the `randomAgg`.

```scala
  override def initialize(buffer: MutableAggregationBuffer): Unit = {}

  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {}

  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {}
```

### Registering and running `randomAgg`

Now our UDAF is looking like:

```scala
import scala.util.Random

class RandomAgg extends UserDefinedAggregateFunction {
  override def inputSchema: StructType = StructType(StructField("value", IntegerType) :: Nil)

  override def bufferSchema: StructType = StructType(StructField("result", IntegerType) :: Nil)

  override def dataType: DataType = IntegerType

  override def deterministic: Boolean = false

  override def initialize(buffer: MutableAggregationBuffer): Unit = {}

  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {}

  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {}

  override def evaluate(buffer: Row): Any = {
    val r = new Random
    r.nextInt()
  }
}
```

Let's register it so that we can use it within an `expr()` an then let's apply it to a trivial Dataframe composed by my two columns `[id: bigint, value: string]`

```scala
//Register the RandomAgg as User Defined Function
spark.udf.register("RandomAgg", new RandomAgg)

//Create a sample DataFrame
val trivialDF = spark.range(20).toDF.withColumn("value", lit("michele"))

//Group by value, then aggregate using randomAgg
trivialDF.groupBy("value").agg(expr("randomAgg(id)")).show
```

Everytime you run the `trivialDF.groupBy("value").agg(expr("randomAgg(id)"))` you'll get a different value.

## Some real computation: Percentile

Given a group of observations, the *Percentile* is the percentage of observations that are lower than a given observation. For example, if in your town you have a Percentile 80% among the tall people, it means that the 80% of the population is shorter than you.

A way to calculate the percentile on separated computers could be to count on each set of data the number of observations lower than the given observation and the toal number of observations. When `spark` will merge the results we will only need to sum those two data together. The computation will be to divide the *lowerthan* observations by the *total number* of observations.

So we could think that we will work on:

* the observation for which we want to calculate the Percentile;
* the counter of *lower-than* observations
* the *total number* of observations

As done above, the first step is to create a Scala class, called `Percentile`:

```scala
import org.apache.spark.sql.Row
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{DataType, StructType}

class Percentile extends UserDefinedAggregateFunction{
  override def inputSchema: StructType = ???

  override def bufferSchema: StructType = ???

  override def dataType: DataType = ???

  override def deterministic: Boolean = ???

  override def initialize(buffer: MutableAggregationBuffer): Unit = ???

  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = ???

  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = ???

  override def evaluate(buffer: Row): Any = ???
}
```

The first edit we need is to add an input parameter to the class. This will be used when we instantiate the UDAF, before registering it.

```scala
class Percentile(observation: Double) extends UserDefinedAggregateFunction {
   ...
```

When we istantiate the class, we will pass the `observation` that we want to compare with the other data. For example how tall you are.

### `inputSchema`

As said for the `RandomAgg` function, the `inputSchema` method has to return a `StructType` that *represents data types of input arguments of this aggregate function*. In this case we do have a real input, which is the data that we want to compare to our `observation`. We assume that those data are `double`, hence:

```scala
override def inputSchema: StructType = StructType(StructField("input", DoubleType) :: Nil)
```

### `bufferSchema`

Things are getting now more interesting. I said above that in this implementation of `Percentile` we need to store

- the counter of *lower-than* observations
- the *total number* of observations

The referential `observation` is already stored in the `class` itself, and it's always the same, it doesn't change as the computation proceeds. The *lower-than* and the *total-number* do change during the computation, and it's this kind of data that need to be stored in the `bufferSchema`. The `bufferSchema` (and the `inputSchema`) are a `List` of types incapsulated in a `StructType`. In the `inputSchema` we saw that we only have one `DoubleType`. Here we have two counters. Counters are usually `Integers`, so we need to build a list with two `IntegerType` elements:

```scala
override def bufferSchema: StructType = StructType(
  StructField("lower", IntegerType) :: StructField("total", IntegerType) :: Nil)
```

Quick note: I personally find this thing of using a list incapsulated within a `StructType` far from being elegant, or at least not readable. But that's how it is, for now. End of the quick note.

### `deterministic`

It answers to the question _Given the same input will this Aggregated Function always produce the same result?_. In our case that is `True`.

```scala
  /* The Percentile Aggregated Function always produce the same result when fed with the same input */
  override def deterministic: Boolean = True
```

### `initialize`

The `initialize` method is called by `spark` just before starting to feed the `UDAF` with the data. You should take the chance to initialize the `buffer`. The `buffer` is where the `UDAF` the information you need for your computation according to the schema defined in `bufferSchema`. In this case it's some counters, and computers start counting from `0`.



```Scala
class Percentile(observation: Double) extends UserDefinedAggregateFunction {
  val LOWERCOUNTER = 0
  val TOTALCOUNTER = 1

...

  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(LOWERCOUNTER) = 0
    buffer(TOTALCOUNTER) = 0
  }

```

If you recall how the schema of `buffer` was defined:

* `buffer(0)` => `StructField("lower", IntegerType)`
* `buffer(1)` => `StructField("total", IntegerType)`

The constants `LOWERCOUNTER` and `TOTALCOUNTER` are there for the sake of readability.

### `evaluate`

Let's start with some real computation. And let's start from the end. At the end we have the total count of items (`buffer(TOTALCOUNTER)`) and the count of items lower than our `observation` (`buffer(LOWERCOUNTER)`). The calculation is trivial:

```Scala
override def evaluate(buffer: Row): Int = {
  val percentileResult: Int = 100 * buffer.getInt(LOWERCOUNTER) / buffer.getInt(TOTALCOUNTER)
  percentileResult
}
```

### `update`

What happens when we receive new data? `spark` will call the `update` method and pass the `buffer` (where we store the information for our computation) and an `input` row. The `input` schema was defined in `inputSchema`.

To calculate the Percentile we need to count how many inputs are lower than our `observation` and how many inputs in total.

```scala
override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
  buffer(TOTALCOUNTER) = buffer.getInt(TOTALCOUNTER) + 1

  if (input.getDouble(0) < observation)
    buffer(LOWERCOUNTER) = buffer.getInt(LOWERCOUNTER) + 1
}
```

At this point the `UDAF` is already working, given that you have only one worker. If you have multiple workers you have to `merge` the `buffer`s of each one.

### `merge`

The `merge` method is called by `spark` to merge the *temporary* results, that's to say the `buffer`s, of two workers. The result is stored in the first `buffer` passed. In our case we only need to sum the counters:

```scala
override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
  buffer1(LOWERCOUNTER) = buffer1.getInt(LOWERCOUNTER) + buffer2.getInt(LOWERCOUNTER)
  buffer1(TOTALCOUNTER) = buffer1.getInt(TOTALCOUNTER) + buffer2.getInt(TOTALCOUNTER)
}
```

## Registering and using the Percentile

Now our `Percentile` is looking like:

```Scala
class Percentile(observation: Double) extends UserDefinedAggregateFunction {
  val LOWERCOUNTER = 0
  val TOTALCOUNTER = 1

  override def inputSchema: StructType = StructType(StructField("input", DoubleType) :: Nil)

  override def bufferSchema: StructType = StructType(
    StructField("lower", IntegerType) :: StructField("total", IntegerType) :: Nil)

  override def dataType: DataType = IntegerType

  /* The Percentile Aggregated Function always produce the same result when fed with the same input */
  override def deterministic: Boolean = true

  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(LOWERCOUNTER) = 0
    buffer(TOTALCOUNTER) = 0
  }

  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    buffer(TOTALCOUNTER) = buffer.getInt(TOTALCOUNTER) + 1

    if (input.getDouble(0) < observation)
      buffer(LOWERCOUNTER) = buffer.getInt(LOWERCOUNTER) + 1
  }

  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(LOWERCOUNTER) = buffer1.getInt(LOWERCOUNTER) + buffer2.getInt(LOWERCOUNTER)
    buffer1(TOTALCOUNTER) = buffer1.getInt(TOTALCOUNTER) + buffer2.getInt(TOTALCOUNTER)
  }

  override def evaluate(buffer: Row): Int = {
    val percentileResult: Int = 100 * buffer.getInt(LOWERCOUNTER) / buffer.getInt(TOTALCOUNTER)
    percentileResult
  }
}
```

The way we built the `UDAF` we have to register a new function every time we want to change the reference `observation`. For example:

```scala
spark.udf.register("Percentile_10", new Percentile(10))
spark.udf.register("Percentile_30", new Percentile(30))
spark.udf.register("Percentile_50", new Percentile(50))
```

To test it, we can build a sample dataframe with numbers from 0 to 99. Applying the `Percentile_10` should return `10`, the `Percentile_30` should return `30 ` and so on.

```scala
val trivialDF = spark.range(100).toDF.withColumn("value", lit("michele"))

trivialDF.groupBy("value").agg(expr("Percentile_10(id)")).show
trivialDF.groupBy("value").agg(expr("Percentile_30(id)")).show
trivialDF.groupBy("value").agg(expr("Percentile_50(id)")).show
```

To calculate some real numbers, we could use the sample data from the book and check which percentage of sold items were less than `1.99`, `4.99`, and `9.99`:

```Scala
val df = spark.read.format("csv").option("header", "true").option("inferSchema", "true").load("../data/retail-data/by-day/*.csv")

spark.udf.register("Percentile_1_99", new Percentile(1.99))
spark.udf.register("Percentile_4_99", new Percentile(4.99))
spark.udf.register("Percentile_9_99", new Percentile(9.99))

df.groupBy("Country").agg(expr("Percentile_1_99(UnitPrice)")).show
df.groupBy("Country").agg(expr("Percentile_4_99(UnitPrice)")).show
df.groupBy("Country").agg(expr("Percentile_9_99(UnitPrice)")).show
```

## Open Questions

* Is there a way to use a template, so that I can more efficiently use this UDAF with an `Int` or any other numering type? Could one in `scala` say

  ```scala
  class Percentile[NumberT <: «Int, Double or any numeric»](observation : NumberT) 
                   							extends UserDefinedAggregateFunction {
   ...
  }
  ```

* What is the correct way of testing (e.g. Unit Test) a `UDAF`? One would test its global behaviour, or the single methods?

* Every time I access to `buffer` I do something like `buffer1.getInt(TOTALCOUNTER)`. How can I use the `scala` generics to write something like `buffer1.totalCounter` which looks so much more elegant?

* Which one is faster. Having the function defined with the observation passed at construction time (`new Percetile(10`)) or passing the observation as a column (`lit(10)`)

* What would happen if I had put `def deterministic: Boolean = false`? I did try to set it, nothing changed, I still get the right results. I suppose, and mine is only a guess, that this parameter is used by `spark` to optimise the execution.

## Links

- [User Defined Aggregate Function API](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.UserDefinedAggregateFunction)
- [Spark: The definitive guide, 1st Edition* by by Bill Chambers and Matei Zaharia](http://shop.oreilly.com/product/0636920034957.do)

