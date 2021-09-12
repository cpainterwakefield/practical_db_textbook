.. _data-modification-chapter:

==============
Modifying data
==============

This chapter will explain the basic mechanisms for adding data to tables, removing data from tables, and modifying data.

Tables used in this chapter
:::::::::::::::::::::::::::

For this chapter, we will work with the tables **bookstore_inventory** and **bookstore_sales**, which simulate a simple database a seller of used books might use.  A full description of these tables can be found in :ref:`Appendix A <appendix-a>`.

Adding data using INSERT
::::::::::::::::::::::::

To add rows to a table in the database, we use a query starting with the keyword **INSERT**.  In its simplest form, you add a single row to a table by providing a value for each column in the table as defined.  For example,

.. activecode:: data_modification_example_insert
    :language: sql
    :dburl: /_static/textbook.sqlite3



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


.. |chapter-end| unicode:: U+274F

|chapter-end|

----

**Notes**
