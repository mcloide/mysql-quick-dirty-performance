# MySQL Quick and Dirty Performance Tips

These are quick and easy tips to gain performance with your **selects** on large or small datasets while using, MySQL mostly, not that it can't be used with other SQL databases.

## Asterisk (or the risk of the star)

As simple as it can be said:

* No
* Just no
* Don't do it
* Don't even think about it
* Stop arguing and don't do it

Consider that your table has 10 columns. Every time that you use `*`, MySQL will have to fetch which of those columns are for that table and translate the * to them. It is not only bad for performance but it also kills readability.

## Fine tune your functions / procedures

Functions and stored procedures are great for code-reuse and to keep business logic encapsulated. They are also great to keep pitfalls on query performance.

Remember, every **function** or **procedure** call will have normally an overhead and if these functions / procedures are ineficient, they will slow down the end result.

For example:

```
DELIMITER;;
CREATE FUNCTION calculate_total_amount(qty int, amount decimal(10,4)) returns decimal(10,4)
DETERMINISTIC
BEGIN

  DECLARE total decimal(10,4) DEFAULT 0;
  SELECT (qty * amount) INTO total;
  
  IF (ISNULL(total)) THEN
    SET total = 0;
  END IF;
  
  RETURN total;
END;;
DELIMITER;
```

It might not be apparent there but there is a great performance pitfall here. This function does a simple `qty * amount` and returns but it also declares a variable and checks if it is null. This makes it a lot slower than it could be.

A simple peformance improvement for this function would be:

```
DELIMITER;;
CREATE FUNCTION calculate_total_amount(qty int, amount decimal(10,4)) returns decimal(10,4)
DETERMINISTIC
BEGIN
  RETURN IFNULL((qty * amount),0);
END;;
DELIMITER;
```

## Read your slow query log

How you configure your slow query log will really depend on how long do you consider a query to perform bad. In general, if your query is taking over 1 second to execute, that is an indicator that it requires some performance tunning.

You can read the documentation on how to enable it here - http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html - but, for **development** your ini changes should look similar to this:

```
# MySQL Slow Query Log
slow_query_log = 1
slow_query_log_file = /var/log/slow_query.log
long_query_time = 1
```

When the slow query log prints the offender queries it will also print something like this:

```
# Time: 2016-11-17T19:03:25.889681Z
# User@Host: root @ localhost []  Id:   917
# Query_time: 2.298696  Lock_time: 0.000119 Rows_sent: 1  Rows_examined: 1847705
```

Whenever the gap between the amount of **Rows_sent** and **Rows_examined** is big that is a direct indicator that this query requires a revision.

## Use Explain

There is a lot that can be explained by how to use Explain (http://dev.mysql.com/doc/refman/5.7/en/explain.html) but whenever you are using explain these are a few things you need to keep in mind:

* Gap between rows and filter is really big
* Type is ALL (and some times ranged is also bad)
* Extra shows using temporary
* Possible keys doesn't have neither a primary, foreign or index

## Simple strategies

These are some simple strategies that can help identify performance issues on a query:

* Lack of filters
* Use of a sub-select / temporary whenever it could be done in a function or join
* No indexes
* Ineficient or wrong groupping - place MySQL with the sql mode to ONLY_FULL_GROUP_BY and you will see interesting results - http://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sql-mode-changes
* A long range brings bad performance, but the same range splitted in smaller ranges generates worst performance.

Ok this last one requires an explanation. Assume that your query is bringing data for a whole year and it takes 2.1 seconds. If you fetch the same query 12 times, how long it would take ? If the answer is higher than 2.1 seconds that points out to some performance pitfall.

## Different queries has different approaches

When dealing with a really complex query that uses sub-selects, joins, etc, make sure that you can filter it as much as it can and, if nothing better can be done, approach it differently. Not every query will benefit from the same strategy to increase performance.

For example, a query that consumes considerable amount of resources from the server might need to be approached by being split, more filters or even a mix between query and code (php, java, node, etc).

## Structure

Keep in mind that a poor table structure will directly affect your query performance.

## SQL Coding

Coding SQL is no different from coding so, if your query *smells* it will perform bad in the same way that if a code *smells* it will perform bad or bring unexpected results.

## Consider caching / ETL

If you exausted what can be done with MySQL for performance, then, consider caching or ETL databases.
