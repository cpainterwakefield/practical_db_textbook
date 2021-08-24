===============
SQL expressions
===============

.. _`Chapter 2`: ../02-data-retrieval/data-retrieval.html
.. _`Chapter 4`: ../04-joins/joins.html
.. _`Chapter 5`: ../05-table-creation/table-creation.html
.. _`Chapter 8`: ../08-grouping-and-aggregation/grouping-and-aggregation.html
.. _`Appendix B`: ../../appendix-b-reference/reference.html

.. index:: expression

An *expression* in SQL is anything that can be *evaluated* - anything that results in a value.  Some examples include literal values, operator expressions, and function call expressions.  Expressions are used in most clauses of a SQL query; for example, in the **SELECT** clause, expressions result in the values we see returned from the query, while in the **WHERE** clause, expressions determine whether or not a row is returned from the query.  You can also **ORDER BY** expressions, and later we'll see other uses for expressions.  This chapter will explore some of the most common expression types; additional expressions will be introduced at the appropriate time.

.. index:: column expression

Column expressions
::::::::::::::::::

The use of a column name in a SQL statement is a special expression that evaluates to the value stored in that column for the current row being processed.  So, when we do

.. activecode:: ch3_example_column
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * FROM books WHERE author = 'J.R.R. Tolkien';

the query execution examines each row of the the table **books** in turn to evaluate the expression ``author = 'J.R.R. Tolkien'``.  This expression compares the value of the **author** column to the literal value ``'J.R.R. Tolkien'`` using the **=** operator.  If the two are the same, the overall expression evaluates to ``True``, and the row is included in the output; otherwise, the row is excluded.

.. index:: literal; numeric, literal; character string, literal; Boolean

Literals
::::::::

Literals are simple values expressed in a form that the database understands as a value.  There are only a few types of literals in SQL, although these can be converted to many different types in the database.  We will discuss SQL data types further in `Chapter 5`_.  The main types of literals you will encounter are:

- Numbers: these are expressed in the usual fashion, for example, ``-1``, ``3.14159``, ``0.0008``; depending on the database, you may also be able to use numeric literals in scientific notation or other formats, for instance, ``6.02e23`` (which stands for :math:`6.02 \times 10^{23}`).
- Character strings: these are strings of characters enclosed in single quotes, for example, ``'apple'``.  If you need to express a literal character string which contains a single quote, you simply write the single quote twice; this is tricky to read, but produces the desired result.  For example,

.. activecode:: ch3_example_literal
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

.. activecode:: ch3_example_comparison
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * FROM books WHERE genre <> 'fantasy';

Though non-standard, most databases also recognize **!=** as an inequality operator.

We can also test to see if a value is less than (**\<**), greater than (**\>**), less than or equal to (**\<=**), or greater than or equal to (**\>=**) some other value.  There is also a ternary operator, **BETWEEN**, that tests if a value is between two other values (see Appendix B, `Comparison operators <../../appendix-b-reference/reference.html#comparison-operators>`_ for details).

.. index:: operators; mathematics, functions; mathematics

Mathematics
-----------

