========================
Data retrieval using SQL 
========================

From this chapter, you will learn the basic structure of a relational database and the basics of retrieving data using SQL.

.. index:: 
    single: table; defined
    single: table; schema
    single: column; defined
    seealso: column; attribute
    seealso: table; relation

Tables
::::::

While relational databases can contain many types of *objects*, the object type that stores data is the *table*.  Each table in the database has a name, which usually provides some indication of what kind of data can be found in the table.  The table structure is defined by the table's *columns*, each of which has a name and an associated data type.  The actual data is contained in the *rows* of the table; each row is one data point and has a value for each column of the table.

We can visualize a simple table as, well, a table:

.. image:: table_illustration.svg

The illustration above shows a table named "fruit_stand" with three columns named "item", "price", and "unit".  Although the illustration does not show the data types, we might infer that the **item** and **unit** columns contain text and the **price** column contains decimal numbers.  Each row of **fruit_stand** contains information about one kind of fruit sold at the fruit stand.

Some notes on terms:

- *Tables* are some times called *relations*, which is their name in the *relational theory* (see chapter XXX) that relational databases are based on.
- The definition of a table's structure is some times called it's *schema* (although that is a word that is heavily overloaded to mean different things in different contexts, so beware).
- *Columns* are also known as *attributes*, particularly in the context of the table's specification and in relational theory.

.. index:: SELECT, FROM, clause

Retrieving data using SELECT
::::::::::::::::::::::::::::

In its simplest form, the **SELECT** statement can be used to retrieve all data from a table.  We just need to know its name:

::

    SELECT * FROM books;

Here, **books** is the name of the table.  The **\*** is a special symbol used with **SELECT** statements to mean "all columns in the table".  The table **books** is one table in the example database for this chapter.  The other tables is named **authors**.  The interactive example below will let you query this database; the query above is already set up for you - click on "Run" to see what it results in.  Then, modify the query to retrieve all rows from **authors**.  The column names for the table are shown across the top of the result table.

.. activecode:: ch2_example1_select
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * FROM books;
    
The statement (or query) above is said to have two *clauses*; a clause is a part of a SQL statement, usually starting with a SQL keyword.  The two clauses in the statement above are the **SELECT** clause, "SELECT \*" and the **FROM** clause, "FROM books".  Most clauses are optional, in the sense that they are not required in every query, although they will be necessary to produce certain desired results.

SQL statement rules and conventions
-----------------------------------

First, note that SQL statements are properly terminated by semicolons.  In some software tools, single statements are allowed to be unterminated - this is true in our interactive examples, in fact.  However, we will always show the semicolon in our examples, as they become very important in settings where you want to send a list of statements to the database at one time.

One implication of this is that it is entirely permissible and (in many cases preferable) to write statements on multiple lines.  The query below is correct, and equivalent to the query above:

::

    SELECT *
    FROM books;

Next, SQL keywords are case-insensitive.  That is, we can write:

:: 

    select * from books;
    Select * From books;
    select * FROM books;

and get the same result for each query.  In the examples in this book, the convention is that keywords will be capitalized.

To some extent, the names of things (tables, columns, functions, etc.) act as if they are case-insensitive.  However, the behavior here varies among databases.  We'll explore more on this topic in the `Chapter 4`_.  A fairly common convention is to simply always put the names of things in lowercase all of the time.  The examples in this book will follow that convention, which will help distinguish keywords from things that exist in the database.

.. _`Chapter 4`: ../04-joins/joins.html

Note the conventions used in this textbook may be different from those used by your teacher, at your place of work, or in code found on the internet!


Retrieving specific columns
---------------------------

So far, we've only retrieved all columns of a table, which may not be the desired result.  Also, the columns are retrieved in the order in which they are defined in the table specification.  We can specify the columns we wish to retrieve by replacing the **\*** in our **SELECT** clause with a comma-separated list of columns:

::

    SELECT author, title
    FROM books;

(Try this and other example queries using the interactive tool above!)


.. index:: WHERE

Filtering rows: the WHERE clause
--------------------------------

Retrieving all of the data from a table is useful, but often not what we want, especially if the table is very large (and tables can get very, very large!)  To see just a subset of rows, we include a **WHERE** clause in our query.  The **WHERE** clause consists of the keyword **WHERE**, followed by an *expression* that evaluates to true or false (a Boolean expression).  The **WHERE** clause goes after the **FROM** clause.  Expressions are discussed more in `the next chapter`_, but for now, let's see some simple examples (again, you can try these in the interactive tool):

.. _`the next chapter`: ../03-expressions/expressions.html

::

    SELECT * FROM books WHERE author = 'Voltaire';

    SELECT author, title, genre
    FROM books
    WHERE publication_year > 1999;

    SELECT birth, death FROM authors WHERE name = 'Ralph Ellison';

Note that character string literals in SQL are enclosed with single quotes - not double quotes.  Double quotes are used in SQL for a different purpose.

Queries can return zero, one, or many rows.  If no rows match the **WHERE** condition, no rows are returned:

::

    SELECT * FROM books WHERE genre = 'romance';


