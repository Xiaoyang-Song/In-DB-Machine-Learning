- xs2485 Xiaoyang Song
  - Researched on both MADlib and MindsDB.
  - Wrote the introduction and problems setups.
- hl3608 Han Liu
  - Researched on both MADlib and MindsDB.
  - Wrote the examples and tutorials.

## In-DB machine learning - MindsDB

### Introduction

From 4111 lectures, we learned that SQL's unique features could help us solve many problems simply. For example, extract data with the simple _SELECT_ statement and _WHERE_ clause, do simple math using aggregation functions like _COUNT()_ and _AVG()_, solve both elementary and complex graph problems like what we did in Project 2, and even solve complicated problems like Fibonacci sequences using recursive WITH Queries, etc. Witnessing how powerful SQL and DBMS can do, one natural thing to think about is: can we use SQL to do even more advanced tasks, like Machine Learning?

The answer is YES! And we will introduce why that is important.

#### Problem addressed: Doing Machine Learning in Database

#### Reasons & Motivations?

I believed that most people who are not familiar with database systems have never imagined that we can do Machine Learning in the database. However, its unpopularness does not mean it is not useful. In fact, doing ML in database is extremely useful and it is an important skill to acquire. There are 3 major reasons about why we should do that:

1. As we know, data is the most important ingredient in Machine Learning (ML). Traditionally, we train our ML models outside the database, which requires moving data from one place to another place, which is expensive, unreliable in terms of safety, and also inconvenient. However, with the idea of In-DB Machine Learning, we can train ML models inside the database. In general, ML models are processed as regular tables inside the database, and we can simply interact with ML models with SQL queries.

2. Secondly, DBMS has a very good property called Integrity Constraints (IC), which must be enforced upon insertion of the data tuples and guarantee that all the data in the database are clean and consistent under the same set of rules (i.e. constraints). This significantly boost the productivity of machine learning programmers and researchers by reducing the time they needed to do data cleaning. As Prof. Wu mentioned in the very first lecture, people spent about 80% of the time doing data cleaning.

3. When doing research for writing this tutorial, we found that many people argued that doing ML in database is not reliability because the DBMS may crash at some time and all records will be lost. However, this is not the case, in the lecture, we covered that there are tons of techniques in DBMS that can be utilized to enforce ACID principle such as Concurrecy Control and Recovery. With Recovery technique like Write Ahead Logging, each transaction will be recoverable if they have been committed and the trained model will not be lost when DBMS crashed. Furthermore, doing ML in our local machine is more vulnerable to crashing issue. If you are training model using Jupyter notebook and at some time your computer crash, then your Jupyter session will automatically stop and all the model will not be recoverable. Remembered that **Jupyter kernel does not have effective mechanisms of dealing with crashing like Recovery in DBMS**. So DBMS is actually more stable and reliable.

