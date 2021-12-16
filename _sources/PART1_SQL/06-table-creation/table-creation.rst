.. _table-creation-chapter:

=============================
Data types and table creation
=============================

This chapter will discuss basic table creation, starting with an explanation of SQL data types.

Data types
::::::::::

The SQL standard defines several basic data types with which columns can be associated.  While most of the basic types exist in all relational database systems, actual implementation of the types varies quite a bit.  Most database systems define additional, non-standard types for various uses.  Unusually, SQLite is dynamically typed, and you can store values of any type in any column no matter how the column is defined.  For all of these reasons, you will want to consult your database system's documentation to understand the types available to you.  In this section we survey the major data types, without discussion of database compatibility; for more information, see Appendix B, :ref:`appendix-b-data-types`.

Numbers
-------

SQL provides support for several different types of numbers, each with different applications and limitations.  However, actual implementation of the standard varies quite a bit; see Appendix B, :ref:`appendix-b-number-types` for a full discussion of number types.

Integers
########

SQL defines three integer types: **INTEGER**, **SMALLINT**, and **BIGINT**.  Implementations of these types vary, but it is not uncommon for **INTEGER** (often abbreviated as **INT**) to store 32-bit integers, **SMALLINT** 16-bit integers, and **BIGINT** 64-bit integers.  Not all databases recognize all of these types, but **INTEGER** is recognized by all of the databases considered for this book.  Additional integer types may be available for your database system.

Exact decimal numbers
#####################

Decimal number types allow for exact storage of numbers that have digits to the right of the decimal point, e.g., 1234.56789.  These numbers are exact (compare to the floating point types below), and permit exact mathematical operations where possible (addition, subtraction, and multiplication).  The two defined types for SQL are **NUMERIC** and **DECIMAL**, which are synonyms of each other.  These types may be defined with parameters representing *precision* and *scale*, where precision is the number of significant digits that can be stored, and scale is the number of digits following the decimal point.  If the precision is given, but not the scale, the scale defaults to zero.

For example, in most implementations:

- **NUMERIC(3, 2)** defines a type that can store the values between -999.99 and 999.99, with a maximum of 2 digits past the decimal point.
- **NUMERIC(4)** defines a type that can store integers between -9999 and 9999.
- **NUMERIC** defines a type that can exactly store decimal values with implementation-defined precision and scale.

Different implementations behave differently when an attempt is made to store values with more digits than are allowed by the specified precision and scale.  This may result in an error, or (in the case of too many digits to the right of the decimal point), it may result in rounding or truncation of the value.

Decimal number types are particularly important for the storage of monetary data, where exact addition, subtraction, and multiplication is necessary.

Floating point numbers
######################

Floating point number types allow for (possibly inexact) storage of real numbers, similar (or sometimes identical to) the `IEEE 754`_ specification.  The SQL standard defines the types **FLOAT**, **REAL**, and **DOUBLE PRECISION** (often abbreviated **DOUBLE**), but implementation of these types vary.  These types can support extremely large and extremely small numbers, and are most useful in scientific and mathematical applications.

.. _`IEEE 754`: https://en.wikipedia.org/wiki/IEEE_754


Character string types
----------------------

There are several data types for storing character data in SQL; again, actual implementations vary.  See Appendix B, :ref:`appendix-b-string-types` for a full discussion of character string types.

The type **CHARACTER**, usually abbreviated as **CHAR**, is used for fixed-length strings.  The type **CHAR** is followed by parentheses enclosing the length of the string.  All values in a column of type **CHAR(4)**, for example, must contain exactly 4 characters.  In practice, many databases relax the "exactly" part of the definition and allow for shorter strings to be stored, although they may pad the value with trailing space characters.  Attempting to store strings longer than *n* usually results in an error.

**CHARACTER VARYING** is usually abbreviated as **VARCHAR**, and is used for strings of varying length up to some maximum, which must be specified just as with the **CHAR** type.  It is usually an error to attempt to store strings longer than the maximum.  (Note for Oracle users: Oracle strongly recommends using their **VARCHAR2** type rather than **VARCHAR**, although both types are recognized.)

