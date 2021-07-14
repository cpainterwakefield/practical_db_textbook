===============
SQL Expressions
===============

.. _`Chapter 2`: ../02-data-retrieval/data-retrieval.html
.. _`Chapter 4`: ../04-joins/joins.html
.. _`Chapter 5`: ../05-table-creation/table-creation.html
.. _`Appendix B`: ../appendix-b-reference/reference.html

.. index:: expression

An *expression* in SQL is anything that can be *evaluated* - anything that results in a value.  Some examples include literal values, operator expressions, and function call expressions.  Expressions are used in most clauses of a SQL query; for example, in the **SELECT** clause, expressions result in the values we see returned from the query, while in the **WHERE** clause, expressions determine whether or not a row is returned from the query.

.. index:: column expression

Column expressions
::::::::::::::::::

The use of a column name in a SQL statement is a special expression that evaluates to the value stored in that column for the current row being processed.  So, when we do

.. activecode:: ch3_section1
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT title FROM books;  

the query execution examines each row of the the table **books** in turn to evaluate the expression ``author = 'Voltaire'``.  This expression compares the value of the **author** column to the literal value ``'Voltaire'`` using the **=** operator.  If the two are the same, the overall expression evaluates to ``True``, and the row is included in the output; otherwise, the row is excluded.

.. index:: literal; numeric, literal; character string, literal; Boolean

Literals
::::::::

Literals are simple values expressed in a form that the database understands as a value.  There are only a few types of literals in SQL, although these can be converted to many different types in the database.  We will discuss SQL data types further in `Chapter 5`_.  The main types of literals you will encounter are:

- Numbers: these are expressed in the usual fashion, for example, ``-1``, ``3.14159``, ``0.0008``; depending on the database, you may also be able to use numeric literals in scientific notation or other formats, for instance, ``6.02e23`` (which stands for :math:`6.02 \times 10^{23}`).
- Character strings: these are strings of characters enclosed in single quotes, for example, ``'apple'``.  If you need to express a literal character string which contains a single quote, you simply write the single quote twice; this is tricky to read, but produces the desired result.  For example,

.. activecode:: ch3_section2
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT author FROM books 
    WHERE title = 'The Handmaid''s Tale';

- Boolean values: ``True`` or ``False``.  Note, however, that not all SQL implementations support Boolean literals.
- Date and time values; how to notate these varies widely among different SQL implementations.
- The special value ``NULL``; we'll talk more about ``NULL`` below.

You can ask for literal expressions in the **SELECT** clause - this is sometimes useful.  In this case, the literal is evaluated as itself for each row in the table you are querying.  For example,

::

    SELECT 42, 'hello', author FROM books;

If you try this query in the interactive tool above, note that the output provides column names based on the literal expressions selected.  Later we will see how to change the names of columns in the output, if we want to make them more meaningful.


Operators and functions
:::::::::::::::::::::::

SQL defines a number of useful operations on its various types.  Some of these use simple operators, as in mathematical expressions, while others take the form of functions.  `Appendix B`_ provides more extensive lists of operators and functions defined by the SQL standard, but we'll discuss some of the most commonly used ones here, along with examples of their use.

.. index:: operators; comparison

Comparison operators
--------------------

We've already seen the equality operator (**=**) used to test if some column is equal to a literal value in the **WHERE** clause of queries.  We can instead test for inequality using the (**<>**) operator:

.. activecode:: ch3_section3_1
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * FROM books WHERE genre <> 'fantasy';

Though non-standard, most databases also recognize **!=** as an inequality operator.

We can also test to see if a value is less than (**\<**), greater than (**\>**), less than or equal to (**\<=**), or greater than or equal to (**\>=**) some other value.  There is also a ternary operator, **BETWEEN**, that tests if a value is between two other values (see Appendix B, `Comparison operators <../appendix-b-reference/reference.html#comparison-operators>`_ for details).

.. index:: operators; mathematics, functions; mathematics

Mathematics
-----------

