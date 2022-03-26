.. _data-retrieval-chapter:

==============
Data retrieval
==============

In this chapter, you will learn more about **SELECT** queries, including how to retrieve specific rows and how to sort the output.


Tables used in this chapter
:::::::::::::::::::::::::::

We will be working with the **simple_books** and **simple_authors** tables for this chapter.  As the names suggest, these are smaller, simplified versions of other tables we will work with later in the book, and the data concerns books and their authors.  You can read a full explanation of these tables in :ref:`Appendix A <appendix-a>`, but for now we recommend simply using a **SELECT** query to view all of the data in these two tables.  Here is an interactive tool you can use for that purpose:

.. activecode:: data_retrieval_example_tables
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT * FROM simple_books;

Remember from the previous chapter that you can **SELECT** \* to retrieve all columns, or you can specify the columns you want:

::

    SELECT author, title FROM simple_books;


.. index:: WHERE

Filtering rows: the WHERE clause
::::::::::::::::::::::::::::::::

Retrieving all of the data from a table is useful, but often not what we want, especially if the table is very large (and tables can get very, very large!)  To see just a subset of rows, we include a **WHERE** clause in our query.  The **WHERE** clause consists of the keyword **WHERE** followed by an *expression* that evaluates to true or false (a Boolean expression). [#]_  The **WHERE** clause is placed after the **FROM** clause.  Expressions are discussed more in :numref:`Chapter {number} <expressions-chapter>`, but for now, let's see some simple examples:

.. activecode:: data_retrieval_example_where
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT * FROM simple_books WHERE author = 'Isabel Allende';

    SELECT author, title, genre
    FROM simple_books
    WHERE publication_year > 1999;

    SELECT birth, death
    FROM simple_authors
    WHERE name = 'Ralph Ellison';

Note that character string literals in SQL are enclosed with single quotes - not double quotes.  Double quotes are used in SQL for a different purpose, which we'll see in :numref:`Chapter {number} <joins-chapter>`.

Queries can return zero, one, or many rows.  If no rows match the **WHERE** condition, zero rows are returned (try pasting this in one of the interactive tools above):

::

    SELECT * FROM simple_books WHERE genre = 'romance';


.. index:: ORDER BY, DESC, ASC

Ordering data: the ORDER BY clause
::::::::::::::::::::::::::::::::::

One surprising fact about relational databases is that the rows in a table are not necessarily ordered in any particular fashion.  In fact, relational DBMSes (RDBMSes) are permitted to store data in whatever fashion is most convenient or efficient, as well as to retrieve data however is most convenient.  For example, in many RDBMSes, data may be initially in the order in which it was added to the table, but a subsequent data modification statement results in the data being re-ordered.

SQL provides a mechanism by which we can put rows in order by whatever criteria we wish.  This is accomplished via the **ORDER BY** clause, which always comes last in any query.  The key phrase **ORDER BY** is followed by a comma-separated list of expressions, which must evaluate to some type that can be put in order: numbers, character strings, dates, etc.  By default, numbers are sorted from smallest to largest, and dates from earliest to latest.  Character strings are a bit trickier, because different databases order them differently. [#]_ SQLite, the dialect we are using, defaults to `lexicographic ordering <https://en.wikipedia.org/wiki/Lexicographic_order>`_ based on `ASCII <https://en.wikipedia.org/wiki/ASCII>`_ values.

Here are some simple queries to try:

.. activecode:: data_retrieval_example_order_by
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT * FROM simple_books ORDER BY publication_year;

    SELECT * FROM simple_authors ORDER BY birth;


Ordering is initially applied using the first expression after the **ORDER BY** keyword.  If any two rows are equal according to that first expression, and there are additional expressions in the **ORDER BY** clause, the next expression is then applied to groups of rows that have equal values for the first expression, and so forth.  For example, suppose you are organizing books for a library or bookstore where books are grouped by genre and then alphabetized by title.  You could write the following query to help with this task:

::

    SELECT author, title, genre
    FROM simple_books
    ORDER BY genre, title;

It is also possible to reverse the ordering for any or all of the criteria using the **DESC** ("descending") keyword.  (You can also use **ASC** for "ascending", but, as that is the default, it is usually omitted.)  If we want to see all books listed from most recent to least recent, we can write:

::

    SELECT * FROM simple_books ORDER BY publication_year DESC;


.. index:: DISTINCT, uniqueness

Retrieving unique rows: the DISTINCT keyword
::::::::::::::::::::::::::::::::::::::::::::

As we will see in later chapters, it is usually good practice to set up database tables in such as way that each record in the table is unique; that is, for each row, there will be no other row in the table that contains exactly the same data in every column.

However, queries that **SELECT** a subset of the columns of a table can easily end up with duplicate results; this may or may not be desired.  Suppose you were interested in browsing the books in our database for particular genres of books, but you weren't sure what genres the database puts books into - that is, you need to determine what would be valid choices given the data.

You could simply run the query:

.. activecode:: data_retrieval_example_distinct
    :language: sql
    :dburl: /_static/textbook.sqlite3

    SELECT genre FROM simple_books;

For this small collection of books, that would probably be fine - there are duplicate values, but we can pretty quickly come up with a unique set.  However, a real database of books could contain many thousands of books.  You wouldn't want to browse that many rows to discover the possible genres!

SQL provides a keyword, **DISTINCT**, that can be added after the **SELECT** keyword and tells SQL that we only want unique results, and if there are duplicates, it should discard them.  This will give us the desired result, a unique set of genres that we can choose from:

::

    SELECT DISTINCT genre FROM simple_books;


Self-check exercises
::::::::::::::::::::

This section contains some simple exercises using the **simple_books** and **simple_authors** tables used in the text above.  If you get stuck, click on the "Show answer" button below the exercise to see a correct answer.

.. activecode:: data_retrieval_self_test_select
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Modify the SQL statement below to retrieve author names only.
    ~~~~
    SELECT * FROM simple_authors;

.. reveal:: data_retrieval_self_test_select_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM simple_authors;


.. activecode:: data_retrieval_self_test_where1
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find all books in the science fiction genre.
    ~~~~


.. reveal:: data_retrieval_self_test_where1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM simple_books WHERE genre = 'science fiction';


.. activecode:: data_retrieval_self_test_where2
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find the publication year and author for the book *Bodega Dreams*.
    ~~~~


.. reveal:: data_retrieval_self_test_where2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT publication_year, author
        FROM simple_books
        WHERE title = 'Bodega Dreams';


.. activecode:: data_retrieval_self_test_where3
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to find all books published prior to 1950.
    ~~~~


.. reveal:: data_retrieval_self_test_where3_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM simple_books WHERE publication_year < 1950;


.. activecode:: data_retrieval_self_test_order
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to get books in order by title.
    ~~~~


.. reveal:: data_retrieval_self_test_order_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM simple_books ORDER BY title;


.. activecode:: data_retrieval_self_test_challenge1
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to get the authors publishing since 1980, in order by author name.
    ~~~~


.. reveal:: data_retrieval_self_test_challenge1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT author
        FROM simple_books
        WHERE publication_year > 1979
        ORDER BY author;


.. activecode:: data_retrieval_self_test_challenge2
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a query to get the unique publication years for the books in our database published since 1980, ordered latest to earliest.
    ~~~~


.. reveal:: data_retrieval_self_test_challenge2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT DISTINCT publication_year
        FROM simple_books
        WHERE publication_year > 1979
        ORDER BY publication_year DESC;


|chapter-end|

----

**Notes**

.. [#] There is actually a third possible value, ``NULL``, which may occur in expressions used in the **WHERE** clause of a query.  ``NULL`` is a complex topic which will be covered in :numref:`Chapter {number} <expressions-chapter>`.  For now, assume a normal Boolean result of true or false.

.. [#] You can change the sort order for strings by applying the **COLLATE** operator. **COLLATE** is out of scope for this textbook, and varies with the dialect of SQL.  Please see the documentation for your particular DBMS.


|license-notice|
