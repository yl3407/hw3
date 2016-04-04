# Homework 3

* Assigned: Monday March 14, 2016
* Due: Tuesday April 5th 8:40 AM
* Value: 3.75% of your grade

In this assignment it's time to get real!  You'll first flex your SQL
muscles and re-perform [HW2](http://github.com/w4111/hw2)'s analyses by
writing SQL and reflecting on the experience. You will then perform some normalization.


# 1. SQL, the sequel

**(16 points total)**


Instead of using PostgreSQL, we will be using the `sqlite3` database engine.
SQLite is an embedded database engine rather than a database management system (DBMS)
because it runs as a library as part of an application, rather than as a standalone
system that you can query across a network. It doesn't support advanced DBMS features
such as concurrent queries and recovery.  However, it is an exceptional database for
single user applications, it supports most of the SQL standard,
is extremely well engineered, and has some of the best documentation in an open 
source project.  [Learn more on Wikipedia](https://en.wikipedia.org/wiki/SQLite).

Note: sqlite3 does not strictly adhere to the SQL standard. It may still return results even if the SQL is wrong!

In sqlite, the database is a single file.  We have loaded the entire Iowa
dataset into a sqlite3 file, so you just need to copy it to your virtual machine.  **Note: This is a 460 MB download, and is 2.8 GB uncompressed**:

1. Download the database. Run: `wget http://www.eugenewu.net/iowa.db.gz`
2. Decompress the file: `gunzip iowa.db.gz`
3. Open the sqlite prompt by running `sqlite3` on the file: `sqlite3 iowa.db`

There are two tables `iowa` and `iowasmall` in the database. You can use `iowasmall` to test your queries, because they will run slowly on the entire `iowa` table.

Type `.help` to see help information, or [read the documentation for the command line tool](https://www.sqlite.org/cli.html) (or see the [SQLite documentation table of contents](https://www.sqlite.org/docs.html))

Now, please write the SQL query for each of the following questions. *Note*: Some queries will take a few minutes to run on your virtual machine. If your query is running for more than ~10 minutes, you either did something wrong, or you may need to create tables with your sub-results to help SQLite run the query more efficiently. For example: If you use the results of a sub-query multiple times, it can help to use `CREATE TABLE AS SELECT ...` then reference that table.

<!--1. (2 points) **Q1**: How many distinct types of items (by `item` attribute) is in this dataset?-->

<!--1. (2 points) **Q2**: How many distinct `vendor`s (by exact string comparison) are in this dataset?-->

1. (2 points) **Q1.1**: Which `store` had the most sales in terms of `bottle_qty`?

1. (2 points) **Q1.2**: At the `store` with the most sales (answer to Q1.1), what was the `description` of the most sold item? (The item that had the highest total `bottle_qty`?)

1. (3 points) **Q1.3**: For each `zipcode`, compute the single most purchased `category_name` by total `bottle_qty`. Return the top 5 (`zipcode`, `category_name`) when sorted in descending order by the most purchased total `bottle_qty`.

1. (3 points) **Q1.4**: Compute the set of all liquors with the characters "scotch" (in lowercase) in its `description`. You should use the `item` attribute to identify a specific liquor.

   Return the value of `city` for all liquor stores that sold at least one of every scotch as defined in the previous sentence.

   *Warning*: You may run into slow queries for this problem. You should be able to run it on `iowasmall`, but it is optional for you to get a solution for `iowa`. Please submit your query only.

   (Optional: It is possible to rewrite the query in a form that can execute quite fast, but it will require you to create some temporary tables and/or experiment with alternate forms to express the problem. If you find the solution, feel free to submit it!)

1. (3 points) **Q1.5**: Let a liquor's value for a given store `S_i` be the price per milliliter (mL) based on `btl_price`, averaged across all sales at `S_i`. *Note*: The `liter_size` field is really the size in mL. Ignore the quantity of bottles sold in a sale. For example: If a single store sells 10 750 mL bottles for $10 each (total cost: $100), and later 1 bottle of a 1000 mL bottle for $20 (total cost: $20), the average is (10/750 + 20/1000)/2 = 0.01666...).

   Let a liquor's _overall_ value be the average of all the per store values, averaged across all `store` in iowa.

   What is the `item` attribute of the liquor with the lowest value, as defined in the previous sentence, across all of iowa? (This is approximately the "best value" item to buy, in terms of cost per unit volume)


1. (3 points) **Q1.6**: Write a short paragraph about the main differences between writing python code 
and writing SQL.  List one or two pros and one cons for each approach. You may wish to compare the solutions from HW2 and HW3 that compute the same result.


# 2. Normalization

**(14 points total)** 


Some warmup questions:

1. (2 points) **Q2.1**: You have a relation `R(A,B,C)` and functional dependencies 
  `C->A, A->B`

  * What are all the non-trivial functional dependencies in the closure
    that have  only one attribute on the right side?
  * What are all the keys of `R`?

1. (3 points) **Q2.2**: You have a relation `S(A, B, C, D)` and functional dependencies 
  `AB->C, BC->D, CD->A, and AD->B`

  * What are all the non-trivial functional dependencies in the closure
    that have  only one attribute on the right side?
  * What are all the keys of `S`?

Now for a real application. 
The Iowa dataset has the following un-normalized schema:

    CREATE TABLE iowa (
        date date,
        convenience_store character varying(128),
        store integer,
        name character varying(128),
        address character varying(128),
        city character varying(128),
        zipcode text,
        store_location character varying(128),
        county_number integer,
        county character varying(128),
        category integer,
        category_name character varying(128),
        vendor_no integer,
        vendor character varying(128),
        item text,
        description character varying(128),
        pack integer,
        liter_size integer,
        state_btl_cost double precision,
        btl_price double precision,
        bottle_qty integer,
        total double precision
    );


You can verify this by opening the sqlite3 prompt, and typing

    .schema iowa

Suppose we have the functional dependencies:

    store → convenience_store, address, name, city, zipcode, store_location,
             county_number, county
    vendor_no → vendor
    category → category_name
    item → category, liter_size, description, state_btl_cost btl_price
    date, store, vendor_no, item → pack, bottle_qty, total


1. (2 points) **Q2.3**: What are the keys in 'iowa'?

1. (3 points) **Q2.4**: Decompose `iowa` into 3NF (Third Normal Form).  Write a few sentences to justify
  why you chose the tables you did.  

1. (2 points) **Q2.5**: Is your schema redundancy and anomaly free?  Justify your answer in
   a few sentences.

1. (2 points) **Q2.6**: We want to ensure that an order cannot purchase more than 10
   bottles (`bottle_qty`).  Can you enforce this using functional 
   dependencies?  Justify your answer

1. (1 point) **Q2.7**: Let's verify whether `store` indeed determines the store name.   How many distinct `name` values 
   exist for `store` number `2508` in the `iowa` dataset?  Solve this by running a SQL query.

1. (1 point) **Q2.8**: In class, we discussed that functional dependencies (and constraints in general) cannot be
  determined just by looking at data in the database.  
  Argue in one or two sentences whether or not `store -> name` is should actually be a functional dependency and why.  




# X.  For Giggles (Optional)

**(0 points total)**

If you are still interested in this dataset, it turns out that you _can_ normalize the store data!
The state of iowa has released a dataset of all liquor stores as a dataset available at
https://data.iowa.gov/Economy/Iowa-Liquor-Stores/ykb6-ywnd

This dataset provides us information about store attributes so we could factor those out of the iowa dataset.





# Submission

**Please submit HW3 through the following Google form**

http://goo.gl/forms/ftnp9g3t5o
