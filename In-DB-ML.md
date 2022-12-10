- xs2485 Xiaoyang Song
  - Wrote the example
- hl3608 Han Liu
  - Wrote the introduction

## In-DB machine learning - MindsDB

### Introduction

From 4111 lectures, we learned that SQL's unique features could help us solve many problems simply. For example, extract data with the simple _SELECT_ statement and _WHERE_ clause, do simple math using aggregation functions like _COUNT()_ and _AVG()_, solve both elementary and complex graph problems like what we did in Project 2, and even solve complicated problems like Fibonacci sequences using recursive WITH Queries, etc. Witnessing how powerful SQL and DBMS can do, one natural thing to think about is: can we use SQL to do even more advanced tasks, like Machine Learning?

The answer is YES!

As we know, data is the most important ingredient in Machine Learning (ML). Traditionally, we train our ML models outside the database, which can be complex and expensive. However, with the idea of In-DB Machine Learning, we can train ML models inside the database. In general, ML models are processed as regular tables inside the database, and we can simply interact with ML models with SQL queries.

#### Why using SQL to do ML (Pros):

- DBMS has a very good property called Integrity Constraint (IC), which make the data clean. As Prof. Wu mentioned in the very first lecture, people spent about 80% of the time doing data cleaning. Doing ML directly in DB can mitigate this issue.
- Instead of downloading and doing ML elsewhere, doing ML in DB is more efficient.
- Can avoid moving data around.
- Exploit advantages of query optimization procedure of DBMS.

In this tutorial, we will introduce [MindsDB](https://mindsdb.com/), an open-source AI layer for existing databases that allows users to effortlessly develop, train and deploy state-of-the-art machine learning models using SQL queries. Another popular in-database ML library is [Apache MADlib](https://madlib.apache.org/index.html). Compared to MindsDB, MADlib has been around longer but only runs on PostgreSQL and Greenplum Database. MADlib employs its built-in functions to support in-db ML. MindsDB, by contrast, is more like an intelligence layer on top of the database. It is a cloud platform that can connect to any database and support a variety of ML frameworks.

### The Problem and Solution

- Explain the problem that it solves.
- How does the technology solve the problem?
- What are the alternatives, and what are the pros and cons of the technology compared with alternatives? (what makes it unique?)
- How it relates to concpts from 4111.
- **NOTE: illustrating the relationship with concpts from this class IS BY FAR THE MOST IMPORTANT ASPECT of this extra credit assignment**

Remember, the more detailed and thorough you contrast the technology with the topics in class the better.
We've discussed the relational model, constraints, SQL features, transactions, query execution and optimization, and recovery. Pick a subset of topics to really dive deep and compare and contrast the similarities and differences between our discussion in class and the technology.

## Tutorial

### Example

In our project 1, we made a Web App about the restaurant ratings and violation records in New York City, where we displayed restaurants locations, ratings, and violation records, and enabled our users to interactively browse those records. In addition, in our Web App, users are also allowed to like/hate and post comments to the restaurants. When we are doing project 1, we originally want to do a recommendation system, which can automatically recommend restaurant to users based on the quality of the restaurants. However, we haven't done that due to time limit.

The basic logic is that we want to recommend restaurants to users based on the overall quality of the restaurants, which are dictated by the attributes **ratings** in the table **Grade** of our project 1. However, those grades are static and will be updated once a month when new inspections are done by government staffs. Therefore, we want to utilize Machine Learning, specifically Linear Regression, to automatically predict the latest **ratings** for all the restaurants in the New York city based on 4 statistics:

- the total number of likes they received (**num_likes**)
- the total number of hates they received (**num_hates**)
- the total number of violation records they had (**num_violations**)
- the total number of comments they received (**num_comments**)

And using all those four statistics, we want to predict the **rating** of that restaurant. Now we will walk you through how we solve this Machine Learning problem in the Database using an online platform [MindsDB](https://mindsdb.com/).

**Note**: our original database in project 1 contains about 5000 records and using all of its data is not a good choice when doing a tutorial; at the same time, it requires many table joins to actually get the four statistics for each restaurants, which is not the point of this tutorial. Therefore, we choose to make a **demo** table and some sample data by ourselves (all data are modified from our original project 1 database) and using them to make the tutorial more understandable.

#### Demo Table & Sample Data

We created a **demo** table consisting of restaurant id (**rid**) as key, restaurant rating (**rating**) as the label (i.e. y value) for regression problem, and all of the four statistics that we mentioned in the previous section (i.e. x values for regression). Here is how it looks like:
(**Note**: we need to connect to our database to do the table creation, which just followed what we did for Project 1)

```sql
%%sql
DROP TABLE IF EXISTS DEMO;
CREATE TABLE IF NOT EXISTS DEMO(
    r_id int PRIMARY KEY,
     rating float,
     num_likes int,
     num_hates int,
     num_violations int,
     num_comments int
);

INSERT INTO demo VALUES
(0, 20, 8, 2, 4, 3 ),
(1, 21, 9, 4, 2, 6),
(2, 35, 2, 1, 0, 2),
(3, 25.5, 2, 8, 1, 2),
(4, 3, 0, 6, 3, 7),
(5, 2, 1, 9, 2, 1),
(6, 17, 2, 3, 4, 3),
(7, 15.5, 2, 0, 3, 6),
(8, 12.5, 5, 8, 4, 3),
(9, 0, 0, 10, 7, 4),
(10, 23, 6, 3, 3, 8),
(11, 12, 4, 2, 6, 3),
(12, 18, 6, 2, 3, 1),
(13, 14, 3, 3, 3, 3),
(14, 15, 4, 1, 4, 4),
(15, 35, 2, 1, 0, 2),
(16, 25.5, 2, 8, 1, 2),
(17, 3, 0, 6, 4, 7),
(18, 21, 11, 3, 2, 1),
(19, 16, 5, 3, 4, 3)
```

### Tutorial

Write a short tutorial on how to use the technology to solve the example. It may help to link to a working example (github repo or colab notebook), or a Loom tutorial video.
