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

Relational database foundations
:::::::::::::::::::::::::::::::

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

Here, **books** is the name of the table.  The **\*** is a special symbol used only with **SELECT** statements to mean "all columns in the table".  The table **books** is one table in the example database for this chapter.  The other tables is named **authors**.  The interactive example below will let you query this database; the query above is already set up for you - click on "Run" to see what it results in.  Then, modify the query to retrieve all rows from **authors**.  The column names for the table are shown across the top of the result table.

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

To some extent, the names of things (tables, columns, functions, etc.) act as if they are case-insensitive.  However, the behavior here varies among databases.  We'll explore more on this topic in the next chapter. << put in a link >>  A fairly common convention is to simply always put the names of things in lowercase all of the time.  The examples in this book will follow that convention, which will help distinguish keywords from things that exist in the database.

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

Retrieving all of the data from a table is useful, but often not what we want, especially if the table is very large (and tables can get very, very large!)  To see just a subset of rows, we include a **WHERE** clause in our query.  The **WHERE** clause consists of the keyword **WHERE**, followed by an *expression* that evaluates to true or false (a Boolean expression).  The **WHERE** clause goes after the **FROM** clause.  Expressions are discussed more a few sections below, but for now, let's see some simple examples (again, you can try these in the interactive tool):

::

    SELECT * FROM books WHERE author = 'Voltaire';

    SELECT author, title, genre
    FROM books
    WHERE publication_year > 1999;

    SELECT birth, death FROM authors WHERE name = 'Ralph Ellison';

Note that character string literals in SQL are enclosed with single quotes - not double quotes.  Double quotes are used in SQL for a different purpose.


.. index:: ORDER BY, DESC, ASC

Ordering data: the ORDER BY clause
----------------------------------

One surprising fact about relational databases is that the rows in a table are not necessarily ordered in any particular fashion.  In fact, relational DBMSes (RDBMSes) are permitted to store data in whatever fashion is most convenient or efficient, as well as to retrieve data however is most convenient.  For example, in many RDBMSes, data may be initially in the order in which it was added to the table, but a subsequent data modification statement (`Chapter 5`_) results in the data being re-ordered.


.. _`Chapter 5`: ../05-data-modification/data-modification.html


