.. _data-modeling-intro-chapter:

=============================
Introduction to data modeling
=============================

When beginning a project requiring a database, it is useful to expend some effort thinking about an appropriate database structure.  Unless your database will be very small (involving one or two tables), if you simply sit down and start creating database tables, you will likely run into trouble fairly quickly.  It is quite easy to create a database that - even though it stores the necessary data - is difficult to understand, update, and query.

Much like software creation, database creation benefits from *analysis* and *design* activities.  Analysis can include discussions with persons with a thorough understanding of the data to be stored, discussions with the intended users of the database, and an examination of representative data.  Design uses the insights gained from analysis to produce a database structure that can correctly store the necessary data and meet user requirements.  Analysis and design are complementary activities; analysis informs design, which in turn uncovers new questions for further analysis.  The back-and-forth between analysis and design works to eliminate the kinds of problems that otherwise result from partial understanding and incorrect assumptions.

Analysis and design may be facilitated using a variety of graphical and non-graphical documents.  *Data models* are one type of document that are most useful when creating a database.  At minimum, a data model captures the various types of data and how these types relate to each other.  Some data models go further in detail, capturing specific programming elements or even aspects of the physical computer systems or networks on which the database will be deployed.

Use data models to reason about data, abstracted from the details of SQL or a programming language.  While it may seem that there are very many types and notations for data models, in practice there are only a few important underlying concepts to learn.  There are many benefits to working with a data model: better realization of user's needs, a database structure that allows easy querying and consistent updates, and documentation of the system for new users, to list a few.  Even for an already established database, if the database lacks documentation in the form of a data model, you may find value in creating one to better understand the system.

This part of the book examines graphical data models at varying levels of abstraction.  In :numref:`Chapter {number} <erd-chapter>`, we discuss the *entity-relationship diagram* (ERD), a high-level model of data.  ERDs are particularly useful at the intersection of analysis and design, facilitating discussion with users and domain experts.  :numref:`Chapter {number} <erd-to-relational-chapter>` explains how to transition from an ERD into a relational database.  This transition may be mediated by less abstract models which have closer correspondence with the relational database, discussed in :numref:`Chapter {number} <other-notations-chapter>`.  :numref:`Chapter {number} <other-notations-chapter>` also discusses some of the many variations in notation in common usage.



|chapter-end|


|license-notice|