You can expect the basic arithmetic operators to work with any numeric values: addition (**+**), subtraction (**-**), multiplication (**\***), and division (**/**) are standard.  Your database may implement others, but make sure you read the documentation for your database to ensure other operators do what you think they do.  You can actually use your database as a simple calculator!  Try running these:

.. activecode:: ch3_section3_2
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT 4 + 7;
    SELECT 302.78 * 14;

(Note for Oracle users: Oracle requires all **SELECT** queries to have a **FROM** clause; the special table **dual** is provided for queries that use no columns and should return one row.  Thus, use ``SELECT 4 + 7 FROM dual;`` in Oracle.)

The SQL standard additionally provides functions for many useful mathematical functions, such as logarithms (**log**, **ln**, **log10**), exponentials (**exp**), square root (**sqrt**), modulus (**mod**), floor and ceiling (**floor**, **ceiling** or **ceil**), trigonometric functions (**sin**, etc.), and more.  Some examples:

::

    SELECT sqrt(3);
    SELECT log10(1e5);
    SELECT cos(0);

You will most likely find yourself using mathematical operators in SQL if you are working with numerical data such as financial data or scientific data.  In `Chapter 4`_ we will discuss the many different data types available for storing numbers - integers, decimal numbers, and floating point values.  Each has applications to various problems.

As a somewhat contrived example applying mathematical operators to an actual table, consider finding out which century a book was published in.  In the English language, the 1st century is traditionally considered to be the years numbered 1 - 100.  Each subsequent 100 years adds 1 to the century, so the 20th century included the years 1901 - 2000.

With a little math, we can extract the century in which each book in our database was published:

::

    SELECT title, floor((publication_year + 99) / 100) AS century FROM books;

Note the use of parentheses to enforce an order of operations; the addition operation occurs before the division; the result of the division is provided to the **floor()** function.  We have also introduced something new - a renaming operation to give our result column a more informative name. The **AS** keyword lets us rename a column in the output of our query.  We will learn more about using **AS** in `Chapter 4`_.

See Appendix B, `Mathematical operators and functions <../appendix-b-reference/reference.html#mathematical-operators-and-functions>`_ for a complete list of standard operators and functions.

.. index:: operators; string, functions; string, string concatenation, LIKE


Character string operators and functions
----------------------------------------

SQL provides two very useful string operators, **||** (two vertical bars) and **LIKE**. The operator **||** is used for string concatenation, which has many applications.  For example, if we don't like the columnar output from our **books** table, we could simply concatenate the columns together (with appropriate spacing or other separators):

.. activecode:: ch3_section3_3
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT title || ', by ' || author FROM books;

(In SQL Server, you will need to use **+** instead of **||**; in MySQL, you will need to use the MySQL **concat** function, e.g.: ``SELECT concat(title, ', by ', author) FROM books;``.)

The **LIKE** operator is a Boolean operator that is used almost exclusively in the **WHERE** clause.  **LIKE** provides very simple pattern matching capabilities in SQL.  A *pattern* is just a string that can contain regular text and special *wildcard* characters, which can match one or many unspecified characters.  The two wildcards are **%**, which can match any string of zero or more characters, and **_**, which can match exactly one of any character.  (If you are familiar with standard regular expression syntax, the **%** wildcard corresponds to the regular expression ".*", and the **_** wildcard corresponds to the regular expression ".".)  Normal text matches itself exactly.

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

In addition to the functions discussed so far, SQL provides functions for various string manipulations tasks, such as substring extraction or replacement, finding the location of a substring, trimming whitespace (or other characters) from the front and/or back of a string, and many more.  See Appendix B, `Character string operators and functions <../appendix-b-reference/reference.html#character-string-operators-and-functions>`_ for these.

.. index:: operators; Boolean, AND, OR, NOT

Boolean operators
-----------------

As discussed in `Chapter 2`_, the **WHERE** clause of a **SELECT** query expects a Boolean expression after the **WHERE** keyword.  Some expressions that are Boolean in SQL include expressions using comparison operators, or an expression using the **LIKE** operator.  Many functions also result in a Boolean value.

SQL provides logical operators that operate on Boolean values.  These operators are **AND**, **OR**, and **NOT**, which perform the logical operations that their names imply.  For example, if we have an expression of the form ``expr1 AND expr2``, the result is ``True`` if and only if both ``expr1`` and ``expr2`` evaluate to ``True``.  Similarly, ``expr1 OR expr2`` evaluates to ``True`` if either ``expr1`` or ``expr2`` are ``True``.  Finally, ``NOT`` inverts the truth value:  ``NOT True`` results in ``False``, and ``NOT False`` results in ``True``.

These logical operators allow us to build up complex Boolean expressions from simpler Boolean expressions to express the particular logical conditions we want for our **WHERE** clause.  So, for example, we might be interested in fantasy books published since the year 2000:

.. activecode:: ch3_section3_4
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * 
    FROM books 
    WHERE genre = 'fantasy' AND publication_year > 2000;

Or, we might be interested in books in either the fantasy or science fiction genres:

::

    SELECT * FROM books 
    WHERE genre = 'fantasy' OR genre = 'science fiction';

If we simply hate science fiction, we might do

::

    SELECT * FROM books WHERE NOT genre = 'science fiction';

which gives the same result as

::

    SELECT * FROM books WHERE genre <> 'science fiction';
    
For more complex expressions involving combinations of **AND**, **OR**, and **NOT**, we may need to use parentheses to make our meaning clear.  In SQL, **NOT** is applied before **AND**, and **AND** is applied before **OR**. For example, perhaps we are interested in any books other than fantasy books published after the year 2000.  We might be tempted to write

::

    SELECT * FROM books
    WHERE NOT genre = 'fantasy' AND publication_year > 2000;

However, this isn't quite right (try it!).  Since the **NOT** is applied first, this query returns books that a) are not fantasy and b) were published since the year 2000.  The expression ``NOT genre = 'fantasy' AND publication_year > 2000`` is equivalent to ``(NOT genre = 'fantasy') AND (publication_year > 2000)``.  To get what we originally wanted, we need to use parentheses explicitly:
::

    SELECT * FROM books
    WHERE NOT (genre = 'fantasy' AND publication_year > 2000);

You can see that the above query only excludes the books in the list

::

    SELECT * FROM books
    WHERE genre = 'fantasy' AND publication_year > 2000;

Similarly, we might be interested in either science fiction or fantasy books, but only if they were published after 2000.  Compare the two queries below:

::

    SELECT *
    FROM books
    WHERE
        genre = 'science fiction'
        OR genre = 'fantasy'
        AND publication_year > 2000;

    SELECT *
    FROM books
    WHERE 
        (genre = 'science fiction' OR genre = 'fantasy')
        AND publication_year > 2000;

The first query above returns any science fiction books, and fantasy books published after 2000.  The second returns the desired result: books published after 2000 in either the fantasy or science fiction genres.

For a fuller discussion of Boolean operators, we need to know more about ``NULL`` values, which will be discussed below.  See Appendix B, `Boolean operators <../appendix-b-reference/reference.html#boolean-operators>`_ for a full reference on SQL Boolean operators.

.. index:: operators; date and time, functions; date and time

Date and time operators and functions
-------------------------------------

Date and time data are extremely important in many database applications, such as those supporting governmental or financial institutions.  SQL provides extensive functionality for managing dates and times.  Unfortunately, this is an area where different SQL implementations vary widely in their conformance to the SQL standard. See Appendix B, `Date and time operators and functions <../appendix-b-reference/reference.html#date-and-time-operators-and-functions>`_ for a fuller discussion, and consult your database implementation's documentation to see what capabilities it offers with respect to date and time handling.

One useful SQL function that most databases implement is the **CURRENT_DATE** function (also try **CURRENT_TIME** and **CURRENT_TIMESTAMP**):

.. activecode:: ch3_section3_5
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT CURRENT_DATE;

We will see in `Chapter 4`_ how this function can be used to automatically record the date in a newly created row.


    
NULL
::::


- NULL
    - meaning of
    - behavior in expressions



Miscellanous topics
:::::::::::::::::::

- basic expressions
    - use a few basic operators for example
    - using expressions in SELECT
    - using expressions in WHERE
    - using expressions elsewhere (e.g., ORDER BY)
    - literal expressions (strings, numbers, etc.) - reference data types in chapter 4
    - allude to more complex - e.g., table value, tuple values, etc.

    
Self-check exercises
::::::::::::::::::::