Examples:

- **CHAR(5)** can store the strings ``'apple'``, ``'1 2 3'``, or ``'x    '`` (with four trailing spaces), but not ``'x'`` or ``'this is too long'``.
- **VARCHAR(5)** can store the strings ``'hello'``, ``'a b'`` or ``'y'``, but not ``'also too long'``.

One disadvantage to **VARCHAR** is the need to predict the maximum length of string that you might need to store.  Many databases now implement some type of arbitrary-length character string type, often named **TEXT**.  Some databases impose limitations on this type (such as not allowing its use for indexed columns).  Be sure to read your database implementation's documentation to understand these limitations before using **TEXT**; if you need portability between databases, it may be best to use **VARCHAR** with a generous size allocation.


Date and time types
-------------------

Management of date and time data is a very complicated affair.  Calendars change over time and differ among cultures, time zones vary geographically, and "leap" adjustments to the calendar and clock occur irregularly.  SQL provides very robust date and time types along with operations on these types that allow for very precise storage and management of these values.  However, here again, implementations vary, and you should read your database system's documentation to understand the fine points.  See Appendix B, :ref:`appendix-b-datetime-types` for a full discussion.

There is no standard syntax for date and time literals in SQL.  In most cases, strings in some implementation-defined format(s) are used to represent dates and times.  Internally, the values may be stored as decimal numbers - offsets from some fixed reference.  In this book we will simply use character strings conforming to the `ISO 8601`_ standard.  Using this format, dates can be usefully compared - ``'2001-04-10'`` is correctly less than ``'2014-01-22'`` - which also means we can put data in order by date columns.  Time values can be trickier due to the possible inclusion of time zone, but we will avoid these complications by simply ignoring them (there are no examples of time values in our database).

.. _`ISO 8601`: https://en.wikipedia.org/wiki/ISO_8601


Additional data types
---------------------

Below is a list of some other data types you might encounter or wish to use in a SQL setting.  These are not supported by all database implementations.

- SQL defines a Boolean data type (**BOOLEAN**) which can store the literal values **True** and **False**, however not all databases support this type.
- SQL also defines types designed to hold binary data.  This can sometimes be useful, although binary data such as images or music files take up a great deal of space; it is often preferable to store them externally, and in the database only record the information needed to retrieve the files (e.g., a file path or URL).
- SQL provides for user-defined types; that is, custom data types created by the database user for specific applications.
- Many databases support types not defined in the SQL standard, or defined as optional extensions, such as types for storing and working with JSON and XML documents, geometric objects, geographical or spatial coordinates, arrays, and more.


Types in SQLite
---------------

As mentioned earlier, SQLite (used in the interactive examples in this book) allows the storage of arbitrary types of data into any column; no type checking is performed.  Essentially, a value in SQLite can be ``NULL``, an integer, a floating point number, or a character string.  However, SQLite supports standard SQL syntax for table creation, including specifying data types for columns; this type information can be viewed as a type of hint to the database user as to what kind of data should be stored.  We will consistently use types you might find in other databases, and store data appropriate to the type in our examples.


Creating tables
:::::::::::::::

Once we have chosen the types for our columns, we can create a table using a **CREATE TABLE** statement.  For our first example, we will create something simple just for demonstration purposes.  As discussed in :numref:`Chapter {number} <basics-chapter>`, you do not need to worry that we are changing the database - we are only working with a copy of the database that is created each time you load the textbook into your browser.  You can reload this page anytime you want to start fresh!

Creating a table from scratch
-----------------------------

Use the **CREATE TABLE** command to create a table.  For now, we will create a simple table by defining the columns in the table.  Later, we will add additional details to the table in the form of *constraints* and *defaults*.  The **CREATE TABLE** command looks like this:

::

    CREATE TABLE (
      column1 type1,
      column2 type2,
      ...
    );

Where "column*n*" is the name of a column, and "type*n*" is a data type that your database supports.  Here is some code to try out:

