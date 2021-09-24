.. _subqueries-chapter:

==============================
Subquery and tuple expressions
==============================

In this chapter we discuss the many ways in which we can use the result of a query within expressions in another query.


Tables used in this chapter
:::::::::::::::::::::::::::

For this chapter we will be using the books dataset (tables **books**, **authors**, etc.), a description of which can be found in :ref:`Appendix A <appendix-a>`.

Scalars, rows, and tables
:::::::::::::::::::::::::

Before we discuss subqueries, it is useful to talk about some additional types of data that come up in SQL.  So far we have assumed that all expressions evaluate to a *scalar* value.  Scalar values are simple values of a single type, such as ``42`` or ``'hello'``.  We can also work with *row* values in SQL.  A row is just an ordered list of values.  We can write down a literal row (SQL calls these "row value constructors") by putting a comma-separated list of expressions between parentheses:

::

    (1, 'hello', 3.1415)

In most databases (but not SQLite), you can **SELECT** the above expression as a literal, with the result showing as a single column.  We will see more useful applications of row expressions in the following sections.

Beyond rows, we can also think in terms of *tables* as values.  Here we are using tables to mean a collection of rows, not necessarily the named object living in the database.  The result from a **SELECT** query is a table in this sense.


Subqueries
::::::::::

A subquery is simply a **SELECT** query enclosed with parentheses and nested within another query or statement.  This *subquery expression* evaluates to a scalar, a row, a column, or a table, depending on the query results and the context in which the subquery appears.

Used as a scalar expression
---------------------------

We can use a subquery in place of a scalar expression as long as we know the subquery will return a single row and column.  For an example, let's return to an example from :numref:`Chapter {number} <joins-chapter>` using our **books** table: how can we find all books by the author of *The Three-Body Problem*?  We will use a subquery to first obtain the **author_id**, and then use that result to get our list of books:

.. activecode:: subqueries_example_simple
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT * FROM books WHERE author_id =
      (SELECT author_id
       FROM books
       WHERE title = 'The Three-Body Problem')
    ;

To see what is happening in this query, first execute the subquery on its own:

::

    SELECT author_id
    FROM books
    WHERE title = 'The Three-Body Problem'

Since this particular query returns exactly one row and one column, we treat the result as a scalar value, and simply substitute the scalar in place of the subquery:

::

    SELECT * FROM books WHERE author_id = 40;

Note that this only works because we only have one book with the title *The Three-Body Problem*.  In general it is not a good idea to assume a query will return a single row unless you use a **WHERE** clause condition on a column known to hold unique values, or unless you are computing an aggregate statistic over a set of rows (we will discuss aggregates in :numref:`Chapter {number} <grouping-and-aggregation-chapter>`).  If multiple rows are returned by the subquery, the query will result in an error.

Used as a row expression
------------------------

We can do something similar to the above, but comparing an entire row expression to the result of the subquery.  The subquery should have as many expressions in its **SELECT** clause as the row expression you are comparing.

- IN operator
- subqueries
    - interpreted as returning tuples
        - [NOT] IN
        - [NOT] EXISTS
        - [NOT] UNIQUE
        - ALL /

.. activecode:: subqueries_self_test_in
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a statement to create a table named **a_authors** containing just authors whose names start with the letter 'A':
    ~~~~

.. reveal:: subqueries_self_test_in_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        CREATE TABLE a_authors AS
          SELECT * FROM authors
          WHERE name LIKE 'A%'
        ;


.. |chapter-end| unicode:: U+274F

|chapter-end|

----

**Notes**