In this tutorial, we will introduce [MindsDB](https://mindsdb.com/), an open-source AI layer for existing databases that allows users to effortlessly develop, train and deploy state-of-the-art machine learning models using SQL queries. There are many other platforms or extensions that are related to doing machine learning in database. One popular approach is to use [MADlib](https://madlib.apache.org/). We also thoroughly researched on it but we found that there are five apparent advantages of using **MindsDB** over **MADlib**:

1. MADlib only supports Postgresql with certain versions: for example, Postgresql 14.5 and the latest version of 15.0 is not supported by MADlib. Therefore, it is not compatible with the latest release and can not exploit the advantages of those new SQL features. (In addition, changing versions is a great pain for most programmers)
2. MADlib is harder to install and use in general. The installation has many prerequisites (i.e. installation of other packages are also required). However, for MindsDB, there is no need of installation at all! It is an online platform.
3. In MindsDB, it is easy to visualize the underlying training process and models, whereas in MADlib, it is not easy to achieve.
4. Compared to MindsDB, MADlib has been around longer but only runs on PostgreSQL and Greenplum Database. MADlib employs its built-in functions to support in-db ML. MindsDB, by contrast, is more like an intelligence layer on top of the database. It is a cloud platform that can connect to any database and support a variety of ML frameworks.
5. MindsDB is new compared to MADlib, why don't we stick with new technology if the older one is not better!

In the following tutorial, we will formulate a problem derived from our project 1 and show how we can solve that by performing machine learning in database.

## Tutorial & Example Walkthrough

### Example Problems

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
(19, 16, 5, 3, 4, 3);
```

We can use the following the visualize the table:

```sql
%%sql SELECT * FROM DEMO
```

The table looks like the following:

<img width="300" alt="Screen Shot 2022-12-09 at 10 27 13 PM" src="https://user-images.githubusercontent.com/82349855/206826944-260550d8-6bb3-4b7c-916c-d5c5c4aa068e.png">

### MindsDB Tutorials

Now, with our demo data, we can proceed to do Machine Learning using **MindsDB**.

#### Get started with MindsDB

To get started, just [create a free MindsDB Cloud account](https://cloud.mindsdb.com/signup) and login to use its cloud editor.

#### Connecting to the database

Now, we need to connect to the database. For illustration purpose, we connect to the project1 part2 database.

```sql
CREATE DATABASE projdb
WITH ENGINE = "postgres",
PARAMETERS = {
    "user": <Your-UNI>,
    "password": <Your-Passcode>,
    "host": "w4111.cisxo09blonu.us-east-1.rds.amazonaws.com",
    "database": "proj1part2"
};
```

If you want to connect to your local database or other database, just modify the corresponding fields in the above query. Then, simply run the query in the MindsDB console and you should be able to connect to your database in the cloud platform like the following:

<img width="500" alt="image" src="https://user-images.githubusercontent.com/30332629/206828479-5499d890-13af-4783-8d98-ac54cb3b49b5.png">

#### Create and train a ML model

With all of the demo data, let's start training!

```sql
CREATE MODEL
  mindsdb.restaurants_ratings_model
FROM projdb
  (SELECT rating, num_likes,num_hates, num_violations, num_comments FROM demo)
PREDICT rating;
```

Here are what we did using the above code:

- CREATE a model called _restaurants_ratings_model_.
- SELECT columns to be used for training and validation.
- Set the target of prediction to be _rating_.

```sql
SELECT *
FROM mindsdb.models
WHERE name='restaurants_ratings_model';
```

Then, simply run the above query to check on current model status and configurations. Note that _mindsdb.models_ is the table that contains all the models that we have created. Here is what it should look like:

<img width="1817" alt="image" src="https://user-images.githubusercontent.com/30332629/206827762-160ffe30-55d2-4a6d-99be-b4ebce7eb4a9.png">

As you can see the current status is **training**, implying that the model is being trained now. After the column STATUS becomes **complete**, we are done with training and can use that to make predictions.

#### Make Predictions

Let's make some predictions using our model. For example, we want to predict the ratings of a restaurant with features (num_likes = 2, num_hates=8, num_violations=5, num_comments=1):

```sql
SELECT rating
FROM mindsdb.restaurant_ratings_model
WHERE num_likes = 2
AND num_hates=8
AND num_violations=5
AND num_comments=1;
```

Simply run the above query and the result is the following:

<img width="375" alt="image" src="https://user-images.githubusercontent.com/30332629/206827944-126d44ad-e188-4c1e-8978-8ab43648ca26.png">

Another example with features (num_likes = 20, num_hates=0, num_violations=2, num_comments=1):

```sql
SELECT rating
FROM mindsdb.restaurant_ratings_model
WHERE num_likes = 20
AND num_hates= 0
AND num_violations=2
AND num_comments=1;
```

The result is given by the following:

<img width="375" alt="image" src="https://user-images.githubusercontent.com/30332629/206828047-c15c6961-6a92-42d2-b5f4-a4ee232e4c18.png">

#### Model Analysis

By default, MindsDB will train many possible ML models for the task you specified at the same time. To check which ML model MindsDB used on our dataset, simply execute the following query:

```sql
DESCRIBE MODEL mindsdb.restaurant_ratings_model.model;
```

<img width="726" alt="image" src="https://user-images.githubusercontent.com/30332629/206828089-2bf1a403-392d-42d4-ab4f-0767f6475846.png">

The table above exhibits all the ML models that MindsDB has tried and indicates that it selected RandomForest for our regression task.

### Conclusion

In general, doing Machine Learning in Database is not hard with MindsDB and its advantages are obvious as we introduced in the first few sections of this tutorial. Let's keep the data in the Database and do ML in database in the future!