.. activecode:: table_creation_example_create
    :language: sql
    :dburl: /_static/textbook.sqlite3

    CREATE TABLE test (
      id INTEGER,
      x  VARCHAR(20),
      y  DATE,
      z  NUMERIC(10,2)
    );

    INSERT INTO test VALUES
      (0, 'this is a test', '2021-06-14', 1234.56),
      (1, 'apple', '2021-01-01', 10.10)
    ;

    SELECT * FROM test;

All database tools provide some mechanism for seeing the definition of tables in the database.  In SQLite, you can see the definition of tables by querying the special table **sqlite_master**:

::

    SELECT sql FROM sqlite_master WHERE name = 'test';


Dropping tables
---------------

We cannot **CREATE** a table when it already exists, so if you try to run the above example more than once (without reloading this page in your browser), you will get an error message.  We need to remove the object before re-creating it.  Removing an object from the database is called *dropping* the object, and is accomplished with a **DROP** statement:

::

    DROP TABLE test;

This statement will cause an error if you do it when there is no table named **test**, however.  This can be inconvenient, because we might want to drop and recreate the table many times when we are developing a database-modifying program, or *script*, but we may not always know the current state of the database.  Fortunately, most databases implement an extension to **DROP** that lets us remove the table if and only if it exists, without an error if it does not exist:

::

    DROP TABLE IF EXISTS test;

(Note for Oracle users: Oracle does not recognize this syntax.)

