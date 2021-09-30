.. _subqueries-chapter:

==========
Subqueries
==========

A subquery is simply a **SELECT** query enclosed with parentheses and nested within another query or statement.  This *subquery expression* evaluates to a scalar, a row, a column, or a table, depending on the query results and the context in which the subquery appears.  In this chapter we discuss the many ways in which we can use the result of a query within another query.


Tables used in this chapter
:::::::::::::::::::::::::::

For this chapter we will be using the books dataset (tables **books**, **authors**, etc.), a description of which can be found in :ref:`Appendix A <appendix-a>`.


Scalars, rows, and tables
:::::::::::::::::::::::::

Before we discuss subqueries, it is useful to talk about some additional types of data that come up in SQL.  So far we have assumed that all expressions evaluate to a *scalar* value.  Scalar values are simple values of a single type, such as ``42`` or ``'hello'``.  We can also work with *row* values in SQL.  A row is just an ordered list of values.  We can write down a literal row (SQL calls these "row value constructors") by putting a comma-separated list of expressions between parentheses:

::

    (1, 'hello', 3.1415)

In most databases (but not SQLite), you can **SELECT** the above expression as a literal, with the result showing as a single column.

Rows can be used in comparison expressions: the two queries below are equivalent (although the first is probably preferred as a matter of style):

.. activecode:: subqueries_example_row_expressions
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT b.title
    FROM
      books AS b
      JOIN authors AS a ON a.author_id = b.author_id
    WHERE
      (a.name, b.publication_year) = ('V. S. Naipaul', 1961);

    SELECT b.title
    FROM
      books AS b
      JOIN authors AS a ON a.author_id = b.author_id
    WHERE a.name = 'V. S. Naipaul'
    AND b.publication_year = 1961;


We will see more useful applications of row expressions in the following sections.

Beyond rows, we can also think in terms of *tables* as values.  Here we are using tables to mean a collection of rows, not necessarily the named object living in the database.  The result from a **SELECT** query is a table in this sense.


Boolean subquery expressions
::::::::::::::::::::::::::::

To start with, we examine Boolean expressions using subqueries.  These are appropriate for use within the **WHERE** clause of another query or statement.


Scalar or row result
--------------------

We can use a subquery in place of a scalar expression as long as we know the subquery will return a single row and column.  For an example, let's return to an example from :numref:`Chapter {number} <joins-chapter>` using our **books** table: how can we find all books published in the same year as *The Three-Body Problem*?  We will use a subquery to first obtain the **publication_year**, and then use that result to get our list of books:

.. activecode:: subqueries_example_single_row
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT * FROM books WHERE publication_year =
      (SELECT publication_year
       FROM books
       WHERE title = 'The Three-Body Problem')
    ;

To see what is happening in this query, first execute the subquery on its own:

::

    SELECT publication_year
    FROM books
    WHERE title = 'The Three-Body Problem';

Since this particular query returns exactly one row and one column, we treat the result as a scalar value, and simply substitute the scalar in place of the subquery:

::

    SELECT * FROM books WHERE publication_year = 2008;

Note that this only works because we only have one book with the title *The Three-Body Problem*.  In general it is not a good idea to assume a query will return a single row unless you use a **WHERE** clause condition on a column known to hold unique values, or unless you are computing an aggregate statistic over a set of rows (we will discuss aggregates in :numref:`Chapter {number} <grouping-chapter>`).  If multiple rows are returned by the subquery, the query will result in an error.  However, if zero rows are returned from the subquery, the result is considered to be ``NULL``, rather than an error.

This same approach works with row expressions, although the syntax is perhaps a bit inconsistent.  If a subquery would return multiple columns, then you need to use a row expression on the left-hand side of your comparison only.  That is, the below is correct SQL:

::

    SELECT * FROM books
    WHERE (author_id, publication_year) =
      (SELECT author_id, publication_year
       FROM books
       WHERE title = 'The Hundred Thousand Kingdoms')
    ;

Putting parentheses around the columns in the **SELECT** clause of the subquery will cause an error.

Comparisons do not have to be equality; you can use any comparison operator:

::

    SELECT * FROM books
    WHERE publication_year >
      (SELECT publication_year FROM books
       WHERE title = 'Americanah')
    ;


Table or column result
----------------------

When a query can return multiple rows (a column or table), we have a different set of operators to work with.  In this section we discuss the **IN** operator and the use of comparison operators with **ALL**, **ANY**, and **SOME**.  Another Boolean operator, **EXISTS**, will wait until we discuss correlated subqueries later in the chapter.  All of the operators that work with multiple rows work also work on subqueries returning zero rows or one row.

