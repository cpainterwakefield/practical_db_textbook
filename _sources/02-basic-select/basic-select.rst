====================
Basic SELECT queries
====================

From this chapter, you will learn the basic structure of a relational database and the basics of retrieving data using SQL.

Relational database foundations
:::::::::::::::::::::::::::::::

While relational databases can contain many types of *objects*, the basic object type that stores data is the *table*.  Each table in the database has a name, which usually provides some indication of what kind of data can be found in the table.  The table structure is defined by the table's *columns*, each of which has a name and an associated data type.  The actual data is contained in the *rows* of the table; each row is one data point and has a value for each column of the table.

We can visualize a simple table as, well, a table:

.. image:: table_illustration.svg

The illustration above shows a table named "fruit_stand" with three columns named "item", "price", and "unit".  Although the illustration does not show the data types, we might infer that the **item** and **unit** columns contain text and the **price** column contains decimal numbers.  Each row of **fruit_stand** contains information about one kind of fruit sold at the fruit stand.

Some notes on terms:

- *Tables* are some times called *relations*, which is their name in the *relational theory* (see chapter XXX) that relational databases are based on.
- The definition of a table's structure is some times called it's *schema* (although that is a word that is heavily overloaded to mean different things in different contexts, so beware).
- *Columns* are also known as *attributes*, particularly in the context of the table's specification and in relational theory.

Example dataset for this book
:::::::::::::::::::::::::::::



.. activecode:: ch2_example1_select
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * FROM books;
    
- example dataset?
- displaying table information --> select name, sql from sqlite_master
- conventions
- choosing columns
- filtering rows (WHERE)
- ordering data (ORDER BY)
- basic data types
- NULL
    - meaning of
    - behavior in expressions
- basic expressions
    - use a few basic operators for example
    - using expressions in SELECT
    - using expressions in WHERE
    - using expressions elsewhere (e.g., ORDER BY)
    - literal expressions
- useful functions and operators
    - Boolean operators
    - Math functions and operators
    - String functions and operators 
    - Date functions and operators
    - Miscellaneous
- DISTINCT

For some of these, need a table showing database implementation?  Or just SQL standard... maybe move table to appendix...
