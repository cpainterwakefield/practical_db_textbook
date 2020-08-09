===============
SQL Expressions
===============

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
::::::::

Literals are simple values expressed in a form that the database understands as a value.  There are only a few types of literals in SQL, although these can be converted to many different types in the database.  We will discuss SQL data types further in `Chapter 4`_.  The main types of literals you will encounter are:

.. _`Chapter 4`: ../04-table-creation/table-creation

- Numbers: these are expressed in the usual fashion, for example, ``-1``, ``3.14159``, ``0.0008``; depending on the database, you may also be able to use numeric literals in scientific notation or other formats, for instance, ``6.02e23``.
- Character strings: these are strings of characters enclosed in single quotes, for example, ``'apple'``.  If you need to express a literal character string which contains a single quote, you simply write the single quote twice; this is tricky to read, but produces the desired result.  For example,

::

    SELECT author FROM books WHERE title = 'The Handmaid''s Tale';

- Boolean values: ``True`` or ``False``.  Note, however, that not all SQL implementations support Boolean literals.
- The special value ``NULL``; we'll talk more about ``NULL`` below.

Some other types, such as dates, are expressed as strings or integers and converted by SQL to the appropriate type when doing comparison, data modification, etc.

Note that you can ask for literal expressions in the **SELECT** clause - this is sometimes useful.  In this case, the literal is evaluated as itself for each row in the table you are querying.  For example,

.. activecode:: ch2_example1_select2
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT 42, 'hello', author FROM books;

Notice that the output provides column names based on the literal expressions selected.  Later we will see how to change the names of columns in the output, if we want to make them more meaningful.

(This interactive SQL window connects to the same database as the earlier interactive window; it is repeated here for your convenience to avoid scrolling.)

Operators and functions
:::::::::::::::::::::::

SQL defines a number of useful operations on its various types.  Some of these use simple operators, as in mathematical expressions, while others take the form of functions.  `Appendix B`_ provides complete lists of the operators and functions defined by the SQL standard, but we'll discuss some of the most commonly used ones here, along with examples of their use.

.. _`Appendix B`: ../appendix-b-reference/reference.html

Mathematics
-----------

To start with, you can expect the basic arithmetic operators to work with any numeric values: addition (**+**), subtraction (**-**), multiplication (**\***), and division (**/**) are standard.  Your database may implement others, but make sure you read the documentation for your database to ensure other operators do what you think they do.  You can actually use your database as a simple calculator!  Try these in the interactive window:

::

    SELECT 4 + 7;
    SELECT 302.78 * 14;

(Note for Oracle users: Oracle requires all **SELECT** queries to have a **FROM** clause; the special table **dual** is provided for queries that use no columns and should return one row.  Thus, use ``SELECT 4 + 7 FROM dual;`` in Oracle.)

The SQL standard additionally provides functions for many useful mathematical functions, such as logarithms (**log**, **ln**, **log10**), exponentials (**exp**), square root (**sqrt**), modulus (**mod**), floor and ceiling (**floor**, **ceiling** or **ceil**), trigonometric functions (**sin**, etc.), and more.  Some examples:

::

    SELECT sqrt(3);
    SELECT log10(1e5);
    SELECT cos(0);

Numbers are also comparable; we will discuss the comparison operators in the section on Boolean operators and functions below.

As a somewhat contrived example applying mathematical operators to an actual table, consider finding out which century a book was published in.  In the English language, the 1st century is traditionally considered to be the years numbered 1 - 100.  Each subsequent 100 years adds 1 to the century, so the 20th century included the years 1901 - 2000.

With a little math, we can extract the century in which each book in our database was published:

::

    SELECT floor((publication_year + 99) / 100) FROM books;


Character string operators and functions
----------------------------------------

SQL provides two very useful string operators, **||** (two vertical bars) and **LIKE**. The operator **||** is used for string concatenation, which has many applications.  For example, if we don't like the columnar output from our **books** table, we could simply concatenate the columns together (with appropriate spacing or other separators):

::

    SELECT title || ', by ' || author FROM books;

(If you are working in SQL Server, you will need to use **+** instead of **||**; if you are working in MySQL, you will need to use the **concat** function: ``SELECT concat(title, ', by ', author) FROM books;``.)

The **LIKE** operator is a Boolean operator that is used almost exclusively in the **WHERE** clause.  **LIKE** provides very simple pattern matching capabilities in SQL.  A *pattern* is just a string that can contain regular text and special *wildcard* characters, which can match one or many unspecified characters.  The two wildcards are **%**, which can match any string of zero or more characters, and **_**, which can match exactly one of any character.  (If you are familiar with standard regular expression syntax, the **%** wildcard corresponds to the regular expression ".*", and the **_** wildcard corresponds to the regular expression ".".)  Regular text matches itself exactly.

Consider the case in which we recall the first name of an author, but not the full name, and wish to look up authors with that first name.  The **%** wildcard can be used here to stand in for the unknown part of the name:

::

    SELECT name FROM authors WHERE name LIKE 'Isabel %';

Since the **%** can match any string, the pattern ``'Isabel %'`` would match "Isabel Allende", "Isabel Granada", or "Isabel del Puerto" for example (only one of these is in our **authors** table, though).

Similarly, if we remember the last part of the name, but not the start, we can use the **%** operator again:

::

    SELECT name FROM authors WHERE name LIKE '% Ginsberg';

We can use the operator more than once:

::

    SELECT title FROM books WHERE title LIKE '%Love%';
    SELECT title FROM books WHERE title LIKE '%Invisible%';

For the last example, recall that **%** can match a zero-length string.

Now, suppose we are interested in authors who use an initial instead of their full first name.  An initial looks like some character followed by a period - both are required.  Here's what the query would look like, using both the **%** and **_** operators:

::

    SELECT name FROM authors WHERE name LIKE '_. %';

The **LIKE** operator can also be combined with two very useful functions, **upper** and **lower**; these functions put strings in all uppercase or lowercase, respectively.  These functions do not make sense in all language settings, of course.  You can use **upper** or **lower** whenever you want to get back strings in all uppercase or lowercase; you can also use them when pattern matching if you aren't sure of the capitalization of the strings in your database:

::

    SELECT * FROM books WHERE lower(title) LIKE '%love%';

In addition to the functions discussed so far, SQL provides functions for various string manipulations tasks, such as substring extraction or replacement, finding the location of a substring, trimming whitespace (or other characters) from the front and/or back of a string, and many more.  There is also a set of functions supporting various operations using regular expressions.  See `Appendix B`_ for more details.

Boolean operators
-----------------


Date and time operators and functions
-------------------------------------



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