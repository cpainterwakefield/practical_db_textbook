.. _grouping-chapter:

========================
Grouping and aggregation
========================

Previous chapters have focused on mechanisms for storing and retrieving data.  SQL also provides facilities for doing simple analyses of the data.  In this chapter, we discuss methods of partitioning data and computing simple statistics on the partitions.


Tables used in this chapter
:::::::::::::::::::::::::::

For this chapter, we will primarily work with the tables **bookstore_inventory** and **bookstore_sales**, which simulate a simple database a seller of used books might use.  The **bookstore_inventory** table lists printed books that the bookstore either has in stock, or has sold recently, along with the condition of the book and the asking price.  When the bookstore sells a book, a record is added to the **bookstore_sales** table.  This table lists the **stock_number** of the book sold, the date sold, and the type of payment.  The column **stock_number** is a unique identifier for each book, and can be used to join the tables.

We will also use the **authors** table to illustrate the interaction of ``NULL`` with aggregate functions.

A full description of these tables can be found in :ref:`Appendix A <appendix-a>`.


Aggregate statistics
::::::::::::::::::::

*Aggregate statistics* are values computed on entire sets of data.  Counts, sums, and averages are examples of aggregate statistics.  SQL provides a number of aggregate functions for computing such statistics.  This section will cover some of the most commonly used functions; for documentation on all of the aggregate functions defined by SQL, see see Appendix B, :ref:`appendix-b-aggregate-functions`.

We start by using the **COUNT** aggregate function to illustrate the basic rules for aggregate functions.  At its simplest, **COUNT** can be used to return the number of rows in a table:

.. activecode:: grouping_example_aggregate
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT COUNT(*) FROM bookstore_sales;

If you add a **WHERE** clause, **COUNT** will only consider rows matching the **WHERE** clause:

::

    SELECT COUNT(*)
    FROM bookstore_sales
    WHERE payment = 'cash';

The use of **\*** within the aggregate function is unique to **COUNT**; it simply means that we want to count rows, not any particular column.  Other aggregate functions require application to a column or expression.  When applied to a column or expression, **COUNT** and the other aggregate functions ignore ``NULL`` values.  For example, observe the result when applying **COUNT** to different columns of the **authors** table:

::

    SELECT COUNT(*), COUNT(author_id), COUNT(birth), COUNT(death)
    FROM authors;

In each of the results above, we obtain a single number for each function in the **SELECT** clause.  Only one row is returned, because we are (at the moment) applying the aggregate function to all rows matching the (optional) **WHERE** clause.  This row represents a *summary* of the data.  As a summary, details of the data cannot be included; it is an error to try to retrieve any expressions on the columns other than aggregates, when any aggregate function is used.  The following query will result in an error in every database except SQLite:

::

    SELECT title, COUNT(*) FROM bookstore_inventory;

While SQLite does not give us an error, the data returned demonstrates why this is an error in most databases; the returned **title** value represents just one row of the table, while the **COUNT(\*)** value is a summary of the whole table.  These two things do not match.

In addition to **COUNT**, the most common aggregate functions implemented by SQLite are **SUM**, **AVG**, **MIN**, and **MAX**.  SQL defines a number of other aggregate functions, including statistical functions such as variance and standard deviation.  As with **COUNT**, the argument of the function is a column or expression, which is evaluated for every row; ``NULL`` values are discarded.  For all aggregates except **COUNT**, if the argument evaluates to ``NULL`` for every row, then the result is ``NULL`` (for **COUNT** the result is zero).

You can probably guess the meaning of these aggregate functions:

- **SUM** computes the sum, can only be applied to numbers
- **AVG** computes the average, can only be applied to numbers
- **MIN** finds the minimum value according to the normal sort order for the value type
- **MAX** finds the maximum value according to the normal sort order for the value type

We can apply all of these to the **price** column of **bookstore_inventory**:

::

    SELECT COUNT(price), SUM(price), AVG(price), MIN(price), MAX(price)
    FROM bookstore_inventory;

