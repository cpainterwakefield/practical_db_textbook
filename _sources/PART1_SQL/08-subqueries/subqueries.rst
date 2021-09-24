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

Rows do not have to just contain literal values; any expressions can be put into a row expression.  Rows can be used in comparison expressions: the two queries below are equivalent (although the first is probably preferred as a matter of style):

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

We can use a subquery in place of a scalar expression as long as we know the subquery will return a single row and column.  For an example, let's return to an example from :numref:`Chapter {number} <joins-chapter>` using our **books** table: how can we find all books by the author of *The Three-Body Problem*?  We will use a subquery to first obtain the **author_id**, and then use that result to get our list of books:

.. activecode:: subqueries_example_single_row
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
    WHERE title = 'The Three-Body Problem';

Since this particular query returns exactly one row and one column, we treat the result as a scalar value, and simply substitute the scalar in place of the subquery:

::

    SELECT * FROM books WHERE author_id = 40;

Note that this only works because we only have one book with the title *The Three-Body Problem*.  In general it is not a good idea to assume a query will return a single row unless you use a **WHERE** clause condition on a column known to hold unique values, or unless you are computing an aggregate statistic over a set of rows (we will discuss aggregates in :numref:`Chapter {number} <grouping-and-aggregation-chapter>`).  If multiple rows are returned by the subquery, the query will result in an error.

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

When a query can return multiple rows, we have a different set of operators to work with.  In this section we discuss the **IN** operator and the use of comparison operators with **ALL**, **ANY**, and **SOME**.  Another Boolean operator, **EXISTS**, will wait until we discuss correlated subqueries later in the chapter.

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

EXISTS
------


Use in SELECT and SET clauses
:::::::::::::::::::::::::::::


Use in FROM clause
::::::::::::::::::


Comparison with joins
:::::::::::::::::::::

- equivalence/subtle differences (multiplicity)
- example: books that have won awards - must use join to get the actual award, but use IN if want to eliminate multiplicity




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