Note that dropping a table also destroys all data stored in the table, and this action is irrevocable (there is no "undo" operation [#]_).  This is one reason that database-modifying programs are usually developed and thoroughly tested using a copy of a database, before ever using it on the "real" database.


Creating a table from a query
-----------------------------

From the perspective of SQL, the result of a **SELECT** query is essentially the same thing as a table.  The difference is that the **SELECT** result is not named and exists only temporarily.  SQL provides a way for us to save the result of a query as a named table, with the table columns defined implicitly based on the result columns.  Any **SELECT** query can be used.  Here is an example making a table from our **books** and **authors** tables:

.. activecode:: table_creation_example_create_as_select
    :language: sql
    :dburl: /_static/textbook.sqlite3

    DROP TABLE IF EXISTS recent_books;  -- good to always start with this

    CREATE TABLE recent_books AS
      SELECT
        a.name AS author,
        b.title,
        b.publication_year
      FROM
        authors AS a
        JOIN books AS b ON a.author_id = b.author_id
      WHERE b.publication_year >= 2010
    ;

    SELECT sql FROM sqlite_master WHERE name = 'recent_books';

    SELECT * FROM recent_books;

(Note for SQL server users: SQL server does not support the above syntax.  The equivalent statement in SQL server looks like: ``SELECT ... INTO new_table FROM ... WHERE ...;``.)


Defaults and auto increments
----------------------------

Table columns can be defined with additional properties that can enhance usage of the database in different ways.  In :numref:`Chapter {number} <constraints-chapter>`, we will talk about various *constraints* that can be used to restrict data to help ensure the validity of the database as a whole.  Another property we can add to a column is a *default* expression - an expression producing a value that will be provided by the database only when we do not provide a value.

Here is an example, showing the usage of the **DEFAULT** keyword:

.. activecode:: table_creation_example_default
    :language: sql
    :dburl: /_static/textbook.sqlite3

    DROP TABLE IF EXISTS test2;

    CREATE TABLE test2 (
      id INTEGER,
      greeting VARCHAR(15) DEFAULT 'Hello'
    );

    INSERT INTO test2 (id, greeting) VALUES (1, 'Good morning');
    INSERT INTO test2 (id, greeting) VALUES (2, NULL);
    INSERT INTO test2 (id) VALUES (3);

    SELECT * FROM test2;

As you can see, when we provided a value (or ``NULL``) for the column **greeting**, what we provided was stored.  When we did not provide the value, the default ``'Hello``' was used.

In the simplest case, as above, we can provide a literal value for a column.  More commonly, we will use an expression, typically calling a function of some sort.  A common usage for this is to record the date and time when a record is added to the database.  Here we will use the **CURRENT_TIMESTAMP** function for this purpose:

::

    DROP TABLE IF EXISTS test3;

    CREATE TABLE test3 (
      purchase VARCHAR(10),
      created_at VARCHAR(20) DEFAULT CURRENT_TIMESTAMP
    );

    INSERT INTO test3 (purchase) VALUES ('apple');

    SELECT * FROM test3;

Another common use for default columns is in combination with a special kind of database object called a *sequence*, which simply generates sequential integers.  This can be used, for example, to create unique identifiers for every row in a table.  This usage is so common that the SQL standard provides syntax that both creates the necessary sequence and sets up the default for the table; not all databases support this syntax, but most provide some mechanism for the generation of sequential values for a column.  The SQL standard syntax does not work in SQLite, so you will not be able to test it in an interactive tool in this book, but the column definition using this syntax is

::

    column_name type GENERATED BY DEFAULT AS IDENTITY

or

::

    column_name type GENERATED ALWAYS AS IDENTITY

The first form allows values to be provided by the user when inserting a row, just like a regular value.  The second form requires that the value always be provided by the database - it cannot be overridden by the user.  (Of the databases considered for this book, only PostgreSQL and Oracle support the standard syntax; they also provide equivalent mechanisms using different syntax.  For MySQL, see documentation on the AUTO_INCREMENT property of the INTEGER data type, and for SQL Server, see the IDENTITY option under CREATE TABLE.  For SQLite, see below.)

SQLite provides a mechanism which is similar to, but different from the standard.  In SQLite, we can create an integer column that automatically provides a new value when one is not provided by the user using the **AUTOINCREMENT** keyword; the new value provided is 1 (if the table is empty) or one greater than the maximum value already stored.  To create a column of this type, the column must also be declared to be a primary key, a topic covered in :numref:`Chapter {number} <constraints-chapter>`.  Here is an example to try out:

::

    DROP TABLE IF EXISTS test4;

    CREATE TABLE test4 (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      greeting VARCHAR(15)
    );

    INSERT INTO test4 (greeting) VALUES ('Hello');
    INSERT INTO test4 (id, greeting) VALUES (4, 'Good day');
    INSERT INTO test4 (greeting) VALUES ('Good afternoon');

    SELECT * FROM test4;

In our database, the tables **bookstore_inventory** and **bookstore_sales** use auto increment columns; **bookstore_sales** also uses the **DEFAULT** property.

Self-check exercises
::::::::::::::::::::

This section contains exercises on table creation.  If you get stuck, click on the "Show answer" button below the exercise to see a correct answer.

.. activecode:: table_creation_self_test_create
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a statement to create a table named **my_table** with columns **a**, **b**, **c**, and **d**.  Column **a** will contain strings of at most 100 characters; **b** will contain dates; **c** will contain numbers with at most 15 digits, three of which come after the decimal point; and **d** will contain strings of exactly two characters.
    ~~~~

.. reveal:: table_creation_self_test_create_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        CREATE TABLE my_table (
          a VARCHAR(100),
          b DATE,
          c NUMERIC(15,3),
          d CHAR(2)
        );

.. activecode:: table_creation_self_test_drop
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a statement to remove **my_table**.
    ~~~~

.. reveal:: table_creation_self_test_drop_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        DROP TABLE my_table;

    or

    ::

        DROP TABLE IF EXISTS my_table;

.. activecode:: table_creation_self_test_create_as_select
    :language: sql
    :dburl: /_static/textbook.sqlite3

    Write a statement to create a table named **a_authors** containing just authors whose names start with the letter 'A':
    ~~~~

.. reveal:: table_creation_self_test_create_as_select_hint
    :showtitle: Show answer
    :hidetitle: Hide answer

    ::

        CREATE TABLE a_authors AS
          SELECT * FROM authors
          WHERE name LIKE 'A%'
        ;


|chapter-end|

----

**Notes**

.. [#] Relational databases allow operations to be wrapped in something called a *transaction*, which does provide a way to undo work.  We will study transactions more in chapter XXX.


|license-notice|
