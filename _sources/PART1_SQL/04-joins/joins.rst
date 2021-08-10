=====
Joins
=====

Test

.. activecode:: ch4_example_test
    :language: sql
    :dburl: /_static/books.sqlite3

    SELECT a.name, b.title, b.publication_year
    FROM authors a, books b
    WHERE a.id = b.author_id;
