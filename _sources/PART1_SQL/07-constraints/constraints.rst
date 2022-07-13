.. _constraints-chapter:

====================
Keys and constraints
====================

In this chapter, we explore the different ways in which data may be constrained in order to help preserve the correctness of our data.

Tables used in this chapter
:::::::::::::::::::::::::::

Some examples in this chapter will use the **books** table and related tables (see :ref:`Appendix A <appendix-a>` for a full description of these tables).  Other examples will use tables created just for the example, then discarded.

Constraints
:::::::::::

As data is critical to so many organizations and applications, it is worth doing everything possible to ensure the correctness and consistency of the data in databases.  This can be achieved in many ways: application software can do checks to ensure data is entered and maintained correctly, separate programs can test and report on data consistency, and databases can enforce correctness using constraints and triggers.  (Triggers are actions the database can perform when certain events, such as an update to a row, occur; in some databases, the actions can even be complete small programs using implementation-specific programming languages.  This book does not cover triggers.)  We focus in this chapter on database constraints.

Constraints are properties of the database that restrict data in various ways.  One of the simplest types of constraints are *domain* constraints: data entered into a column is constrained to be of the type defined for the column.  (Domain constraints are not enforced in SQLite, so we cannot demonstrate using the book's database.)  Below we discuss constraints that enforce data properties for entire tables - primary and foreign key constraints - and those that apply to individual rows: not null, uniqueness, and check constraints.

Primary keys
::::::::::::

A *primary key* is a column or set of columns of a table which are required to contain unique values for each row stored in the table.  Controlling for uniqueness ensures that we can associate a specific, single item of data with data in another table by storing the key value in the other table. It also prevents unnecessary duplication of data that could result in incorrect statistics (for example, when taking counts or sums).  The id columns discussed in :numref:`Chapter {number} <joins-chapter>` - special columns used specifically to ensure each row can be uniquely identified - are commonly used as primary keys.  A table can have only one primary key.

Some examples can be found in our books database.  The table **authors**, for example, has **author_id** as a primary key.  The database will enforce the uniqueness property of the **author_id** column by preventing any data modification queries which would result in a duplicate primary key.  Deleting rows can never violate a primary key constraint, but inserts and updates can.  Either of the following queries will result in an error because there already exists a row in the database with **author_id** = 1:

.. activecode:: constraints_example_primary_key
    :language: sql
    :dburl: /_static/textbook.sqlite3

    INSERT INTO authors (author_id, name)
    VALUES (1, 'Charles Dickens');

    UPDATE authors SET author_id = 1 WHERE author_id = 2;

Because we need to compare primary key values to ensure uniqueness, it is also forbidden to insert ``NULL`` values into a primary key column.  (SQLite varies from this standard behavior, though. If you insert a ``NULL`` into an integer primary key column, SQLite will generate a unique value for you.  For other primary key column types, or multi-column primary keys, ``NULL`` values are permitted in SQLite.)

An example of a multi-column primary key can be found in the table **authors_awards**.  For this table, the primary key is the pair (**author_id**, **award_id**).  This does not mean that each column is a primary key, e.g., that each column is independently unique.  Instead, it is the pair of values which must be unique.  That is, you may not insert the pair ``(4, 10)`` twice, but you may insert all of ``(4, 10)``, ``(4, 11)``, and ``(5, 10)`` (assuming none of those pairs is already in the table).

Creating a primary key
-----------------------

There are a few ways to create a primary key on a table.  It is easiest to create a primary key when creating the table.  If the primary key is a single column, then you may simply put the phrase **PRIMARY KEY** at the end of the column definition:

::

      DROP TABLE IF EXISTS test;
      CREATE TABLE test (
        x VARCHAR(10) PRIMARY KEY
      );

You can also create the primary key as a separate entry in the table definition; you must use this form when creating a multi-column primary key.  The first definition below is equivalent to the one above, while the second creates a multi-column primary key:

::

    DROP TABLE IF EXISTS test2;
    DROP TABLE IF EXISTS test3;

    CREATE TABLE test2 (
      x VARCHAR(10),
      PRIMARY KEY (x)
    );

    CREATE TABLE test3 (
      x INTEGER,
      y INTEGER,
      PRIMARY KEY (x, y)
    );

Some database systems also allow the addition of a primary key to an existing table using the **ALTER TABLE** command, as long as the existing data would conform to the constraint.  **ALTER TABLE** can also be used to remove (drop) constraints from a table.  (SQLite does not support this usage.)

Foreign keys
::::::::::::

We are also interested in the relationships between data in different tables.  For example, every row in the **books** table has an **author_id** value which lets us look up author information in the **authors** table.  How can we ensure that the **author_id** value is valid, that is, that it always has a corresponding row in the **authors** table?  For a relational database, the solution is a *foreign key* constraint.

A foreign key constraint applies to a column or list of columns in one table (the *referencing* table), and references a column or list of columns in another table (the *referenced* table).  The constraint requires that one of two things be true:

- The values in the column or columns in the referencing table exist in the referenced column or columns
- The values in the column or columns in the referencing table are ``NULL``

The column or columns in the referenced table must be constrained to be unique, either by making them the primary key (the usual case), or through a uniqueness constraint (see below).

Our database defines a foreign key between **books** and **authors**.  The foreign key constraint is on the **author_id** column of **books** and references the **author_id** column of **authors**.  If we want to add a new book to **books**, we must add it with an **author_id** value, which must be a valid **author_id** from the **authors** table.  The foreign key by itself would allow a ``NULL`` **author_id** value, but we have further constrained the **author_id** column to not be null (using the **NOT NULL** constraint defined later on).

For example, the code below will fail due to the foreign key constraint on **books** referencing **authors**.  (Note that, unlike other database systems, SQLite will not enforce foreign keys unless you specifically tell it to - that is what the line of code below starting with the keyword **PRAGMA** is doing.  The **PRAGMA** keyword is not part of the SQL standard, and is only needed in SQLite.)

.. activecode:: constraints_example_foreign_key
    :language: sql
    :dburl: /_static/textbook.sqlite3

    PRAGMA foreign_keys = ON;

    INSERT INTO books (author_id, title)
    VALUES (99, 'Unknown');   -- 99 is not a valid author id

We also cannot do operations which would destroy existing relationships.  For example, there exist records in the **books** table for which the **author_id** = 1.  If we were to delete the author with this id value, we would violate the foreign key constraint on the **books** table.  We would also violate a similar foreign key constraint on **authors_awards**.  This code therefore produces an error:

::

    PRAGMA foreign_keys = ON;

    DELETE FROM authors WHERE author_id = 1;

However, if we first remove all books and awards for this author, we can successfully remove the author:

::

    PRAGMA foreign_keys = ON;

    DELETE FROM books WHERE author_id = 1;
    DELETE FROM authors_awards WHERE author_id = 1;
    DELETE FROM authors WHERE author_id = 1;

We similarly cannot drop the **authors** table without first dropping any referencing tables.

The **books** and **authors** examples demonstrate a common pattern, which is that foreign key constraints relate columns that we are likely to want to use in a join query (:numref:`Chapter {number} <joins-chapter>`).  This does not mean that a foreign key is a necessary condition for a join; one of the strengths of a relational database is that relationships between data do not have to be predetermined.  However, the presence of a foreign key is an indication that there exists a natural relationship between the data in the referencing and referenced tables.

Foreign key constraints are also known as *referential integrity constraints*.

Creating a foreign key constraint
---------------------------------

As with primary keys, there are multiple ways to create a foreign key constraint.  If the foreign key constrains a single column, then we can add it in the column definition for a table using the **REFERENCES** keyword:

::

    DROP TABLE IF EXISTS referencing;
    DROP TABLE IF EXISTS referenced;

    CREATE TABLE referenced (
      x INTEGER PRIMARY KEY
    );

    CREATE TABLE referencing (
      xx INTEGER REFERENCES referenced (x)
    );

Note that although the foreign key constraint only appears in the referencing table definition, the constraint affects both tables.  The code above ensures that values in the **xx** column of **referencing** are either ``NULL`` or contained in the **x** column of **referenced**.

We can also create a foreign key constraint with a separate **FOREIGN KEY** entry in the table definition.  This form must be used for multi-column foreign keys:

::

    DROP TABLE IF EXISTS referencing2;
    DROP TABLE IF EXISTS referenced2;

    CREATE TABLE referenced2 (
      a VARCHAR(10),
      b VARCHAR(20),
      PRIMARY KEY (a, b)
    );

    CREATE TABLE referencing2 (
      c INTEGER PRIMARY KEY,
      aa VARCHAR(10),
      bb VARCHAR(10),
      FOREIGN KEY (aa, bb) REFERENCES referenced2 (a, b)
    );

In the above example, the *pair* (**aa**, **bb**) in **referencing2** must match a corresponding (**a**, **b**) pair in **referenced2**; the columns are not constrained independently.

Note that it is possible (and sometimes useful) to create a foreign key constraint in which the referencing and referenced tables are the same table.  For example, a company might have a table of employees that references itself:

::

    CREATE TABLE employees (
      id INTEGER PRIMARY KEY,
      name VARCHAR(100),
      supervisor_id INTEGER REFERENCES employees (id)
    );

As with primary keys, some database systems allow the addition or removal of foreign key constraints using the **ALTER TABLE** command.  (This usage is not supported by SQLite.)


Enforcement mechanisms
----------------------

The default behavior for a foreign key constraint is to reject any attempt to modify data in a way that would violate the constraint.  However, SQL provides additional options that can be applied for **DELETE** or **UPDATE** queries.  Adding the phrase **ON DELETE SET NULL** to the foreign key constraint indicates that a deletion of a referenced table row should result in setting corresponding referencing key values to ``NULL`` (if permitted).  The phrase **ON DELETE CASCADE** indicates that referencing rows should be deleted along with the referenced row.  Similarly, **ON UPDATE SET NULL** results in setting referencing key values to ``NULL`` if the referenced key value is changed; **ON UPDATE CASCADE** changes the referencing key values to match the changed referenced key value.  Finally, if you want to use the default behavior explicitly, you can use **ON DELETE RESTRICT** and **ON UPDATE RESTRICT**.

Here is an example to try, using **CASCADE** for both deletions and updates (modify to try different settings):

::

    PRAGMA foreign_keys = ON;

    DROP TABLE IF EXISTS works;
    DROP TABLE IF EXISTS composers;

    CREATE TABLE composers (
      id INTEGER PRIMARY KEY,
      name VARCHAR(30)
    );

    INSERT INTO composers VALUES
      (1, 'Beethoven'),
      (2, 'Mozart')
    ;

    CREATE TABLE works (
      title VARCHAR(50),
      composer_id INTEGER REFERENCES composers (id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
    );

    INSERT INTO works VALUES
      ('Symphony No. 1', 1),
      ('Symphony No. 2', 1),
      ('String Quartet No. 1', 2)
    ;

    DELETE FROM composers WHERE name = 'Beethoven';

    UPDATE composers SET id = 4 WHERE name = 'Mozart';

    SELECT * FROM composers;
    SELECT * FROM works;


Other constraints
:::::::::::::::::

SQL provides some additional constraints you may find useful, which are described in this section.

UNIQUE
------

Occasionally you may need to ensure that a column or set of columns contains unique values, but you do not want to set the column or columns as a primary key (for example, when some other set of columns is already the primary key).  The **UNIQUE** constraint can be used for this purpose; simply add the **UNIQUE** keyword as part of the column definition.  One difference between a **UNIQUE** constraint and a primary key constraint is that the **UNIQUE** constraint does not prevent ``NULL`` values. However, databases deal with ``NULL`` values in a unique column in different ways; some allow multiple rows to contain ``NULL``, and others allow only a single ``NULL`` row (effectively treating ``NULL`` as a comparable value).  Note that the final statement below will fail due to a violation of the **UNIQUE** constraint on column **x**.

.. activecode:: constraints_example_other
    :language: sql
    :dburl: /_static/textbook.sqlite3

    DROP TABLE IF EXISTS test4;
    CREATE TABLE test4 (
      x INTEGER UNIQUE
    );

    INSERT INTO test4 VALUES (1);
    INSERT INTO test4 VALUES (2);
    INSERT INTO test4 VALUES (1);

You can also create a **UNIQUE** constraint as a separate entry in the table definition (this is required for a multi-column constraint):

::

      DROP TABLE IF EXISTS test5;
      CREATE TABLE test5 (
        x INTEGER,
        y INTEGER,
        UNIQUE (x, y)
      );

Note that a primary key constraint on a set of columns already implies something stronger than **UNIQUE**, thus there is no need to specify **UNIQUE** if **PRIMARY KEY** is already in place.

NOT NULL
--------

``NULL`` values can be a source of many data errors.  If some bug in your software incorrectly inserts ``NULL`` values into your database, the data becomes corrupt, and queries against the data may produce wrong answers. Also, since ``NULL`` values are not comparable, they tend to be "invisible" when querying, unless looked for specifically using **IS NULL**.   This combination of factors can result in many lost hours of work trying to resolve differences between what you believe is true and what your queries are telling you.

It can be valuable, then, to constrain columns to not allow ``NULL`` values at all, using the **NOT NULL** constraint.  In our database, one example is the **authors** table, which has a **NOT NULL** constraint on the **name** column - we always want a value in the **name** column [#]_.  You can find other examples in the books-related tables.

Note that **NOT NULL** is implied on all columns in a primary key, so there is no need to specify **NOT NULL** for those columns.

CHECK
-----

SQL provides a general constraint form that you can use to apply simple Boolean conditions on your data.  The **CHECK** constraint can be an expression involving a single column or multiple columns of the table.  This expression is typically limited in what else it can incorporate; for example subqueries are typically not allowed, and some databases do not allow function calls (implementations vary).

Here is an example, showing both single column and table constraint forms:

::

    DROP TABLE IF EXISTS test6;
    CREATE TABLE test6 (
      a INTEGER CHECK (a BETWEEN 0 AND 100),
      b INTEGER,
      CHECK (b > a)
    );

    INSERT INTO test6 VALUES (42, 200);
    INSERT INTO test6 VALUES (-1, 6);   -- error
    INSERT INTO test6 VALUES (10, 5);   -- error


..
  Behind the scenes
  :::::::::::::::::

  While **NOT NULL** and **CHECK** constraints affect single rows of data and can easily be verified when an **INSERT** or **UPDATE** is performed, key constraints and **UNIQUE** constraints require checks against entire tables.  In these cases, the constraint test fails (primary keys and **UNIQUE**) or succeeds (foreign keys) if a set of values matches some row in some table.  You might wonder how these checks can be performed efficiently in situations where the table to be tested contains very many rows.

  A full answer will have to wait until chapter XXX, but the short answer is that the data values we need to search to test our constraint are *indexed* - stored in a special data structure that allows us to find a particular value very fast, without having to examine every row.  Primary keys and columns constrained to be unique are automatically indexed by the database.  Using the index, the database can quickly detect if a duplicate value is about to be created.  To test a foreign key constraint, the database has to determine whether the data we are putting into the referencing table exists in the referenced table.  Since the referenced columns must either form a primary key or be constrained with **UNIQUE**, the data to be searched is indexed once again.

  Indexes are also very important in speeding up queries and statements of all kinds.  We can add additional indexes to the ones implied by our constraints in order to speed up specific searches or modifications.  We will discuss how to use indexes to improve the performance of queries and statements in chapter XXX.


|chapter-end|

----

**Notes**

.. [#] It is possible that we might wish to record some book for whom the author is unknown (or anonymous), which might seem like an instance in which we would want ``NULL``; after all, one possible meaning of ``NULL`` is "unknown".  However, what does it mean for an unknown author to have an entry in the **authors** table in the first place?   What meaning would we give, if any, to the birth and death date fields for the ``NULL`` author?  And what does it mean if multiple books relate to that author record?  Are they all by the same, unknown author, or by different authors, both of whom are unknown?  A slightly better choice for a book with no known author may be to allow ``NULL`` values in the **author_id** column in **books** - this is closer to the desired meaning.  However, this introduces problems of its own, such as the fact that an inner join of **books** and **authors** will now leave out any books with unknown authors, so we would need to be very careful in writing our queries.  None of this is to say that ``NULL`` is never the right choice, only that it introduces complexity and therefore more opportunity for software bugs and data corruption.  Consider your options carefully.


|license-notice|
