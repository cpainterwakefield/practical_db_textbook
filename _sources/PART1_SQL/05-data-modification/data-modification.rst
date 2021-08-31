=======================
Modifying data with SQL
=======================

We have so far concentrated on the basics of retrieving data from relational databases using SQL, and future chapters will explore more advanced topics in data retrieval.  You may be wondering how to get data *into* a database in the first place.  This chapter will explain the basic mechanism for adding data to tables, as well as how to remove or modify data.

Adding data using INSERT
::::::::::::::::::::::::

.. activecode:: ch5_example_insert
    :language: sql
    :dburl: /_static/books.sqlite3

    SELECT b.title, a.name AS award, ba.year
    FROM books AS b, awards AS a, books_awards AS ba
    WHERE b.id = ba.book_id
    -- missing: AND a.id = ba.award_id
    ;

Removing data with DELETE
:::::::::::::::::::::::::

Modifying data with UPDATE
::::::::::::::::::::::::::

- INSERT
    - with columns or without
    - omitted columns (mention defaults)
    - multiple rows
    - using SELECT result
- DELETE
    - application of WHERE clause
    - pro-tip: avoiding deleting what you don't want to DELETE
- UPDATE
    - WHERE clause again
    - SET expressions
- mention: TRUNCATE, MERGE
