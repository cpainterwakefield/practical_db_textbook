.. _appendix-a:

==============================================
Appendix A: Example datasets used in this book
==============================================

..
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

  Though an interactive block for this database was not included in chapter 2, this database contains the **fruit_stand** table shown.

  .. activecode:: appendix_a_ch2_fruit_stand
      :language: sql
      :dburl: /_static/fruit_stand.sqlite3

      SELECT * FROM fruit_stand;

  The expanded books database
  :::::::::::::::::::::::::::

  We are now ready to describe the database we will be using for the rest of this book.  The new database is still centered around **book** and **authors** tables, modified to use id columns as described above, but also adds several other tables.  All of the tables and their basic relationships to each other are described below, after which we will discuss some basic join queries using the tables.  The descriptions below are also repeated in `Appendix A`_ for future reference.

  .. container:: data-dictionary

      Table **authors** records persons who have authored books:

      ========== ================= ===================================
      column     type              description
      ========== ================= ===================================
      author_id  integer           unique identifier for author
      name       character string  full name of author
      birth      date              birth date of author, if known
      death      date              death date of author, if known
      ========== ================= ===================================

  .. container:: data-dictionary

      Table **books** records works of fiction, non-fiction, poetry, etc. by a single author:

      ================ ================= ===================================
      column           type              description
      ================ ================= ===================================
      book_id          integer           unique identifier for book
      author_id        integer           author_id of book's author from **authors** table
      title            character string  book title
      publication_year integer           year book was first published
      ================ ================= ===================================


  .. container:: data-dictionary

      Table **editions** records specific publications of a book:

      ================== ================= ===================================
      column             type              description
      ================== ================= ===================================
      edition_id         integer           unique identifier for edition
      book_id            integer           book_id of book (from **books** table) published as edition
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
      award_id           integer           unique identifier for award
      name               character string  name of award
      sponsor            character string  name of organization giving the award
      criteria           character string  what the award is given for
      ================== ================= ===================================


  .. container:: data-dictionary

      Table **authors_awards** is a *cross-reference* table (explained below) relating **authors** and **awards**; each entry in the table records the giving of an award to an author (not for any particular book) in a particular year:

      ================== ================= ===================================
      column             type              description
      ================== ================= ===================================
      author_id          integer           author_id of the author receiving the award
      award_id           integer           award_id of the award received
      year               integer           year the award was given
      ================== ================= ===================================


  .. container:: data-dictionary

      Table **books_awards** is a *cross-reference* table (explained below) relating **books** and **awards**; each entry in the table records the giving of an award to an author for a specific book in a particular year:

      ================== ================= ===================================
      column             type              description
      ================== ================= ===================================
      book_id            integer           book_id of the book for which the award was given
      award_id           integer           award_id of the award given
      year               integer           year the award was given
      ================== ================= ===================================


  Data models
  :::::::::::

  ERD and other notations


----

|license-notice|
