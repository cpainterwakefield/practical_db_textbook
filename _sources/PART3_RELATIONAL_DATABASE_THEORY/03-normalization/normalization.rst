.. _normalization-chapter:

=============
Normalization
=============

In this chapter we discuss some aspects of how a database is structured - what relation schemas should we use to store our data?  While the material in this chapter is deeply rooted in relational database theory, the basic concepts can be understood without the theoretical foundation, and are important for all database practitioners.  The first part of this chapter, through :numref:`section {number} <normalization-by-example>`, will therefore discuss normalization from an informal perspective that should be accessible without having read the previous chapters on the relational model of the database.  The remainder of the chapter will introduce the mathematical formalism of normalization.  If you are reading the first part of this chapter without having read earlier chapters on the relational model of the database, note we use the terms relation and attribute in discussing the objects in our database, corresponding roughly to tables and columns respectively.

Motivation
::::::::::

Normalization is the process of modifying a database structure to meet certain requirements. These requirements are defined by a series of *normal forms*, which we will define shortly.  We do normalization so that we may fully realize the benefits of using a relational database.  A database that is not properly normalized is generally more difficult to use and more prone to data errors.

Considerations
--------------

- clean/clear expression
  - do one thing
  - reduce NULLs
- ensure data integrity
- efficiency (space vs time)

Designing a database
--------------------

Databases can be created using a number of approaches.  The approach taken depends greatly on the circumstances which have led to the need for a database.

In some cases, data have been previously collected and stored in some fashion, but not organized into something we would consider a database.  Many scientific, industrial, and business processes produce large amounts of data in the form of sensor readings, application logs, reports, and form responses.  This data may exist in electronic form or on paper.  There may be little structure to the data; it may exist in a *flat* form in which there is only one type of record which stores every piece of information relevant to some event.  Creating a database to more efficiently work with such data may be best accomplished using a *top-down* approach, in which relations (or tables) are broken down into multiple, smaller relations.

In contrast, when creating a new software application, a *bottom-up* approach may be preferred.  The application developers and other interested parties work to identify the data that need to be collected and stored.  A multitude of relations will arise naturally, corresponding to different parts of the application.

Whatever the approach, data modeling (:numref:`Part {number} <data-modeling-part`) can provide useful insights, and will typically lead to a better database design; we strongly recommend this as a first step.  Data modeling is very effective at producing relations that accurately represent independent concepts and the relationships between them.

Normalization provides a different perspective on database design.  As with data modeling, our understanding of the real world and our data informs our design choices.  However, while data modeling focuses on mapping concepts in the real world to relations, normalization works to produce a database structure that is more resistant to data errors.  Data modeling ensures our database accurately captures the data we need, while normalization ensures our database can be used effectively.  The two activities are thus complementary.  Whether or not normalization is applied formally, an understanding of normalization and its trade-offs is important for any database designer.


.. _normalization-by-example:

Normalization by example
::::::::::::::::::::::::

- adjunct to design (paths to getting there)
- meaning of normal forms
  - intuitions (do one thing)
  - update anomalies
- basic example without math
  - intuitive explanation of BCNF
- Key and superkey (and functional dependencies?)
- 1NF-3NF (keep this brief)
  - Examples
- Functional dependencies
  - Defined
  - Trivial/non-trivial/completely non-trivial
  - Rules/properties (combining, splitting, transitive)
  - Closure
- BCNF
  - Definition & example
  - BCNF supercedes 1-3NF
  - possible reason to relax to 3NF
  - Complete walkthrough
- Multivalue dependencies & 4NF
- Higher NFs?  Myabe 5NF by example (6NF deals w/temporal data, not in scope)

|chapter-end|


|license-notice|