IN
####

The **IN** operator lets us compare some expression to every row returned from a subquery.  If the expression equals any result from the subquery, then the **IN** expression evaluates to ``True``.  For example, we can ask our database for a list of books which have won awards - books with book ids matching some book id in the **books_awards** table:

.. activecode:: subqueries_example_multiple_rows
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT * FROM books WHERE book_id IN
      (SELECT book_id FROM books_awards)
    ;

SQL also provides the **NOT IN** operator as simply the Boolean inverse of **IN**.  We can get a list of books that did not win any of the awards listed in our database by a simple modification of the above query:

::

    SELECT * FROM books WHERE book_id NOT IN
      (SELECT book_id FROM books_awards)
    ;

The **IN** operator also works with row expressions, when we want to compare against multiple column subquery results.  Here is a query that asks for books published in the same year as the author's death.  (We are using the **substring** function as implemented by SQLite to get just the first four characters of each author's death date.  Although the substring is a character string and book publication years are stored as integers, SQLite is able to do an appropriate type conversion to make the comparison.)

::

    SELECT a.name AS author, b.title, b.publication_year
    FROM
      authors AS a
      JOIN books AS b ON a.author_id = b.author_id
    WHERE
      (a.author_id, b.publication_year) IN
        (SELECT author_id, substring(death, 1, 4) FROM authors)
    ;

As always, it can be helpful to execute the subquery separately to see what values it returns in order to better understand what the entire query is doing.

**IN** also has a useful application not involving a subquery.  If we follow **IN** with a comma-separated list of expressions inside parentheses, the operator will test the expression to the left of **IN** against every expression listed in the parentheses. Note that, while the expression list looks like a row expression, it is very different; every expression in the list after **IN** should have a type compatible with the expression being compared.

For example, we might be interested in books by a few different authors:

::

    SELECT a.name AS author, b.title
    FROM
      books AS b
      JOIN authors AS a ON a.author_id = b.author_id
    WHERE author IN
      ('Virginia Woolf', 'Kazuo Ishiguro', 'Iris Murdoch');


If we want to compare multiple values (i.e., row expressions), we must use parentheses for each expression.  In this case, the general form of the expression is

::

    (expr1, expr2, ...) IN ((test11, test12, ...), (test21, test22, ...), ...)


ALL, ANY, SOME
##############