In addition to ignoring ``NULL`` values, it is possible to indicate that duplicate values should be ignored, by putting the **DISTINCT** keyword before the argument of the aggregate function.  This is mostly useful when used with **COUNT**.  For example, if we want to know the total number of books versus the number of unique titles in our inventory, we could do:

::

    SELECT COUNT(title), COUNT(DISTINCT title) FROM bookstore_inventory;


Grouping
::::::::

Aggregates can be very useful when applied to an entire table or to a set of rows matching a **WHERE** clause.  Sometimes, though, we want more than one count, or sum, or average from a table; we may want these statistics over some subsets of rows, organized by some common attribute.  For example, our **bookstore_inventory** table includes books in different conditions; we might be interested in the average price we are charging for books in "good" condition separately from books in "fair" condition, and so forth.

SQL provides the **GROUP BY** clause for this purpose.  **GROUP BY** lets us specify an attribute, such as the condition of a book, by which to organize a table into *groups*.  Membership in a specific group is based on the **GROUP BY** expression; all members of a group share the same value for the expression.  The groups form a *partition* of the data; every row (matching the optional **WHERE** clause) is assigned to a group, and no row is assigned to more than one group.

With **GROUP BY** in effect, we can now retrieve information about each group as a whole; each row of our output will represent information about one group.  If we put an aggregate function expression in our **SELECT** clause, the aggregate is applied to each group's rows separately.  In addition to aggregates, we can **SELECT** the **GROUP BY** expression - this is allowed (and makes sense) because all of the rows in each group will have the same value for the expression. You usually want to include the grouping expression as a label for the group - otherwise you will not know what group each aggregate expression belongs to!

The **GROUP BY** clause comes immediately after the **WHERE** clause, or after **FROM** if there is no **WHERE** clause.  Here is an example of grouping on our bookstore inventory by book condition:

.. activecode:: grouping_example_grouping
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT condition, COUNT(*), AVG(price)
    FROM bookstore_inventory
    GROUP BY condition;

If we want to exclude books that have already been sold, we could add a **WHERE** clause (here we use a subquery - subqueries are discussed in :numref:`Chapter {number} <subqueries-chapter>`):

::

    SELECT condition, COUNT(*), AVG(price)
    FROM bookstore_inventory
    WHERE stock_number NOT IN
      (SELECT stock_number FROM bookstore_sales)
    GROUP BY condition;

It is also possible to group by more than one expression, in which case each group is defined by a unique setting for all of the expressions.  Our **bookstore_sales** table contains information about the date in which a book was sold, as well as the type of payment used in the purchase.  We might be very interested in knowing sales totals by month, or by type of payment, or both.  To get the price paid, we will have to join in the **bookstore_inventory** table.

To start with, let's retrieve sales totals by month (here we will use SQLite's substring() function to extract the 2-digit month number; in other databases it may be possible to extract a month by name):

::

    SELECT
      substring(s.date_sold, 6, 2) AS month,
      SUM(i.price) AS total_sales
    FROM
      bookstore_sales AS s
      JOIN bookstore_inventory AS i ON s.stock_number = i.stock_number
    GROUP BY month;

Note that we can use the alias defined in our **SELECT** clause in our **GROUP BY** clause without having to rewrite the function expression.

Now, let's break down our total sales by type of month *and* type of payment:

::

    SELECT
      substring(s.date_sold, 6, 2) AS month,
      s.payment,
      SUM(i.price) AS total_sales
    FROM
      bookstore_sales AS s
      JOIN bookstore_inventory AS i ON s.stock_number = i.stock_number
    GROUP BY month, s.payment
    ORDER BY month, s.payment;

Here we have sorted by our grouping expressions as well, just to ensure that our groups come out in a consistent fashion.

Filtering grouped data
----------------------

When we group, we are generating a new set of rows representing the groups present in our data.  If we include a **WHERE** clause in our query, it is applied to the data *before* grouping.  The **WHERE** clause, then, cannot be used to filter the set of rows produced by grouping.  If we want to filter the grouped data, we must do so using a **HAVING** clause.

