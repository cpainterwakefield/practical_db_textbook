============
Introduction
============

    *In God we trust, all others bring data.*

    -- attributed to W. Edwards Deming

The main goal of this chapter is to get you thinking about what it is we are studying when we study databases.  After exploring what we mean by *data* and *database*, you will learn some modern database concepts, and then wrap up with a quick example of working with a *relational database*, the subject of most of this book.

What is data? What is a database?
:::::::::::::::::::::::::::::::::

What is data?  A dictionary may tell you that *data* is the plural of *datum*, and that a datum is a piece of information or a fact.  Okay, so data is pieces of information.  Living in the "information age", we hear a lot about the importance of data.  Companies like Google and Facebook collect lots and lots of data.  Data must be very useful!  

Here are some data:

- The ratio of a circle's circumference to its diameter is :math:`\pi`
- Elephants are classified as belonging to the taxonomic order Proboscidea
- The population of planet earth in 2018 was approximately 7.6 billion
- Isaac Newton was born on December 25, 1642

Are these useful?  Why or why not?

The value of data
-----------------

- relevance to current needs
- accuracy, completeness
- context, aggregation



The modern database
:::::::::::::::::::


Relational databases
::::::::::::::::::::

- What is data?  What is a database?
- Characteristics of a modern database?
- Relational database basics
    - A simple example

.. activecode:: ch1_example_simple
    :language: sql
    :dburl: /_static/fruitstand.sqlite3

    A simple example of a SQL query, perhaps listing products at a fruit seller's:
    ~~~~
    SELECT * FROM fruitstand;
