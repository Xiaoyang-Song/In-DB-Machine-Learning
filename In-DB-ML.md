
- xs2485 Xiaoyang Song
  - Wrote the example
- hl3608 Han Liu
  - Wrote the introduction

## In-DB machine learning/MADlib

### Introduction

From 4111 lectures, we learned that SQL's unique features could help us solve many problems simply. For example, extract data with the simple SELECT statement, do aggregation with functions like COUNT and AVG, solve graph problems like what we did in Project 2, and even solve recursive problems with WITH Queries, etc. One natural thing to think about is: can we use SQL to do even more things, like Machine learning? 

The answer is YES! 

As we know, data is the most important ingredient in Machine Learning (ML). Traditionally, we train our ML models outside the database, which can be complex and expensive. However, with the idea of In-DB Machine Learning, we can train ML models inside the database. In general, ML models are processed as regular tables inside the database, and we can simply interact with ML models with SQL queries. 

#### Why using SQL to do ML (Pros):
- DBMS has a very good property called Integrity Constraint (IC), which make the data clean. As Prof. Wu mentioned in the very first lecture, people spent about 80% of the time doing data cleaning. Doing ML directly in DB can mitigate this issue.
- Instead of downloading and doing ML elsewhere, doing ML in DB is more efficient.
- Can avoid moving data around.
- Exploit advantages of query optimization procedure of DBMS.


In this tutorial, we will introduce an open-source library for scalable in-database analytics and machine learning called [Apache MADlib](https://madlib.apache.org/index.html). It provides data-parallel implementations of mathematical, statistical, graph and machine learning methods for structured and unstructured data. Another trending in-database ML library/platform is [MindsDB](https://mindsdb.com/). Compared to MindsDB, MADlib has been around longer but only runs on PostgreSQL and Greenplum Database. MADlib employs its built-in functions to support in-db ML. MindsDB, by contrast, is more like an intelligence layer on top of the database. It is a cloud platform that can connect to any database and support a variety of ML frameworks.


### The Problem and Solution

- Explain the problem that it solves.
- How does the technology solve the problem?
- What are the alternatives, and what are the pros and cons of the technology compared with alternatives? (what makes it unique?)
- How it relates to concpts from 4111.
- **NOTE: illustrating the relationship with concpts from this class IS BY FAR THE MOST IMPORTANT ASPECT of this extra credit assignment**



Remember, the more detailed and thorough you contrast the technology with the topics in class the better.
We've discussed the relational model, constraints, SQL features, transactions, query execution and optimization, and recovery. Pick a subset of topics to really dive deep and compare and contrast the similarities and differences between our discussion in class and the technology.

### Tutorial

**Note: Installation is less relevant than a tutorial highlighting the main concepts as they relate to 4111.**

#### Example

Construct and describe a compelling example for the technology that is motivated by one member's project 1 application. The example and tutorial must be original.

#### Tutorial

Write a short tutorial on how to use the technology to solve the example. It may help to link to a working example (github repo or colab notebook), or a Loom tutorial video.