We can alternately use comparison operators, in conjunction with the **ALL** or **ANY** or **SOME** keywords, to compare an expression against the results of a subquery.  For example, we can ask again for books which have won awards by using the equality operator together with the **ANY** keyword as follows (note that the **ALL**/**ANY**/**SOME** keywords are not supported by SQLite, so you cannot test this within the textbook's interactive tools):

::

    SELECT * FROM books WHERE book_id = ANY
      (SELECT book_id FROM books_awards);

**SOME** is just a synonym for **ANY**.  The **IN** operator when used with subqueries is equivalent to **= ANY**.  However, **ANY** cannot be used with an expression list in the same way **IN** can.

In contrast, **ALL** requires that every row returned the subquery passes the comparison test.  For example, to find books published before all books by the author Willa Cather:

::

    SELECT * FROM books WHERE publication_year < ALL
      (SELECT publication_year FROM books WHERE author_id =
        (SELECT author_id FROM authors WHERE name = 'Willa Cather')
      )
    ;

Note here we have used a subquery inside another subquery!  We can nest subqueries in this fashion; we can also use multiple subqueries within a compound Boolean expression.

The **NOT IN** operator is equivalent to **<> ALL**.


Use in statements
-----------------

Subqueries do not have to be used only within other **SELECT** queries.  The use of subqueries within the **WHERE** clause of **DELETE** or **UPDATE** statements can be very powerful, often making up for the fact that we cannot do joins within those types of statements.  For example, we could use a subquery to remove any authors from our database for whom we have no books:

::

    DELETE FROM authors
    WHERE author_id NOT IN
      (SELECT author_id FROM books);

There are no rows matching this condition in the database (unless you add them), so the above query does not remove any rows, although it runs successfully.


Correlated subqueries
:::::::::::::::::::::

In all of our examples so far, we used subqueries which are executable on their own as separate **SELECT** queries.  The subquery can be executed once, and the result of the subquery substituted in its place in the outer query.  It is possible, however, to construct queries that are dependent on the outer query.  When a subquery references some attribute from the outer query in an expression, we say that the subquery is *correlated* with the outer query.

For example, consider the problem of finding books published after the author's death (posthumous books).  We previously saw a way of using a subquery to get books published in the same year as the author's death:

::

    SELECT a.name AS author, b.title, b.publication_year
    FROM
      authors AS a
      JOIN books AS b ON a.author_id = b.author_id
    WHERE
      (a.author_id, b.publication_year) IN
        (SELECT author_id, substring(death, 1, 4) FROM authors)
    ;

It is not clear how we can modify this query's **WHERE** clause to indicate that we want the author ids to match, but that an inequality shoudld be used to compare the publication year with the author's death year.  What we want to do is, for each book in the outer query, compare its publication year to the death year of *its author only*.  To do this, we need our subquery to only return results relevant for the current row in the outer query - in this case, the subquery should return the scalar value representing the book's author's death year.

Here is the solution:

.. activecode:: subqueries_example_correlated
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT
      a1.name AS author, a1.death, b.title, b.publication_year
    FROM
      authors AS a1
      JOIN books AS b ON a1.author_id = b.author_id
    WHERE b.publication_year >
      (SELECT substring(death, 1, 4)
       FROM authors AS a2
       WHERE a2.author_id = a1.author_id)
    ;

Note that we have a situation where ambiguity must be resolved using aliasing - we have two instances of the **authors** table, one used in the outer query and one in the subquery.  If we simply refer to **author_id** in the subquery, SQL assumes we mean the subquery's **authors** table.  To refer to the outer query's **authors** table, we must give it an alias (**a1**) to distinguish it.  While not necessary, we have chosen to alias the subquery's table (**a2**) as well, to avoid any chance of confusion.

As you can see, we can no longer run the subquery independent of the outer query.  In effect, we are running the subquery over and over again, once for each row we encounter in the outer query.

As in this example, correlated subqueries tend to be most useful when both the outer query and the subquery work with the same table.  When the outer query and subquery work with different tables, it is typically possible to write the query as either correlated or uncorrelated.

EXISTS
------

The **EXISTS** operator precedes a subquery; there is no operand before the **EXISTS** keyword.  An **EXISTS** expression evaluates to ``True`` only if the subquery returns one or more rows.  The actual data from the subquery is ignored, so you can put anything you want in the **SELECT** clause.  We will use a constant ``1`` in our examples, just to emphasize that the data we are returning is unimportant.

Many uncorrelated subqueries can be rewritten as correlated subqueries using **EXISTS**.  For example, to find all books that have won awards, we can either do

::

    SELECT * FROM books WHERE book_id IN
      (SELECT book_id FROM books_awards)
    ;

as we did earlier, or, using **EXISTS**:

::

    SELECT * FROM books AS b WHERE EXISTS
      (SELECT 1
       FROM books_awards AS ba
       WHERE ba.book_id = b.book_id)
    ;

You can also use **NOT EXISTS**, which evaluates to the Boolean inverse of **EXISTS**.

Subqueries in other clauses
:::::::::::::::::::::::::::

We have seen numerous examples of subqueries used in **WHERE** clauses.  However, subquery expressions can be used in other contexts.  In particular, subqueries returning scalars can be useful in **SELECT** clauses and in the **SET** clauses of **UPDATE** statements.  Subqueries returning tables can also be used in place of named tables in the **FROM** clause of a **SELECT** clause.

SELECT
------

Used in a **SELECT** clause, subqueries can be used to retrieve values that are not easily obtained from the tables used in the outer query.  Used in this way, the subquery must return a scalar.  These subqueries are almost always correlated, as we want to return a value that is specific to each row.

For example, we might want to include, in a listing of books, the total number of books written by the author.  For this we will use the aggregate expression **COUNT(\*)**, which simply counts the number of rows matching the **WHERE** clause in a **SELECT** query [#]_.  (Aggregates are discussed fully in :numref:`Chapter {number} <grouping-chapter>`.)

.. activecode:: subqueries_example_other_clauses
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT
      a.name AS author,
      (SELECT COUNT(*) FROM books AS b2 WHERE b2.author_id = a.author_id)
        AS author_total,
      b1.title
    FROM
      authors AS a
      JOIN books AS b1 ON b1.author_id = a.author_id
    ;

SET
---

Used in the **SET** clause of an **UPDATE** statement, subqueries provide a way to work around the issue that we cannot use joins in an **UPDATE**.  If we want to update rows in some table with data from a second table, we can simply use a subquery to obtain the proper value.

As an example, in preparation of this book's database, a statement was run to populate the **publication_year** column of **books** using book edition information.  (The **editions** table in the database only has entries for a few books, to keep the size manageable, but the original database had complete data.)  This statement uses another aggregate expression to obtain the earliest publication year from the editions table for each book:

::

    UPDATE books
    SET publication_year =
      (SELECT MIN(publication_year)
       FROM editions
       WHERE books.book_id = editions.book_id)
    ;

Note: if you run the statement above, you will update most books to have a ``NULL`` publication year - when the subquery returns zero rows, the result is interpreted as ``NULL``.  (Do not panic if you executed this statement - changes are only made to a copy of the data.  You can obtain an unmodified copy of the database by refreshing your browser window.)  You can modify the statement to only update rows for which we have editions data using another subquery:

::

    UPDATE books
    SET publication_year =
      (SELECT MIN(publication_year)
       FROM editions
       WHERE books.book_id = editions.book_id)
    WHERE EXISTS
      (SELECT 1 FROM editions WHERE books.book_id = editions.book_id)
    ;


FROM
----

Subqueries can also be used within the **FROM** clause of a **SELECT** query, in which case the subquery result acts like a table containing exactly the data returned by the subquery.  In this usage, the subquery expression *must* be given a name using aliasing.  The subquery cannot be correlated!  The subquery expression can be used to obtain computed data not available in any table in the database.  For example, above we used a correlated subquery to retrieve author book counts on a row-by-row basis to go with each book title.  We could instead compute all author totals using an uncorrelated subquery, and then join to the result as if it were a table.  (Here the subquery uses both grouping and aggregation, covered in :numref:`Chapter {number} <grouping-chapter>`.)

::

    SELECT
      a.name AS author,
      c.count AS author_total,
      b.title
    FROM
      authors AS a
      JOIN books AS b ON b.author_id = a.author_id
      JOIN
        (SELECT author_id, COUNT(*) AS count
         FROM books
         GROUP BY author_id) AS c
        ON c.author_id = a.author_id
    ;


Comparison with joins
:::::::::::::::::::::

Subqueries are comparable to joins in the sense that they both involve multiple tables.  There are many cases in which a subquery can substitute for a join or vice-versa.  However, there are some subtle differences.

First, of course, is that, short of using **SELECT** clause subqueries, you can only return data that actually appears in the outer query's tables.  If you need your result to contain data contained in multiple tables, it is generally best to join the tables rather than using **SELECT** clause subqueries.  (The example used above of a **SELECT** clause subquery is an exception, since the data we pulled in was not actually stored in any table.)  Using a separate subquery for each column needed is unwieldy, hard to read, and probably inefficient.

On the other hand, if you are retrieving data from one table only, it may be preferable to use a subquery.  Consider these two queries to retrieve books that have won awards:

.. activecode:: subqueries_example_comparison
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT * FROM books WHERE book_id IN
      (SELECT book_id FROM books_awards)
    ORDER BY title;

    SELECT b.*
    FROM
      books AS b
      JOIN books_awards AS ba ON ba.book_id = b.book_id
    ORDER BY b.title;

Both queries return the same data, but the second query has duplicate rows - each book appears once for each award it has won.  In the first query, the **IN** operator merely tests for the presence of a book in the awards table, not how many times it appears, so duplicates are avoided.

The **NOT EXISTS** and **NOT IN** operators are particularly interesting in that they can provide clean solutions to questions that otherwise require an outer join, such as listing the books which have *not* won awards.

In many cases, though, you have choices in how you approach a query.  Which you use depends on your personal preference and style.  Here are three different queries for finding books published in the same year as the author's death - one using an uncorrelated subquery (repeated from above), one using a correlated subquery with **EXISTS**, and one using a join:

::

    SELECT a.name AS author, b.title, b.publication_year
    FROM
      authors AS a
      JOIN books AS b ON a.author_id = b.author_id
    WHERE
      (a.author_id, b.publication_year) IN
        (SELECT author_id, substring(death, 1, 4) FROM authors)
    ;

    SELECT a1.name AS author, b.title, b.publication_year
    FROM
      authors AS a1
      JOIN books AS b ON a1.author_id = b.author_id
    WHERE EXISTS
      (SELECT 1
       FROM authors AS a2
       WHERE a2.author_id = a1.author_id
       AND substring(a2.death, 1, 4) = b.publication_year)
    ;

    SELECT a1.name AS author, b.title, b.publication_year
    FROM
      authors AS a1
      JOIN books AS b ON a1.author_id = b.author_id
      JOIN authors AS a2 ON
        a2.author_id = a1.author_id
        AND substring(a2.death, 1, 4) = b.publication_year
    ;


Self-check exercises
::::::::::::::::::::

This section contains some exercises using the books data set (reminder: you can get full descriptions of all tables in :ref:`Appendix A <appendix-a>`).  If you get stuck, click on the "Show answer" button below the exercise to see a correct answer.  There are many ways to answer these questions; try to use at least one subquery for each.

.. activecode:: subqueries_self_test_scalar_1
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to list books (title, publication_year) by the author Viet Thanh Nguyen:
    ~~~~

.. reveal:: subqueries_self_test_scalar_1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT title FROM books WHERE author_id =
          (SELECT author_id FROM authors WHERE name = 'Viet Thanh Nguyen')
        ;


.. activecode:: subqueries_self_test_scalar_2
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query giving the author of *How We Became Human*:
    ~~~~

.. reveal:: subqueries_self_test_scalar_2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM authors WHERE author_id =
          (SELECT author_id FROM books WHERE title = 'How We Became Human')
        ;


.. activecode:: subqueries_self_test_scalar_3
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to list authors born after the death of author Albert Camus:
    ~~~~

.. reveal:: subqueries_self_test_scalar_3_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM authors WHERE birth >
          (SELECT death FROM authors WHERE name = 'Albert Camus')
        ;


.. activecode:: subqueries_self_test_in_1
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to list books for which we have editions information:
    ~~~~

.. reveal:: subqueries_self_test_in_1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM books WHERE book_id IN
          (SELECT book_id FROM editions)
        ;


.. activecode:: subqueries_self_test_in_2
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to list the titles of books by living authors (assume a ``NULL`` death date means the author is living):
    ~~~~

.. reveal:: subqueries_self_test_in_2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT title FROM books WHERE author_id IN
          (SELECT author_id FROM authors WHERE death IS NULL)
        ;


.. activecode:: subqueries_self_test_challenge_1
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to list the authors who have won any kind of Pulitzer prize (a book award starting with the string 'Pulitzer') for their books:
    ~~~~

.. reveal:: subqueries_self_test_challenge_1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM authors WHERE author_id IN
          (SELECT author_id FROM books_awards WHERE award_id =
            (SELECT award_id FROM awards WHERE name LIKE 'Pulitzer%'))
        ;


.. activecode:: subqueries_self_test_challenge_2
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to list authors who have won book awards but not author awards:
    ~~~~

.. reveal:: subqueries_self_test_challenge_2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name
        FROM authors
        WHERE author_id IN
          (SELECT author_id FROM books WHERE book_id IN
            (SELECT book_id FROM books_awards))
        AND author_id NOT IN
          (SELECT author_id FROM authors_awards)
        ;


.. activecode:: subqueries_self_test_challenge_3
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find books by authors with only one book (according to our database).  *Hint*: one way is to ask, for each book, whether there exist *other* books by the same author.
    ~~~~

.. reveal:: subqueries_self_test_challenge_3_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT a.name, b1.title
        FROM
          authors AS a
          JOIN books AS b1 ON a.author_id = b1.author_id
        WHERE NOT EXISTS
          (SELECT 1
           FROM books AS b2
           WHERE b2.author_id = b1.author_id
           AND b2.book_id <> b1.book_id)
        ORDER BY a.name;  -- to make it easier to verify


.. activecode:: subqueries_self_test_challenge_4
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to list all awards (either author awards or book awards) won by author J. M. Coetzee.
    ~~~~

.. reveal:: subqueries_self_test_challenge_4_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name
        FROM awards
        WHERE award_id IN
          (SELECT award_id FROM authors_awards WHERE author_id =
            (SELECT author_id FROM authors WHERE name = 'J. M. Coetzee'))
        OR award_id IN
          (SELECT award_id FROM books_awards WHERE book_id IN
            (SELECT book_id FROM books WHERE author_id =
              (SELECT author_id FROM authors WHERE name = 'J. M. Coetzee')))
        ;


.. |chapter-end| unicode:: U+274F

|chapter-end|

----

**Notes**

.. [#] For this particular problem, we could instead use something called a *window function*, which will be discussed briefly in chapter XXX.



.. raw:: html

   <div style="width: 520px; margin-left: auto; margin-right: auto;">
   <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">
   <img alt="Creative Commons License" style="border-width:0; display:block; margin-left:
   auto; margin-right:auto;" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a>
   <br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/InteractiveResource"
   property="dct:title" rel="dct:type"><i>A Practical Introduction to Databases</i></span> by
   <span xmlns:cc="http://creativecommons.org/ns#" property="cc:attributionName">
   Christopher Painter-Wakefield</span> is licensed under a
   <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">
   Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.</div>