The **HAVING** clause works just like the **WHERE** clause, but applies to the set of rows generated by grouping.  Using **HAVING**, we can filter by expressions available to us after grouping: any expressions that we grouped by (our group labels), or aggregate functions on the groups.  The **HAVING** clause comes after the **GROUP BY** clause.

Here we use **HAVING** to list books for which we have more than one copy in our bookstore inventory, in order by the number of copies:

::

    SELECT author, title, COUNT(*)
    FROM bookstore_inventory
    GROUP BY author, title
    HAVING COUNT(*) > 1
    ORDER BY COUNT(*) DESC;

We can, of course, use both **WHERE** and **HAVING** in the same query - here we group books that have not been sold by author and title; then we report  the titles (groups) with multiple copies:

::

    SELECT author, title, COUNT(*)
    FROM bookstore_inventory
    WHERE stock_number NOT IN
      (SELECT stock_number FROM bookstore_sales)
    GROUP BY author, title
    HAVING COUNT(*) > 1;


Self-check exercises
::::::::::::::::::::

This section contains exercises on grouping and aggregation, using the **bookstore_inventory** and **bookstore_sales** tables.  If you get stuck, click on the "Show answer" button below the exercise to see a correct answer.

.. activecode:: grouping_self_test_count
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to count the number of books in our inventory by the author Toni Morrison:
    ~~~~

.. reveal:: grouping_self_test_count_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT COUNT(*) FROM bookstore_inventory WHERE author = 'Toni Morrison';


.. activecode:: grouping_self_test_statistics
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find the minimum, maximum, and average price of a book in "good" condition:
    ~~~~

.. reveal:: grouping_self_test_statistics_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT MIN(price), MAX(price), AVG(price)
        FROM bookstore_inventory
        WHERE condition = 'good';


.. activecode:: grouping_self_test_distinct
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find out how many different authors we have books by:
    ~~~~

.. reveal:: grouping_self_test_distinct_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT COUNT(DISTINCT author) FROM bookstore_inventory;


.. activecode:: grouping_self_test_grouping_1
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to get the average price of a book, by author; sort by highest average price first:
    ~~~~

.. reveal:: grouping_self_test_grouping_1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT author, AVG(price)
        FROM bookstore_inventory
        GROUP BY author
        ORDER BY AVG(price) DESC;


.. activecode:: grouping_self_test_grouping_2
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to get the average price of a book, by author and condition; sort by author and condition:
    ~~~~

.. reveal:: grouping_self_test_grouping_2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT author, condition, AVG(price)
        FROM bookstore_inventory
        GROUP BY author, condition
        ORDER BY author, condition;


.. activecode:: grouping_self_test_grouping_3
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to give the number of books sold, and the total sales from those books, by condition.  Exclude books for the payment type "trade in".
    ~~~~

.. reveal:: grouping_self_test_grouping_3_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT i.condition, COUNT(*) AS books_sold, SUM(i.price) AS sales
        FROM
          bookstore_inventory AS i
          JOIN bookstore_sales AS s ON s.stock_number = i.stock_number
        WHERE s.payment <> 'trade in'
        GROUP BY i.condition;


.. activecode:: grouping_self_test_having
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find which authors' books have an average price less than 3.
    ~~~~

.. reveal:: grouping_self_test_having_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT author, AVG(price)
        FROM bookstore_inventory
        GROUP BY author
        HAVING AVG(price) < 3;


.. activecode:: grouping_self_test_challenge_1
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to get the difference between the maximum and minimum price of a book for each possible book condition:
    ~~~~

.. reveal:: grouping_self_test_challenge_1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT condition, MAX(price) - MIN(price)
        FROM bookstore_inventory
        GROUP BY condition;


.. activecode:: grouping_self_test_challenge_2
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find the maximum price of any book in our inventory, and list the books at that price.  *Hint*: you will need to use a subquery for this one.
    ~~~~

.. reveal:: grouping_self_test_challenge_2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM bookstore_inventory WHERE price =
          (SELECT MAX(price) FROM bookstore_inventory);




|chapter-end|


|license-notice|
