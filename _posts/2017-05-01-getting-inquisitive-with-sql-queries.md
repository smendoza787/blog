---
published: true
layout: post
title: "Getting Inquisitive With SQL Queries"
author: "Sergio"
date: '2017-05-01 16:16:01 -0600'
permalink: /2017/05/01/getting-inquisitive-with-sql-queries
---

After going through a neat tutorial on [SQL](https://sqlbolt.com/), I decided to gid a little deeper into the query language to better understand it.

# [](#header-3)What is SQL?

SQL, aka Structured Query Language, is a domain-specific language designed to allow both technical and non-technical users to query, manipulate and transform data from a relational database. Or in layman's terms, its a language used to communicate with a database, since it does not understand plain ol' english.

There are many types of SQL databases, including SQLite, MySQL, PostgreSQL, Oracle, and Microsoft SQL Server. All of these databases support the SQL language standard in their own nuanced ways.

# [](#header-2)Relational Databases

It's pretty important to understand what relational databases are before understanding SQL. A relational database represents a collection of related (two-dimensional) tables.

These tables are very similar to what you would see in an Excel Spreadsheet. There are a fixed number of columns that represent the attributes of each table (Dogs table: name, age, breed; Cars table: make, model, body, color, etc.). Eact row in these tables would represent a particular Dog, or Car with their attributes listed in each column respectively.

The goal of learning SQL is to be able to communicate with these tables, and query useful information across them. Such as What is the average age of Golden Retriever owners? Or, what is the most popular color of a particular car?

# [](#header-2)SELECT Queries

In order to retrieve data from these databases, we must learn to write SQL SELECT statements, also known as queries.

A query is just a statement that declares what data we are looking for, where to find it in the database, and optionally how to transform it before it's returned.

You can think of a table in a database as an entity, and each row of that table is a specific instance of that type (ex. Dogs, students, cars, etc.). The columns would then represent the common properties shared by each instance of that entity.

The most basic query would select and display a couple columns from a table and return all rows:

```sql
SELECT column, another_column
FROM mytable;
```

This example would return our table and only display the columns we selected.

In order to return the table with all of the columns in that table, we would write something like:

```sql
SELECT * FROM mytable;
```

This would return mytable with all of the columns associated with that table. The * symbol is called the wildcard symbol and it selects ALL columns to be returned with our table.

# [](#header-2)Adding Constraints (WHERE)

We can now select ALL columns to display for our table, but what if we wanted to filter out our results to display the results of a particular condition? Thats where the WHERE clause comes in.

```sql
SELECT column, another_column, …
FROM mytable
WHERE condition
 AND/OR another_condition
 AND/OR …;
 ```

The WHERE clause applies the condition to all rows of the database and only returns rows that make the condition true. Multiple AND or OR statements can be chained onto the WHERE clause if needed.

# [](#header-2)Sorting Results (ORDER BY)

Most data returned by our database is in no particular order. That can make it difficult to read our returned data. We can use the ORDER BY clause to sort our retrieved data by a certain column attribute in ascending or descending order.

```sql
SELECT column, another_column, …
FROM mytable
WHERE condition(s)
ORDER BY column ASC/DESC;
```

When the ORDER BY clause is used, it will return all rows in alpha-numerical order based on which column you chose to order by.

You can also use the DISTINCT and LIMIT keywords to either return distinct values (one of multiple rows of the same value), or to limit the values returned by however much you specify.

# [](#header-2)JOINing Tables

In the real world, we usually look at more than one database when we are comparing data. You can join tables based on a shared column of data. (i.e. A table of Students that each have a teacher_id column, and another table of Teachers where each row of teachers has their own id).

```sql
SELECT column, another_table_column, …
FROM mytable
INNER JOIN another_table
 ON mytable.id = another_table.id
WHERE condition(s)
ORDER BY column, … ASC/DESC
LIMIT num_limit OFFSET num_offset;
```

We can use the OFFSET clause to start capturing values at a certain row instead of the default "first" row.

Hopefully this basic overview of SQL queries helps you get started on your SQL learning journey. For more info and a great resource on SQL, visit SQLBolt.com.