You can expect the basic arithmetic operators to work with any numeric values: addition (**+**), subtraction (**-**), multiplication (**\***), and division (**/**) are standard.  Your database may implement others, but make sure you read the documentation for your database to ensure other operators do what you think they do.  You can actually use your database as a simple calculator!  Try running these:

.. activecode:: ch3_example_math
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

You will most likely find yourself using mathematical operators in SQL if you are working with numerical data such as financial or scientific data.  In `Chapter 4`_ we will discuss the many different data types available for storing numbers - integers, decimal numbers, and floating point values.  Each has applications to various problems.

As a somewhat contrived example applying mathematical operators to an actual table, consider the problem of finding out which century a book was published in.  In the English language, the 1st century is traditionally considered to be the years numbered 1 - 100.  Each subsequent 100 years adds 1 to the century, so the 20th century included the years 1901 - 2000.

With a little math, we can extract the century in which each book in our database was published:

::

    SELECT title, floor((publication_year + 99) / 100) AS century FROM books;

Note the use of parentheses to enforce an order of operations; the addition operation occurs before the division; the result of the division is provided to the **floor()** function.  We have also introduced something new - a renaming operation to give our result column a more informative name. The **AS** keyword lets us rename a column in the output of our query.  We will learn more about using **AS** in `Chapter 4`_.

See Appendix B, `Mathematical operators and functions <../../appendix-b-reference/reference.html#mathematical-operators-and-functions>`_ for a complete list of standard operators and functions.

.. index:: operators; string, functions; string, string concatenation, LIKE


Character string operators and functions
----------------------------------------

SQL provides two very useful string operators, **||** (two vertical bars) and **LIKE**. The operator **||** is used for string concatenation, which has many applications.  For example, if we don't like the columnar output from our **books** table, we could concatenate the columns together (with appropriate spacing or other separators):

.. activecode:: ch3_example_string
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

    SELECT title FROM books WHERE title LIKE '%Earth%';

Now, suppose we are interested in authors who use an initial instead of their full first name.  An initial looks like some character followed by a period - both are required.  Here's what the query would look like, using both the **%** and **_** operators:

::

    SELECT name FROM authors WHERE name LIKE '_.%';

The **LIKE** operator can also be combined with two very useful functions, **upper** and **lower**; these functions put strings in all uppercase or lowercase, respectively.  These functions do not make sense in all language settings, of course.  You can use **upper** or **lower** whenever you want to get back strings in all uppercase or lowercase; you can also use them when pattern matching if you aren't sure of the capitalization of the strings in your database:

::

    SELECT * FROM books WHERE lower(title) LIKE '%love%';

In addition to the functions discussed so far, SQL provides functions for various string manipulations tasks, such as substring extraction or replacement, finding the location of a substring, trimming whitespace (or other characters) from the front and/or back of a string, and many more.  See Appendix B, `Character string operators and functions <../../appendix-b-reference/reference.html#character-string-operators-and-functions>`_ for these.

.. index:: operators; Boolean, AND, OR, NOT

Boolean operators
-----------------

As discussed in `Chapter 2`_, the **WHERE** clause of a **SELECT** query expects a Boolean expression after the **WHERE** keyword.  Some expressions that are Boolean in SQL include expressions using comparison operators, or an expression using the **LIKE** operator.  Many functions also result in a Boolean value.

SQL provides logical operators that operate on Boolean values.  These operators are **AND**, **OR**, and **NOT**, which perform the logical operations that their names imply.  For example, if we have an expression of the form ``expr1 AND expr2``, the result is ``True`` if and only if both ``expr1`` and ``expr2`` evaluate to ``True``.  Similarly, ``expr1 OR expr2`` evaluates to ``True`` if either ``expr1`` or ``expr2`` are ``True``.  Finally, ``NOT`` inverts the truth value:  ``NOT True`` results in ``False``, and ``NOT False`` results in ``True``.

These logical operators allow us to build up complex Boolean expressions from simpler Boolean expressions to express the particular logical conditions we want for our **WHERE** clause.  So, for example, we might be interested in fantasy books published since the year 2000:

.. activecode:: ch3_example_boolean
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
    WHERE genre = 'science fiction' OR genre = 'fantasy'
    AND publication_year > 2000;

    SELECT *
    FROM books
    WHERE
        (genre = 'science fiction' OR genre = 'fantasy')
        AND publication_year > 2000;

The first query above returns any science fiction books, and fantasy books published after 2000.  The second returns the desired result: books published after 2000 in either the fantasy or science fiction genres.

For a fuller discussion of Boolean operators, we need to know more about ``NULL`` values, which will be discussed below.  See Appendix B, `Boolean operators <../../appendix-b-reference/reference.html#boolean-operators>`_ for complete documentation on the SQL Boolean operators.

.. index:: operators; date and time, functions; date and time

Date and time operators and functions
-------------------------------------

Date and time data are extremely important in many database applications, such as those supporting governmental or financial institutions.  SQL provides extensive functionality for managing dates and times.  Unfortunately, this is an area where different SQL implementations vary widely in their conformance to the SQL standard. See Appendix B, `Date and time operators and functions <../../appendix-b-reference/reference.html#date-and-time-operators-and-functions>`_ for a fuller discussion, and consult your database implementation's documentation to see what capabilities it offers with respect to date and time handling.

One useful SQL function that most databases implement is the **CURRENT_DATE** function (also try **CURRENT_TIME** and **CURRENT_TIMESTAMP**):

.. activecode:: ch3_example_datetime
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT CURRENT_DATE;

We will see in `Chapter 5`_ how this function can be used to automatically record the date in a newly created row.


.. index:: NULL, three value logic

NULL
::::

In many database applications, it is sometimes necessary to provide no information on some aspect of piece of data.  For example, in querying our **authors** table, we can see that some entries in the **death** column are blank.  This probably means that the author for that row had not died at the time the data was entered, and thus the column was simply not applicable for that author; there is no death date for the author.  In contrast, some **birth** dates are blank; in this case, the column certainly applies to the author - they were clearly born at some point!  However, that information was unknown to the person entering the data into the table, so nothing was entered.

These notions, of data that are *not applicable* or *unknown* are captured with a special value in SQL:  ``NULL`` [#]_.  ``NULL`` values represent the *absence of information*.  In the **authors** table, the blanks in our result do not indicate that empty strings are in the database.  Instead, ``NULL`` values stand in for the missing information.  Unfortunately, ``NULL`` does not tell us the *reason* the data is missing - whether it is not applicable or simply unknown.  If this distinction is important for your database, you will need to use extra columns to indicate the meaning of the ``NULL``, or use some value other than ``NULL``.

Because ``NULL`` is truly an absence of information, ``NULL`` values used in expressions usually result in ``NULL`` when the expression is evaluated.  For example, what is the result of ``2 + NULL``?  We simply cannot know - the ``NULL`` isn't telling us anything, so the result is unknown, or ``NULL``.

A very important consequence of this behavior is that ``NULL`` values cannot be usefully compared with anything, even other ``NULL`` values!  That is, an expression like ``x = NULL`` is never ``True`` even if *x* itself contains ``NULL``.  This might seem counterintuitive, but if you think of the expression ``NULL = NULL`` as asking the question, "Is this unknown thing the same as this other unknown thing?", you can see that the answer should be "unknown", or ``NULL``.

To find out if a value is ``NULL`` or not ``NULL`` requires special operators: **IS NULL** and **IS NOT NULL**.  For example, if we want to discover authors for whom we have no death date, we would do the query:

.. activecode:: ch3_example_null
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * FROM authors WHERE death IS NULL;

You can discover authors for whom we do have death dates by replacing **IS NULL** with **IS NOT NULL** in the above query.

What happens if we instead do

::

    SELECT * FROM authors WHERE death = NULL;

?

In this case, the expression ``death = NULL`` will evaluate to ``NULL`` for every row in the table.  The **WHERE** clause will filter these out, because it only accepts expressions that evaluate to ``True``, and ``NULL`` is not the same as ``True``.

``NULL`` values can sometimes lead us astray.  Consider the question of finding all authors who were alive in the year 2000 or later.  It might be tempting to write a query such as

::

    SELECT * FROM authors
    WHERE birth <= '2000-12-31'
    AND death >= '2000-01-01';

This is a perfectly valid query - dates in this standard format can be compared in this fashion in our database.  However, if you do the query, you'll see that all of our living authors are not in the result.  This happened, again, because the **death** column in those rows contained ``NULL`` values; comparing these to ``'2000-01-01'`` also yielded ``NULL``, and the **WHERE** clause therefore filtered them out.

In this case, we need to use more logic, and query the database thus:

::

    SELECT * FROM authors
    WHERE birth <= '2000-12-31' AND
        (death >= '2000-01-01' OR death IS NULL);

This works correctly, but you might be wondering why.  We said that ``NULL`` used in expressions usually results in ``NULL``; here, we have a compound Boolean expression using the operators **AND** and **OR**.  So shouldn't we again lose all living authors?  Well, no: Boolean operators are an exception.  This is because, used in Boolean expressions, ``NULL`` means that we simply cannot know if the value is ``True`` or ``False``; the value is unknown.  However, we can still evaluate an expression like ``True OR NULL`` to be ``True``, because ``True OR True`` is ``True``, and so is ``True OR False`` in Boolean logic.  Either way, we get ``True``, so not knowing which it might be doesn't matter.  So the expression in the parentheses is ``True`` if either one of the two conditions within it is true.

On the other hand, ``False OR NULL`` will gives us ``NULL``.  In this case, whether the ``NULL`` is standing in for ``True`` or ``False`` actually matters, because each gives a different outcome. Since we do not know the outcome, the result is ``NULL``.

Because Boolean expressions can result in ``True``, ``False``, or ``NULL``, we say that SQL has a *three-valued logic* (not truly Boolean logic).  Appendix B, `Boolean operators <../../appendix-b-reference/reference.html#boolean-operators>`_ provides truth tables for this three-valued logic, but as shown above, you can usually work out the answer by simply thinking of ``NULL`` as meaning "unknown".


.. index:: conditional expressions

Conditional expressions
:::::::::::::::::::::::

SQL provides expressions for doing simple conditional logic.  The basic conditional expression in SQL is the **CASE** expression, which comes in two forms.  In the most general form, **CASE** lets you specify what the expression should evaluate to depending on a list of conditions.  The effect is similar to using if/else or case statements in some programming languages.

The basic form of the **CASE** expression is

::

    CASE WHEN condition1 THEN result1
         [WHEN condition2 THEN result2]
         ...
         [ELSE result]
    END

The **CASE** keyword comes first, followed by one or more **WHEN** clauses giving a condition and the result if the condition is true.  The first true condition determines the result.  If none of the conditions evaluate to ``True``, then the **ELSE** result is used, if provided, or ``NULL`` if there is no **ELSE** clause.  The expression is finished with the **END** keyword.

For example, we could put our books into different categories, maybe for different sections in a library, using **CASE**:

.. activecode:: ch3_example_case
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT
        author, title,
        CASE WHEN genre = 'fantasy' THEN 'speculative fiction'
             WHEN genre = 'science fiction' THEN 'speculative fiction'
             WHEN genre = 'poetry' THEN 'poetry'
             WHEN genre = 'history' THEN 'non-fiction'
             ELSE 'general fiction'
        END
        AS category
    FROM books;

Here we have included tests for some genres not present in our current dataset.  A library application might have many categories, each encompassing multiple genres.  Using a **CASE** expression would be one way to output books with their categories, although it depends on knowledge of all the possible genres in our database.  A more data-driven way would be to look up categories in another database table using a *join*, a technique we will discuss in `Chapter 4`_.

Another form of **CASE** matches an expression to possible values.  The above query can be re-written using this form:

::

    SELECT
        author, title,
        CASE genre
            WHEN 'fantasy' THEN 'speculative fiction'
            WHEN 'science fiction' THEN 'speculative fiction'
            WHEN 'poetry' THEN 'poetry'
            WHEN 'history' THEN 'non-fiction'
            ELSE 'general fiction'
        END
        AS category
    FROM books;

There are two functions that perform specialized conditional logic.  The **COALESCE** function takes a variable number of arguments.  The result of the function is the first non-``NULL`` expression in the argument list, or ``NULL`` if all arguments are ``NULL``.  This can be useful for replacing ``NULL`` values with more descriptive values:

::

    SELECT name, COALESCE('died: ' || death, 'living')
    FROM authors;

Finally, the **NULLIF** function takes two arguments; if the arguments are equal, the function results in ``NULL``, otherwise it results in the first argument.  This can be used to replace specific values with ``NULL``.  For example,

::

    SELECT title, author, NULLIF(genre, 'science fiction')
    FROM books;



Self-check exercises
::::::::::::::::::::

This section contains some exercises using the same books and authors database used in the text above.  If you get stuck, click on the "Show answer" button below the exercise to see a correct answer.

.. activecode:: ch3_self_test_comparison
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find all books published from the year 1980 through the year 2000, in order by publication year:
    ~~~~

.. reveal:: ch3_self_test_comparison_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    There are usually many ways to achieve the same goal in SQL.  Here are a couple of solutions:

    ::

        SELECT * FROM books
        WHERE publication_year >= 1980 AND publication_year <= 2000
        ORDER BY publication_year;

        SELECT * FROM books
        WHERE publication_year BETWEEN 1980 AND 2000
        ORDER BY publication_year;


.. activecode:: ch3_self_test_pattern
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find the authors whose name starts with the letter "J":
    ~~~~

.. reveal:: ch3_self_test_pattern_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM authors WHERE name LIKE 'J%';


.. activecode:: ch3_self_test_boolean1
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find books written between 1950 and 1999, excluding poetry:
    ~~~~

.. reveal:: ch3_self_test_boolean1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM books
        WHERE publication_year >= 1950 AND publication_year <= 1999
        AND genre <> 'poetry';


.. activecode:: ch3_self_test_boolean2
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find books written before 1950 or after 1999, excluding science fiction:
    ~~~~

.. reveal:: ch3_self_test_boolean2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM books
        WHERE (publication_year < 1950 OR publication_year > 1999)
        AND genre <> 'science fiction';


.. activecode:: ch3_self_test_boolean3
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find books with a title beginning with the letters "T" or "I", in the fiction, fantasy, or poetry genres:
    ~~~~

.. reveal:: ch3_self_test_boolean3_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT * FROM books
        WHERE (title LIKE 'T%' OR title LIKE 'I%')
        AND (genre = 'fiction' OR genre = 'fantasy' OR genre = 'poetry');


.. activecode:: ch3_self_test_null1
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find authors for whom we have no birth date:
    ~~~~

.. reveal:: ch3_self_test_null1_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM authors WHERE birth IS NULL;


.. activecode:: ch3_self_test_null2
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query to find deceased authors born after 1915:
    ~~~~

.. reveal:: ch3_self_test_null2_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT name FROM authors
        WHERE death IS NOT NULL
        AND birth > '1915-12-31';


.. activecode:: ch3_self_test_conditional
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    Write a query giving book titles and authors together with the century in which they were written, spelled out like 'Twentieth Century' (you only need to worry about the 20th - 21st centuries):
    ~~~~

.. reveal:: ch3_self_test_conditional_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        SELECT title, author,
            CASE WHEN publication_year >= 1900 AND publication_year < 2000 THEN 'Twentieth Century'
                 WHEN publication_year >= 2000 AND publication_year < 2100 THEN 'Twenty-first Century'
            END
            AS century
        FROM books;


.. [#] Database scholars frequently reject calling ``NULL`` a *value*.  If ``NULL`` were truly a value, then it should be comparable to itself and other values.  One alternative is to say that a column is in a ``NULL`` *state*, rather than that it contains a ``NULL`` value.  However, this distinction breaks down in other SQL settings, such as grouping and aggregation (`Chapter 8`_).  Because of this and other concerns, the inclusion of ``NULL`` in SQL is controversial.
