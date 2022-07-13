.. _advanced-sql-chapter:

===============
Advanced topics
===============

This chapter provides brief coverage of some advanced SQL capabilities that did not fit neatly into previous chapters: views, common table expressions, and window functions.  These topics are not thoroughly covered (particularly common table expressions and window functions).  It is hoped that this introduction will suffice to give you some understanding of the additional possibilities within SQL.

Tables used in this chapter
:::::::::::::::::::::::::::

For this chapter we will be using the books dataset (with tables **books**, **authors**, etc.), a description of which can be found in :ref:`Appendix A <appendix-a>`.


Views
:::::

A *view* is a database object that acts as a saved **SELECT** query, the result of which can be queried directly as if it were a table.  The data itself is not stored; the query that the view represents must be executed each time the view is queried, ensuring that up-to-date data is returned [#]_.  Views are particularly useful for saving complex queries that must be executed frequently, or that will be used by people with minimal SQL skills; the complex parts of the query can be written once, tested, and saved, reducing errors in later usage.

To create a view, use the **CREATE VIEW** statement:

.. activecode:: advanced_example_view
    :language: sql
    :dburl: /_static/textbook.sqlite3

    CREATE VIEW book_editions AS
    SELECT
      a.name AS author,
      b.title,
      e.publication_year,
      e.publisher,
      e.publisher_location,
      e.title AS published_title,
      e.pages,
      e.isbn10,
      e.isbn13
    FROM
      editions AS e
      JOIN books AS b ON b.book_id = e.book_id
      JOIN authors AS a ON a.author_id = b.author_id
    ;

    SELECT author, published_title, publication_year, publisher
    FROM book_editions
    WHERE title = 'The Two Towers';

To remove a view, use the **DROP VIEW** statement:

::

    DROP VIEW book_editions;


Common table expressions
::::::::::::::::::::::::

Related to both views and subqueries, *common table expressions* (CTEs) let us define a **SELECT** query and assign it a name for use within the context of a larger **SELECT** query.  Multiple CTEs may be used within a query.  Unlike a view, CTEs only exist for the lifetime of the query in which they are defined.  Unlike subqueries, CTEs may not be correlated with the main query (unless used itself in a subquery).  A common use of CTEs is in place of subqueries used in the **FROM** clause of the main query; the CTE effectively moves the subquery out of the body of the main query, which makes it easier to read.  In addition, one CTE can refer to another CTE defined earlier in the query, which eliminates the need to nest subqueries of this type.

CTEs are defined prior to the main **SELECT** clause, using a **WITH** clause:

::

    WITH
      name1 AS
        (select query 1),
      name2 AS
        (select query 2),
      ...
    SELECT ...

Here is an example listing books along with some additional pieces of information: the number of awards the book has won, and the number of printed editions of the book (keeping in mind that we only have edition information for books by J.R.R. Tolkien).  We could easily provide either one of these pieces of information simply using joins and grouping and aggregation, but providing both in the same query would require writing at least one subquery or using window functions (which are discussed in the next section).  Here we use CTEs to do our grouping and aggregation steps separately, then we join those results in the main query.

.. activecode:: advanced_example_cte
    :language: sql
    :dburl: /_static/textbook.sqlite3

    WITH
      ec AS
        (SELECT book_id, COUNT(*) AS count
         FROM editions
         GROUP BY book_id),
      ac AS
        (SELECT b.book_id, COUNT(ba.book_id) AS count
         FROM
          books AS b
          LEFT JOIN books_awards AS ba ON b.book_id = ba.book_id
        GROUP BY b.book_id)
    SELECT
      au.name AS author,
      ac.count AS "awards won",
      ec.count AS "editions in print",
      b.title
    FROM
      authors AS au
      JOIN books AS b ON b.author_id = au.author_id
      JOIN ac ON ac.book_id = b.book_id
      LEFT JOIN ec ON ec.book_id = b.book_id
    ;


Window functions
::::::::::::::::

As we saw in :numref:`Chapter {number} <grouping-chapter>`, grouping and aggregation let us report aggregate statistics on groups of data, along with attributes common to the group (typically, attributes that we grouped by).  However, the individual elements of the group are not visible.  *Window functions* provide a mechanism for reporting information related to some grouping of data while also listing all individual rows.  In general, all aggregate functions are available as window functions, and there are additional functions that relate a row to its membership in the group (such as its rank within the group according to some ordering).

As an example, suppose we wish to list all books, along with the number of books by the same author, and the ordinal number of the book as part of the author's body of work, in order by publication year (e.g., was this the author's first, second, or third book?).  We can do this with window functions:

.. activecode:: advanced_example_window
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT
      a.name AS author,
      COUNT(*) OVER
        (PARTITION BY b.author_id)
        AS author_count,
      ROW_NUMBER() OVER
        (PARTITION BY b.author_id ORDER BY b.publication_year)
        AS book_rank,
      b.title,
      b.publication_year
    FROM
      authors AS a
      JOIN books AS b ON b.author_id = a.author_id
    ORDER BY a.name, book_rank;

Note that windowing occurs *after* application of any **WHERE** conditions, and even after grouping and application of **HAVING** conditions.  This makes window functions useful in application to already grouped data, for example, but it also means that you cannot apply **WHERE** or **HAVING** conditions to the window function result itself.

Window functions have a number of additional options allowing for fairly complex processing, which we do not cover here.



|chapter-end|

----

**Notes**

.. [#] Some databases also provide *materialized views*, which store actual data; these are used when executing the query for a view would take too long.  Such views do become out of date and must be refreshed periodically.

|license-notice|
