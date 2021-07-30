===================
Introduction to SQL
===================

    *In God we trust, all others bring data.*

    -- attributed to W. Edwards Deming

The main goal of this chapter is to get you thinking about what it is we are studying when we study databases.  After exploring what we mean by *data* and *database*, you will learn some modern database concepts, and then wrap up with a quick example of working with a *relational database*, the subject of most of this book.

What is data? What is a database?
:::::::::::::::::::::::::::::::::


.. index:: data; defined

Data
----

What is data?  A dictionary may tell you that *data* is the plural of *datum*, and that a datum is a piece of information or a fact.  Okay, so data is pieces of information.  Living in the "information age", we hear a lot about the importance of data.  Companies like Google and Facebook collect lots and lots of data.  Data must be very useful!  

Here are some data:

- The ratio of a circle's circumference to its diameter is :math:`\pi`
- Elephants are classified as belonging to the taxonomic order Proboscidea
- The population of planet Earth in 2018 was approximately 7.6 billion
- Isaac Newton was born on December 25, 1642

Are these useful?  Why or why not?

There are numerous ways in which we might consider the utility of the data available.  Probably most important is whether or not the data is actually relevant to current needs; that is, does it answer a question you need the answer to?  It is very well to know that elephants are proboscideans, but does that information advance you towards your current goals?

Another important consideration is the accuracy of the data.  To the best of the author's knowledge the data listed above are accurate.  It helps that these are well-known facts; it is much more difficult to verify, for example, a person's yearly income.  Sometimes data with some inaccuracy can still be useful, at least if we have enough of it and if we are interested in asking questions about the data *in aggregate*, that is, all together.  In that setting, we can tolerate a certain amount of error as statistical "noise".

Completeness is a quality that is dependent on the task at hand.  Perhaps it is helpful to know that elephants are proboscideans, but what we really wanted was a complete list of proboscideans, living or extinct; in that case, the data is woefully incomplete.  Or perhaps we want to know many more facts about Isaac Newton than simply his birth date.  Again, incomplete data is not necessarily useless, but missing data can be troublesome if not handled carefully.

Ultimately, data can be useful to us only if we have access to it, and that is where the database comes in. 

.. index:: 
    single: database; defined
    single: database management system
    see: DBMS; database management system

Database
--------

Turning again to the dictionary, the word *database* is usually defined as a collection of related data.  Definitions will also often mention that the database includes facilities for accessing the data in some useful fashion, and that a computer is involved.  The word database is fairly new, as words go; the term *data-base* first appeared around 1962 [#]_.  It is not a coincidence that this is shortly after the introduction of durable random access storage for computers in the form of the hard drive.  (This connection is explored further in chapter xxx.)

While the dictionary definitions are a good starting point, databases in the modern era are complex creations that include contributions from almost every area of computer science: computer engineering, software engineering, programming languages, networks, algorithms, data structures, and more.  How all these elements work together to make a database *system* is outside of the scope of this book; we will touch on certain areas where they are useful, but our focus is on the skills of practical application to users of databases.

Here are two working definitions that will be useful going forward:

database
    An organized collection of data.

database management system (DBMS)
    The software that manages the storage of data in a database and provides facilities for searching, retrieving, and modifying data in a database.

However, the word database is frequently used as shorthand to mean both a database and the DBMS managing it.

.. index:: database; properties of, RDBMS

Properties of modern databases
::::::::::::::::::::::::::::::

- abstraction - including query languages
- multiuser, network
- define RDBMS == SQL database

need to check sources on this!!!


.. index::
    single: SQL
    see: Structured Query Language; SQL
    single: query language

Structured Query Language (SQL)
:::::::::::::::::::::::::::::::

An important characteristic of modern databases is that they abstract low-level operations on files, tables, indexes and so forth into high-level requests to the database management system (DBMS).  Requests to the database are typically expressed in a *query language*.  Query languages are a type of programming language, but typically differ in that query languages tend to be declarative rather than imperative.  Whereas imperative languages require the programmer to specify exactly what steps to take to perform a task, declarative languages simply describe a desired outcome.

Structured query language (SQL) is the most popular query language for relational databases, and is an example of a declarative language.  SQL was developed by engineers at IBM in the 1970s <<citation needed>> and was standardized by standards organizations ANSI and ISO starting in 1986, with regular updates.  New features continue to be added to SQL as relational database technology and applications evolve.

Even though a standard exists for SQL, relational databases have slightly different implementations of the relational model, and similarly different "dialects" of SQL.  Despite these differences, most SQL dialects share enough in common that, with care, highly portable SQL code can be written.  Examples in this textbook use the SQLite database engine, and thus the dialect of SQL used by SQLite.  To the extent possible, however, examples will be portable to other dialects; where there are significant differences between implementations, these will be noted.  Due to the large number of relational DBMSes available, only a few highly popular DBMSes are included:  SQLite, PostgreSQL, MySQL, Oracle, and Microsoft SQL Server.

Note that `Appendix A`_ provides SQL scripts to create each of the databases used in this book for each of the database engines listed above.

.. _`Appendix A`: ../appendix-a-datasets/datasets.html

.. index::
    see: DQL; data query language
    single: data query language
    see: DML; data manipulation language
    single: data manipulation language 
    see: DDL; data definition language
    single: data definition language
    see: DCL; data control language
    single: data control language

SQL is sometimes subdivided into smaller languages focused on particular tasks: a data query language (DQL) for retrieving data, a data manipulation language (DML) for modifying data, a data definition language (DDL) for defining and modifying database objects, and a data control language (DCL) for managing authorization and access to data.  These distinctions are largely unimportant, and will not be used in this textbook.

The basic element of SQL is the *statement* or *query*.  While a distinction can be made between these two (statements acting to change the state of the database, queries acting to retrieve data), it is common to use the terms interchangeably.  Statements in SQL read much like English sentences, except they are terminated by semicolons instead of periods.  Here are a couple of examples of SQL statements; you might already have some intuition as to what these do (they will be thoroughly explained in later chapters):

::

    SELECT title, author FROM books;

    UPDATE books
    SET publication_year = 2021
    WHERE title = 'A Practical Introduction to Databases';

SQL is often pronounced as "sequel", although it is equally valid to simply say each letter.

Example database
::::::::::::::::

As you work through this textbook, there will be frequent interactive examples based on books and book publishing.  There are several databases available, which are described in more detail in `Appendix A`_.

.. _`Appendix A`: ../appendix-a-datasets/datasets.html

- displaying table information --> select name, sql from sqlite_master




.. [#] `"database, n" <http://www.oed.com/view/Entry/47411>`_. OED Online. Oxford University Press. June 2013. Retrieved July 12, 2013.
