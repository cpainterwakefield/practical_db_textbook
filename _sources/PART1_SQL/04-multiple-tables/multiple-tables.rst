==========================
Queries on multiple tables
==========================

.. _`Part 2`: ../../PART2_DATA_MODELING/index.html
.. _`Chapter 5`: ../05-table-creation/table-creation.html


So far, we have seen how to retrieve data from individual tables, filtering data on different criteria, ordering the data, and formatting the data with various expressions.  Now we turn to the question of how to retrieve data from more than one table in a single query.  For example, using the example database from previous chapters, we might to see book titles together with author's name **and** birth and death dates. The author's name is in both the **authors** and **books** tables, but book titles are in **books**, while author birth and death dates are in **authors**.  How can we get these together in one result?  This chapter will explain how to retrieve data from multiple tables using *joins*, along the way introducing an expanded books database that will be used going forward.


Simple joins
::::::::::::

To start with, we will consider an abstract example with a small amount of data.  Specifically, we will work with the two tables shown below, named **s** and **t**:

.. image:: joins1.svg

There is no real meaning to this data, but you might notice that the table data suggests a relationship.  Specifically, table **s** has a column **sy** containing small integers; table **t** has a column **ty** similarly containing small integers.  What we want to accomplish is to connect rows from table **s** with rows from table **t** when the values in the **sy** and **ty** columns are the same.  The desired result looks like this:

.. image:: joins1_result.svg

What we are going to do is tell the database that we want to pair up *all* the rows from both tables, and then specify which of the combined rows we want to keep.  Telling the database to pair up all the rows is accomplished simply by doing a **SELECT** query where both tables are listed in the **FROM** clause.  You can see this in action in the interactive tool below:

.. activecode:: ch4_example_cross_product
    :language: sql
    :dburl: /_static/joins.sqlite3

    Note that this tool accesses a database containing just the abstract tables described in this section.  It does not contain any tables related to authors and books!
    ~~~~
    SELECT * FROM s, t;

This is not what we were trying to accomplish, yet.  What we have created here is called a *cross product* of **s** and **t**.  You can see that the desired rows are actually present in the result, but there are a lot of rows we don't want, too.  How can we keep just the desired rows?  Well, they are the only rows where **sy** equals **ty**.  Logically, we can add a **WHERE** clause to express that we want just those rows:

::

    SELECT * FROM s, t
    WHERE sy = ty;

Add this **WHERE** clause to the query in the interactive tool above to confirm that it gives the desired result.  This **WHERE** clause condition (``sy = ty``) is known as a *join condition*, because it relates rows from one table with rows from another table.  The addition of the join condition turns our cross product into a *join*.

In the example so far, we had the case of each row in **s** matching exactly one row in **t**, and each row in **t** matching exactly one row in **s**.  What happens if this is not the case?  First, consider tables **s2** and **t2** below, in which a row in each table fails to match any rows in the other table:

.. image:: joins2.svg

Joining **s2** and **t2** on columns **sy** and **ty** now gives us:

.. image:: joins2_result.svg

This result can again be understood by considering the cross product of **s2** and **t2**, filtered by our join condition (``sy = ty`` again).  Any cross product row where **sy** is 4 has no matching **ty** entry, so the row fails the join condition and gets filtered out.  Similarly any rows where **ty** is 3 are filtered out.  (Tables **s2** and **t2** are also accessible from the interactive tool above, so you can try the cross product and join for yourself.)

We can also have the case where more than one row in one table matches some row in the other table.  Here are two more tables to consider:

.. image:: joins3.svg

If we take the cross product here, it is clear that we will have two combined rows where **sy** and **ty** both equal 2.  We will keep both, since both meet the join condition requirement, giving us this result:

.. image:: joins3_result.svg

Tables **s3** and **t3** are also in the database accessible in the interactive tool above.

