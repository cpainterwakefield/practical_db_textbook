.. _constraints-chapter:

====================
Keys and constraints
====================

In this chapter, we explore the different ways in which data may be constrained in order to help preserve the correctness of our data.

Tables used in this chapter
:::::::::::::::::::::::::::

Constraints
:::::::::::

As critical as data is to so many organizations and applications, it is worthwhile doing everything possible to ensure the correctness and consistency of the data in databases.  This can be achieved in many ways: application software can do checks to ensure data is entered and maintained correctly, separate programs can test and report on data consistency, and databases can enforce correctness using constraints and triggers.  (Triggers are actions the database can perform when certain events, such as an update to a row, occur; in some databases, the actions can be complete small programs using implementation-specific programming languages.  This book does not cover triggers.)

Constraints are properties of the database that restrict data in various ways.  One of the simplest types of constraints are *domain* constraints: data entered into a column is constrained to be of the type defined for the column.  (Domain constraints are not enforced in SQLite, so we cannot demonstrate using the book's database.)  Below we discuss constraints that enforce data properties for entire tables (primary and foreign key constraints) and those that apply to individual rows (not null, uniqueness, and check constraints).

Primary keys
::::::::::::

A *primary key* is a column or set of columns of a table which are required to contain unique values for each row stored in the table.  A common example of this are the id columns discussed in :numref:`Chapter {number} <joins-chapter>` - special columns used specifically to ensure each row can be uniquely identified.  Controlling for uniqueness ensures that we can associate a specific, single datum with data in another table by storing the key value in the other table; it also prevents unnecessary duplication of data that could result in incorrect statistics like row counts.  A table can have only one primary key.

Some examples can be found in our books database.  The table **authors**, for example, has **author_id** as a primary key.  The database will enforce the uniqueness property of the **author_id** column by preventing any data modification queries which would result in a duplicate primary key.  Deleting rows can never violate a primary key constraint, but inserts and updates can.  Either of the following queries will result in an error because there exists a row in the database with **author_id** = 1:

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

      CREATE TABLE test (
        x VARCHAR(10) PRIMARY KEY
      );

You can also create the primary key as a separate entry in the table definition; you must use this form when creating a multi-column primary key.  The first definition below is equivalent to the one above, while the second shows a multi-column primary key:

::

    CREATE TABLE test2 (
      x VARCHAR(10),
      PRIMARY KEY (x)
    );

    CREATE TABLE test3 (
      x INTEGER,
      y INTEGER,
      PRIMARY KEY (x, y)
    );

Some database systems also allow the addition of a primary key to an existing table using the **ALTER TABLE** command.  (This form is not supported by SQLite.)

Foreign keys
::::::::::::

While primary keys are constraints on a single table, a *foreign key* is a constraint that relates two tables.  A foreign key constraint applies to a column or list of columns in one table, and *references* a column or list of columns in another table.  The constraint requires that one of two things be true:

- The values in the column or columns in the first table exist in the referenced column or columns
- The values in the column or columns in the first table are ``NULL``

The column or columns in the referenced table must also be constrained to be unique, either by making them the primary key (the usual case), or through a uniqueness constraint (see below).

For example, our database defines a foreign key between **books** and **authors**.  The foreign key constraint is on the **author_id** column of **books** and references the **author_id** column of **authors**.  If we want to add a new book to **books**, we must add it with an **author_id** value which is a valid **author_id** from the **authors** table.  The foreign key by itself would allow a ``NULL`` **author_id** value, instead, but we have further constrained the **author_id** column to not be null (using the **NOT NULL** constraint defined below).

For example, the code below will fail due to the foreign key constraint on **books** referencing **authors**.  (Note that, unlike other database systems, SQLite will not enforce foreign keys unless you specifically tell it to - that is what the first line of code below is doing.)

.. activecode:: constraints_example_foreign_key
    :language: sql
    :dburl: /_static/textbook.sqlite3

    PRAGMA foreign_keys = ON;

    INSERT INTO books (author_id, title)
    VALUES (99, 'Unknown');   -- 99 is not a valid author id

We also cannot do operations which would destroy existing relationships.  For example, there exist records in the **books** table for which the **author_id** = 1.  If we were to delete the author with this id value, we would violate the foreign key constraint on the **books** table.  We would also violate a similar foreign key constraint on **authors_awards**.  This code also produces an error:

::

    PRAGMA foreign_keys = ON;

    DELETE FROM authors WHERE author_id = 1;

However, if we first remove all books and awards for this author, we can successfully remove the author:

::

    PRAGMA foreign_keys = ON;

    DELETE FROM books WHERE author_id = 1;
    DELETE FROM authors_awards WHERE author_id = 1;
    DELETE FROM authors WHERE author_id = 1;

Foreign key constraints are also known as *referential integrity constraints*.

Other constraints
:::::::::::::::::

- not null
- unique
- check