Not surprisingly, SQL provides a mechanism by which we can put rows in order by whatever criteria we wish.  This is accomplished via the **ORDER BY** clause, which almost always comes last in any query.  The key phrase **ORDER BY** is followed by a comma-separated list of expressions (again, we'll talk more about these soon), which must resolve to some type that can be put in order: numbers, strings (text), dates, etc.  By default numbers are sorted from smallest to largest, dates from earliest to latest.  Strings are a bit trickier in SQL, because different databases order them differently by default [#]_.  SQLite, by default, uses lexicographic ordering based on ASCII_ values.

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

It is also possible to reverse the ordering for any or all of the criteria using the **DESC** ("descending") keyword.  (You can also use **ASC**, but as that is the default, it is usually omitted.)  Example:

::
    
    SELECT * FROM books ORDER BY publication_year DESC;



Expressions
:::::::::::

An *expression* in SQL is anything that can be evaluated to give some value - literal values, operator expressions, function call expressions and so forth.  

Most importantly, the use of a column name in a SQL statement is a special expression that evaluates to the value stored in that column for the current row being processed.  So, when we do

::

    SELECT title FROM books;

the expression **title** is evaluated on a row-by-row basis for each row in the table **books**.

We can build more complex expressions from simple expressions using operators or functions.  When we do

::

    SELECT * FROM books WHERE author = 'Voltaire';

the query execution examines each row of the the table **books** in turn to evaluate the expression ``author = 'Voltaire'``.  This expression compares the value of the **author** columns to the literal value ``'Voltaire'`` using the **=** operator.  If the two are the same, the overall expression evaluates to true, and the row is included in the output; otherwise, the row is excluded.

So far we've also seen column expressions used in the **ORDER BY** clause.  As we introduce additional clauses in future chapters, more opportunities to use expressions will arise.  Below we examine some more types of expressions in SQL.

Literals
--------

Literals are simple values expressed in a form that the database understands as a value.  There are only a few types of literals in SQL, although these can be converted to many different types in the database.  We will discuss many different types that SQL uses in `Chapter 4`_.  The main types of literals you will encounter are:

.. _`Chapter 4`: ../04-table-creation/table-creation

- Numbers: these are expressed in the usual fashion, for example, ``-1``, ``3.14159``, ``0.0008``; depending on the database, you may also be able to use literals in scientific notation or other formats, e.g., ``6.02e23``.
- Character strings: these are strings of characters enclosed in single quotes, for example, ``'apple'``.  If you need to express a literal character string which contains a single quote, you simply write the single quote twice; this is tricky to read, but produces the desired result.  E.g.,

::

    SELECT author FROM books WHERE title = 'The Handmaid''s Tale';

- Boolean values: **True** or **False**.  Note, however, that not all SQL implementations support Boolean literals.
- The special value **NULL**; we'll talk more about **NULL** below.

Some other types, such as dates, are expressed as strings or integers and converted by SQL to the appropriate type when doing comparison, data modification, etc.

Note that you can ask for literal expressions in the **SELECT** clause - this is sometimes useful.  In this case, the literal is evaluated as itself for each row in the table you are querying.  For example,

::

    SELECT 42, 'hello' FROM books;


Operators and functions
-----------------------

SQL defines a number of useful operations on its various types.  Some of these use simple operators, as in mathematical expressions, while others take the form of functions.  `Appendix B`_ provides complete lists of the operators and functions defined by the SQL standard, but we'll discuss some of the most commonly used ones here, along with examples of their use.

.. _`Appendix B`: ../appendix-b-reference/reference.html

To 

======== ===================== ================================ ===========================================
operator meaning               usage
======== ===================== ================================ ===========================================
\+       addition              *x* + *y*
\-       subtraction           123.45 - 0.10
\*       multiplication        10 * 3
/        division              1024 / 128
ABS      absolute value        ABS(-15.5)
MOD      modulus (remainder)   MOD(72, 5)                       integers only
LOG      logarithm             LOG(*base*, *x*)                 in SQL Server, use LOG(*x*, *base*)
LN       natural logarithm     LN(2.71828)                      in SQL Server, use LOG(*x*)
LOG10    base-10 logarithm     LOG10(1000)                      in Oracle, use LOG(10, *x*)
EXP      exponential function  EXP(1.0)
POWER    raise to the power    POWER(*base*, *x*)
SQRT     square root           SQRT(123.45)
FLOOR    floor function        FLOOR(123.45)
CEIL     ceiling function      CEIL(123.45)                     same as CEILING
SIN      sine function         SIN(3.14159)                     argument in radians
COS      cosine function       COS(3.14159)
TAN      tangent function      TAN(3.14159)
ASIN     inverse sine          ASIN(1)
ACOS     inverse cosine        ACOS(1)
ATAN     inverse tangent       ATAN(1e10)
======== ===================== ================================ ===========================================



- useful functions and operators
    - Boolean operators
    - Math functions and operators
    - String functions and operators 
    - Date functions and operators
    - Miscellaneous


NULL
::::


- NULL
    - meaning of
    - behavior in expressions



Miscellanous topics
:::::::::::::::::::

- DISTINCT



- basic expressions
    - use a few basic operators for example
    - using expressions in SELECT
    - using expressions in WHERE
    - using expressions elsewhere (e.g., ORDER BY)
    - literal expressions (strings, numbers, etc.) - reference data types in chapter 4
    - allude to more complex - e.g., table value, tuple values, etc.

A look ahead 
::::::::::::

Topics still to cover relating to SELECT: joins, subqueries, grouping & aggregation, set operations, and more

For some of these, need a table showing database implementation?  Or just SQL standard... maybe move table to appendix...


.. [#] You can change the sort order for strings by applying the **COLLATE** operator. **COLLATE** is a bit out of scope for this textbook, and varies with the dialect of SQL.  Please see the documentation for your particular DBMS.