.. index:: ORDER BY, DESC, ASC

Ordering data: the ORDER BY clause
----------------------------------

One surprising fact about relational databases is that the rows in a table are not necessarily ordered in any particular fashion.  In fact, relational DBMSes (RDBMSes) are permitted to store data in whatever fashion is most convenient or efficient, as well as to retrieve data however is most convenient.  For example, in many RDBMSes, data may be initially in the order in which it was added to the table, but a subsequent data modification statement (`Chapter 6`_) results in the data being re-ordered.


.. _`Chapter 6`: ../06-data-modification/data-modification.html


Not surprisingly, SQL provides a mechanism by which we can put rows in order by whatever criteria we wish.  This is accomplished via the **ORDER BY** clause, which almost always comes last in any query.  The key phrase **ORDER BY** is followed by a comma-separated list of expressions (again, we'll talk more about these soon), which must resolve to some type that can be put in order: numbers, strings (text), dates, etc.  By default numbers are sorted from smallest to largest, dates from earliest to latest.  Strings are a bit trickier, because different databases order them differently by default [#]_.  SQLite, by default, uses lexicographic ordering based on ASCII_ values.

.. _ASCII: https://en.wikipedia.org/wiki/ASCII

Here are some simple queries to try:

::

    SELECT * FROM books ORDER BY publication_year;

    SELECT * FROM authors ORDER BY birth;


Ordering is first applied using the first expression after the **ORDER BY**.  If any two rows are equal according to that expression, and there are additional expressions, they are applied with groups of rows that have equal values for the first expression, and so forth.  For example, suppose you are organizing books for a library or bookstore where books are grouped by genre, and then alphabetized by title.  You could do the following query to help with this task:

::

    SELECT author, title, genre
    FROM books
    ORDER BY genre, title;

It is also possible to reverse the ordering for any or all of the criteria using the **DESC** ("descending") keyword.  (You can also use **ASC**, but as that is the default, it is usually omitted.)  If we want to see all books from most recent to least recent, we can do:

::
    
    SELECT * FROM books ORDER BY publication_year DESC;

.. index:: DISTINCT, uniqueness

Retrieving unique rows: the DISTINCT keyword
--------------------------------------------

As we'll see in later chapters, it is usually good practice to set up database tables in way that each record in the table is unique; that is, for each row, there is no other row in the table that is exactly the same in every column.

However, queries that **SELECT** a sub-set of the columns of a table can easily end up with duplicate results; this may or may not be desired.  Suppose you were interested in browsing the books in our database for particular genres of books, but you weren't sure what genres the database puts books into - that is, what are valid choices given the data?

You could simply do:

::

    SELECT genre FROM books;

and for our small collection of books, that would probably be fine - there are duplicate values, but we can pretty quickly come up with a unique set.  However, a real database of books would contain thousands or millions of books.  You wouldn't want to browse that many rows to discover the possible genres!

SQL provides a keyword, **DISTINCT**, that goes after the **SELECT** keyword and tells SQL that we only want unique results - if there are duplicates, discard them.  This will give us the desired result, a unique set of genres that we can choose from:

::

    SELECT DISTINCT genre FROM books;


Self-check exercises
::::::::::::::::::::

This section contains some simple exercises using the same books and authors database used in the text above.  If you get stuck, click on the "Show answer" button below the exercise to see a correct answer.

.. activecode:: ch2_self_test_select
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Modify the SQL statement below to retrieve author names only:
    ~~~~
    SELECT * FROM authors;

.. reveal:: ch2_self_test_select_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM authors;


.. activecode:: ch2_self_test_where1
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find all books in the science fiction genre:
    ~~~~
    

.. reveal:: ch2_self_test_where1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM books WHERE genre = 'science fiction';


.. activecode:: ch2_self_test_where2
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find the publication year and author for the book *Bodega Dreams*:
    ~~~~
    

.. reveal:: ch2_self_test_where2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT publication_year, author 
        FROM books 
        WHERE title = 'Bodega Dreams';


.. activecode:: ch2_self_test_where3
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find all books published prior to 1700;
    ~~~~
    

.. reveal:: ch2_self_test_where3_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM books WHERE publication_year < 1700;


.. activecode:: ch2_self_test_order
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to get books in order by title:
    ~~~~
    

.. reveal:: ch2_self_test_order_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM books ORDER BY title;


.. activecode:: ch2_self_test_challenge1
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to get the authors publishing since 1980, in order by author name:
    ~~~~
    

.. reveal:: ch2_self_test_challenge1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT author
        FROM books
        WHERE publication_year > 1979
        ORDER BY author;


.. activecode:: ch2_self_test_challenge2
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to get the unique publication years for the books in our database published since 1980, ordered latest to earliest:
    ~~~~
    

.. reveal:: ch2_self_test_challenge2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT DISTINCT publication_year 
        FROM books
        WHERE publication_year > 1979
        ORDER BY publication_year;


----

**Notes**

.. [#] You can change the sort order for strings by applying the **COLLATE** operator. **COLLATE** is a bit out of scope for this textbook, and varies with the dialect of SQL.  Please see the documentation for your particular DBMS.