Two tables can be related via multiple columns rather than just one in each table.  To join them, you would use a compound join condition using **AND**.  In fact, join conditions do not have to be equality (although they usually are); any logical expression relating rows in one table with rows in another can be used.  Your starting point is the same; think about the cross product, and which rows you want to keep from it [#]_.

Finally, you are also free to add additional conditions to the **WHERE** clause to further refine the result.  For example,

::

    SELECT * FROM s, t
    WHERE sy = ty
    AND tz = 'blue';

We have a lot more to talk about with joins, but before moving on, let's see how to answer the question raised earlier, of seeing both book titles and author dates in one query result using the database from the previous chapters.  Here's an interactive tool on that database (we will be changing to a new database in a couple of sections).

.. activecode:: ch4_example_simple_books_join
    :language: sql
    :dburl: /_static/simple_books.sqlite3

    This tool accesses the database used in chapters 2 and 3.
    ~~~~
    SELECT title, author, birth, death
    FROM books, authors
    WHERE author = name;

Note here that we are choosing specific columns to return as part of our result.  The column **name**, used in the join condition, is the column containing author names in the **authors** table.  We compare this column to the **author** column in **books** for our join, but we don't include it in the columns we retrieve; otherwise we would have the same author name showing in two different columns.


Names of things
:::::::::::::::

We have (mostly) not worried about the *names* of things in our discussion so far.  We have said that we can use a column name as an expression representing the value in the column for some row under consideration, but we now need to consider some scenarios in which a column's name by itself is not sufficiently specific.  We have also given some examples where we renamed the output columns for a **SELECT** query, but we deferred discussion of that technique.  This section will go into both of these topics and more.

Name collisions and ambiguity
-----------------------------

In all of our examples so far, all of the columns in the tables we queried had unique names.  For example, the cross product of **s** and **t** contained columns named **sx**, **sy**, **ty**, and **tz**.  However, we will often not be so lucky when working with multiple tables.  When two columns from tables involved in a query have the same name, we say that the column names *collide*.  When a naming collision occurs, we cannot use the column name by itself as an expression in any part of our query, because the database will not know which table's column you mean; the database will give an error message that the column name is *ambiguous*.

Qualified names
---------------

Fortunately, there is an easy way to specify a particular column in a particular table: simply give the table name first, followed by a dot ("."), and then the column name.  You can do this even if names are not ambiguous. For example the last query above could be expressed as

::

    SELECT books.title, books.author, authors.birth, authors.death
    FROM books, authors
    WHERE books.author = authors.name;

This has the added benefit of making clear where each column is coming from, for anyone reading the query who is not familiar with the database.

You can also use the asterisk shortcut to mean all columns in a specific table by prefixing with the table name and dot:

::

    SELECT books.*, authors.birth, authors.death
    FROM books, authors
    WHERE books.author = authors.name;

The expressions using both the table name and the column name are known as *qualified* column names, and can be used with any database.  In some database implementations, tables can be grouped together into larger containers; in those databases, it is possible to have multiple tables of the same name (in different containers), which now must be qualified using the container name.  Each database implementation is different, so you will need to learn about your particular database system's rules for qualifying names.

When doing a join, it is good practice to qualify all of your column names as we did in the queries above.  This will make it easier for anyone reading or maintaining your code to understand what your query is doing.

Aliasing
--------

SQL provides facilities to change the names of tables and columns within the context of a single query.  This can be useful, and at times, necessary.  We already used column renaming to get nicer column headers in our output.  For example, in the query

::

    SELECT title, floor((publication_year + 99) / 100) AS century FROM books;

we supplied the name "century" for the output column (which otherwise would have a header that looked like the mathematical expression we computed).  This technique is known as *aliasing*, and is accomplished with the **AS** keyword.  Aliasing for columns is most often used for the purpose of giving a helpful name for the column in the output, although it can be applied for other reasons we shall see.

Aliasing can also be used with tables.  This is often used to shorten table names to keep qualified names short and readable.  Here, the **AS** keyword is used in the **FROM** clause after each table that should be renamed.  The alias can then be used in the **SELECT**, **WHERE**, and other clauses in place of the table name.  Here is a query we did above, rewritten using table aliasing:

::

    SELECT b.title, b.author, a.birth, a.death
    FROM books AS b, authors AS a
    WHERE b.author = a.name;

When working with large queries using many tables, aliasing can make the query significantly smaller and more readable.

One instance where table aliasing is required is when joining a table to itself.  This can be done when there is some kind of relationship between rows within the same table, and happens more often than you might guess.  As an example of a query we might do with our books and authors database, consider the question, "what books were published in the same year as *The Three-Body Problem*?".  Here is one way to answer that question with a query:

::

    SELECT b2.*
    FROM books AS b1, books AS b2
    WHERE
      b1.publication_year = b2.publication_year   -- join condition
      AND b1.title = 'The Three-Body Problem';

If this seems confusing, think about it as using two tables, **b1** and **b2**, each containing the same data as **books**.  Then work through what happens if you take the cross product of **b1** and **b2** and apply the join condition ``b1.publication_year = b2.publication_year``; finally, filter that result with the condition ``b1.title = 'The Three-Body Problem'``.

When using table aliasing, you should qualify all of your column names using the aliases as a matter of good style.  Some databases allow you to use original table names instead of aliases, but mixing aliases with original table names is inconsistent and confusing, and in some cases can result in incorrect code that is difficult to debug.

Just remember, aliasing only affects the query in which the renaming occurs; a new query will know nothing about any previous aliasing applied to tables or columns.

As a final note, the **AS** keyword is actually optional in SQL - you can create an alias with this keyword omitted.  Simply put a valid identifier string after the name of a table or after a column expression:

::

  SELECT b.title, b.author, a.birth, a.death
  FROM books b, authors a
  WHERE b.author = a.name;

Leaving out a keyword may seem strange, but you are likely to read code at some point using this form of aliasing, so be aware.


Names with spaces or mixed-case
-------------------------------

Usually, names of things are case-insensitive and do not contain spaces.  Also, the case used when displaying the output headers for a query may be all uppercase or all lowercase, depending on the database (for this textbook, lowercase is the norm).  It is possible, however, to use names which are case-sensitive and which contain spaces.  To do this, put the name within double quotes.  For example, in the query:

::

    SELECT name AS "Name of Author" FROM authors;

the header in the output column will be both mixed-case and contain spaces.

Very rarely, you may encounter a database where table or column names are mixed-case or contain spaces.  This can occur when the database creator used double quotes in the SQL code creating the tables.  In general, this is not a good practice, as it forces the use of double quotes for any queries using the table.


Basic data relationships
::::::::::::::::::::::::

While data can be related to each other in very complex ways, there are some basic relationship types that capture the important aspects of most relationships.  These relationships are commonly called "one-to-one", "one-to-many", and "many-to-many".

*One-to-one* describes a relationship between two types of data.  If we think of each data type as having its own table, then each row in one table has a well-defined relationship with *at most* one row in the other table, and vice versa.  Sometimes each row in a table has exactly one corresponding row in the other table, and vice versa; other times, some rows in one or both tables may not have a corresponding row in the other table.  When there is a true one-to-one correspondence between tables, it is sometimes desirable to combine the tables into one larger table (whether or not to do this is a design concern that we will consider more in `Part 2`_).

An example of a one-to-one relationship, sticking with our books theme, might appear in a database for a seller of used books.  In this database, each of the seller's books is recorded in a table named **catalog**.  Each row in **catalog** will record things such as the book's author and title, condition, and current price.  This imagined database also contains a table named **sales**, which records information when a book is sold, such as the date sold, payment type, a receipt number, and a reference back to the **catalog** to allow us to join the tables together.  (We will shortly discuss what these references should look like.)  Note that every record in the **sales** table corresponds to exactly one record in the **catalog** table; however, any unsold books still in the seller's possession will not have a corresponding **sales** record.

*One-to-many* refers to the case when rows in one table correspond to some number of rows in another table, but rows in the second table correspond to at most one row in the other table.  In some cases, rows in the first table always have at least one corresponding row; other times, rows can have zero or more corresponding rows.  In our earlier books database, we had exactly one **books** record for each **authors** record.  This is not reflective of the real world, in which authors may have written many books.  In the expanded database we will start using shortly, we assume a one-to-many relationship between authors and books - each author has one or more books, but each book has exactly one author.  (Even this is not reflective of the real world - many books exist that were written by two or more authors working together!  However, for simplicity our database only contains single-author books.)  Note that we can also talk of *many-to-one* relationships, which are just the symmetric equivalent of one-to-many; we can say that **authors** is in a one-to-many relationship with **books**, or that **books** is in a many-to-one relationship with **authors**.

*Many-to-many*, you can probably guess, implies that rows in one table may correspond to multiple rows in the other table, and vice versa.  One possible example of this, of course, is the relationship between authors and books in the real world, as mentioned above.  In our expanded database, though, our examples of many-to-many relationships will involve book and author awards.  For example, the Hugo Award is given out each year to a book in the science fiction genre.  In our database, there are many books that have won a Hugo Award; therefore, rows in the **awards** table can relate to multiple rows in the **books** table.  Especially good science fiction books might win both a Hugo Award and a Nebula Award; so rows in the **books** table can correspond to multiple **awards** rows.  As we will see, this relationship type requires additional work to represent efficiently in a relational database; in fact, it will require a third table to keep track of the connections between records in the original two tables.


Identity columns
::::::::::::::::

Regardless of the relationship type, if we want to make the connection between data in one table and data in another using a join, we need the tables to share some data elements in common (in the case of one-to-one or one-to-many) or with a third table (in the case of one-to-many).  In our original books database, the common element was the author's name, which was in both the **books** and **authors** tables; this let us join the two tables with the join condition ``books.author = authors.name``.

For some types of data, some element of the data is unique for every possible data item and can be used as an identifier for the data in a database.  For example, international travel to many countries requires the traveler to have a passport; the issuing country together with the passport number uniquely identifies a traveler.  However, this only works for international travel; most countries do not require passports for travel within the country's own borders, and therefore there are many people who have no passport at all.  A database trying to track domestic travelers, then, cannot use passport information as a unique identifier.

Author names might seem like a good identifier for authors, but in fact, we have to be careful here as well, due to multiple authors sharing the same name.  For example, there are two novelists named Richard Wright, and both a novelist and a poet named David Diop.  We could further distinguish between these authors using their birth dates, or if that wasn't enough, we could consider their birthplace or other attributes of the author.  That only works, of course, if we know the birth date and so forth of each author in our database, and in any case it begins to be an unsatisfactory solution due to the complexity of having to store multiple pieces of information about each author for any tables that relate to our **authors** table.

The solution we adopt, and which is widely used in practice, is to create an artificial unique identifier, or *id*, for each author in our database.  Unique identifiers can take different forms.  Probably the most common scheme is to keep a counter in the database (a special database object called a *sequence* - we will discuss these in `Chapter 5`_), and increment it each time a row is added to a table; the counter value is used as the id value for the new row.  Another popular scheme is to use a very large integer generated at random - a *universally unique identifier*, or UUID.  In this scheme, due to the large number of possible UUIDs, each new id value is very likely to be different from any other previously id in the table. (It is easy to detect if there is a duplicate, in which case another value is generated.)

In the expanded books database (to be revealed in the next section, finally), the **authors** table has an **id** column.  Each row in the **authors** table has a unique **id** value.  The **books** table, meanwhile, no longer has a column storing the author's name.  Instead, it has the column **author_id**.  Each **author_id** is equal to some **id** value from the **authors** table.  Thus, to join the two tables we simply use the join condition ``authors.id = books.author_id``.


The expanded books database
:::::::::::::::::::::::::::

We are now ready to describe the database we will be using for the rest of this book.  The new database is still centered around **book** and **authors** tables, modified to use id columns as described above, but also adds several other tables.  All of the tables and their basic relationships to each other are described below, after which we will discuss some basic join queries using the tables.

.. container:: data-dictionary

    Table **authors** records persons who have authored books:

    ========== ================= ===================================
    column     type              description
    ========== ================= ===================================
    id         integer           unique identifier for author
    name       character string  full name of author
    birth      date              birth date of author, if known
    death      date              death date of author, if known
    ========== ================= ===================================

.. container:: data-dictionary

    Table **books** records works of fiction, non-fiction, poetry, etc. by a single author:

    ================ ================= ===================================
    column           type              description
    ================ ================= ===================================
    id               integer           unique identifier for book
    author_id        integer           id of book's author from **authors** table
    title            character string  book title
    publication_year integer           year book was first published
    ================ ================= ===================================


.. container:: data-dictionary

    Table **editions** records specific publications of a book:

    ================== ================= ===================================
    column             type              description
    ================== ================= ===================================
    id                 integer           unique identifier for edition
    book_id            integer           id of book (from **books** table) published as edition
    publication_year   integer           year this edition was published
    publisher          character string  name of the publisher
    publisher_location character string  city or other location(s) where the publisher is located
    title              character string  title this edition was published under
    pages              integer           number of pages in this edition
    isbn10             character string  10-digit international standard book number
    isbn13             character string  13-digit international standard book number
    ================== ================= ===================================


.. container:: data-dictionary

    Table **awards** records various author and/or book awards:

    ================== ================= ===================================
    column             type              description
    ================== ================= ===================================
    id                 integer           unique identifier for award
    name               character string  name of award
    sponsor            character string  name of organization giving the award
    criteria           character string  what the award is given for
    ================== ================= ===================================


.. container:: data-dictionary

    Table **authors_awards** is a *cross-reference* table; each entry in the table records the giving of an award to an author in a particular year:

    ================== ================= ===================================
    column             type              description
    ================== ================= ===================================
    author_id          integer           id of the author receiving the award
    award_id           integer           id of the award received
    year               integer           year the award was given
    ================== ================= ===================================


.. container:: data-dictionary

    Table **books_awards** is a *cross-reference* table; each entry in the table records the giving of an award to an author for a specific book in a particular year:

    ================== ================= ===================================
    column             type              description
    ================== ================= ===================================
    book_id            integer           id of the book for which the award was given
    award_id           integer           id of the award received
    year               integer           year the award was given
    ================== ================= ===================================

One-to-many relationships in the expanded database
--------------------------------------------------

.. activecode:: ch4_example_one_to_many
    :language: sql
    :dburl: /_static/books.sqlite3

    SELECT a.name, b.title, b.publication_year
    FROM authors AS a, books AS b
    WHERE a.id = b.author_id;


Many-to-many relationships in the expanded database
---------------------------------------------------


Joins using cr
Joins
- joins on 2 tables with id columns
- joins on 2 tables with xref table
- joins on 3 tables or more

JOIN clause, inner and outer joins
::::::::::::::::::::::::::::::::::



.. [#] Cross products are seldom a desired result on their own.  Because a cross product has a number of rows equal to the number of rows in one table times the number of rows in the other table, the product is very large when the tables involved are large.  When database systems process joins, they generally do not create the cross product and then apply the **WHERE** clause conditions, as that would require a lot of memory or temporary storage and be very slow; however, the conceptual model is helpful in understanding the end result.  We will discuss some strategies databases use to implement joins in part 4, chapter XXX.
