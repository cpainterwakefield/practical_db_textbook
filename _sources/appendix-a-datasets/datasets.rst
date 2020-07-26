==============================================
Appendix A: Example datasets used in this book
==============================================

Below are interactive SQL interfaces for all of the various databases used in this book, organized by chapter.  Remember that you query the **sqlite_master** table to see the specifications of objects in a given database, e.g.:

::

    SELECT sql FROM sqlite_master WHERE type = 'table';

to see the specifications of the tables in a given database.

Chapter 2: Basic SELECT queries
:::::::::::::::::::::::::::::::

Books and authors database
--------------------------

This database is the simplest form of the books database, containing a **books** table and an **authors** table.

.. activecode:: appendix_a_ch2_books
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    SELECT * FROM books;

Fruit stand database
--------------------

Though an interactive block for this one was not included in chapter 2, this database contains the fruit_stand table shown.

.. activecode:: appendix_a_ch2_fruit_stand
    :language: sql
    :dburl: /_static/fruit_stand.sqlite3

    SELECT * FROM fruit_stand;
