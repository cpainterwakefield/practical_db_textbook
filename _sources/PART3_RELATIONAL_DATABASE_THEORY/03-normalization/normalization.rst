.. _normalization-chapter:

=============
Normalization
=============

In this chapter we discuss some aspects of how a database is structured - what relation schemas should we use to store our data?  While the material in this chapter is deeply rooted in relational database theory, the basic concepts can be understood without the theoretical foundation, and are important for all database practitioners.  The first part of this chapter, through :numref:`section {number} <normalization-by-example>`, will therefore discuss normalization from an informal perspective that should be accessible without having read the previous chapters on the relational model of the database.  The remainder of the chapter will introduce the mathematical formalism of normalization.  If you are reading the first part of this chapter without having read earlier chapters on the relational model of the database, note we use the terms relation, attribute, and tuple in discussing the objects in our database; these terms closely correspond to table, column, and row in relational database systems.

Motivation
::::::::::

Normalization is the process of modifying a database structure to meet certain requirements. These requirements are defined by a series of *normal forms*, which we will define shortly.

A primary goal of normalization is to make it easier to maintain a correct collection of data.  Correct data is complete and self-consistent.  The database should not contain data contradicting what we understand to be true.

There are a few clues that can indicate a database is susceptible to common kinds of data corruption.  A very big clue is when a database exhibits redundancy, that is, when the same facts are recorded multiple times.  Another clue is when the database contains NULL in many places.  When these issues are present in a relation, it is usually also the case that the relation is doing too many things.  Relations work best when they contain data regarding a single thing or concept.  It can sometimes be difficult to detect that a relation is doing too much, though, which is why redundancy and excessive NULLs are such useful clues.

Modification anomalies
----------------------

There are three common types of errors that we can consider to better understand the need for normalization.  Each of these *modification anomalies* arise from one of the data modification operations available to us: insert, update, and delete.  We can illustrate each of these with the relation pictured below, which lists information about some classes and teachers at a fictional school.  The relation gives the class name, starting time, room number, instructor name, instructor office number, and instructor department.  Right away, we should be concerned that there is information in the relation about both *classes* and *instructors*.

.. table:: **classes**
    :class: lined-table

    ======== ==================== ===== =========== ====== ==========
    class_id class_name           time  instructor  office department
    ======== ==================== ===== =========== ====== ==========
    1        Algebra I            8:00  Mr. Reyes   124    Math
    2        Geometry             10:00 Mr. Reyes   124    Math
    3        World History        9:00  Ms. Tan     111    Humanities
    4        English Literature   10:00 Ms. Larsen  105    Humanities
    5        Physics              11:00 Ms. Musa    122    Science
    6        Chemistry            13:00 Ms. Musa    122    Science
    7        Music                9:00  Mr. Pal     103    Humanities
    ======== ==================== ===== =========== ====== ==========

Insert anomaly
##############

Consider what happens when a new instructor joins the faculty at the school.  Mr. Hassan is a new faculty member in the department of Math, but he has not yet been assigned any classes to teach.  How should we add Mr. Hassan to the database?  There are a few options, but none of them are good - whatever we do, we must provide values for **class_id**, **class_name**, and **time** in our new tuple.  We might think that NULL is the best choice for each of these attributes, as otherwise we must create fake class information.  Either way, we are making trouble for ourselves later on - our database now contains a tuple that must be handled in a special fashion, different from the other tuples in the relation.

Delete anomaly
##############

Now, consider what happens when Ms. Tan takes a job at another school.  If we are incautious, we will delete the tuple for Ms. Tan - removing World History from our list of classes altogether.  Similarly, we cannot remove English Literature from the list of classes without removing all information about Ms. Larsen.  The only way to avoid these issues with our current database design is to replace the existing values with NULLs.  As with the insert anomaly example, this leaves us with tuples that require special handling.

Update anomaly
##############

Update anomalies are a direct consequence of the redundancy in our database.  Consider what happens when Mr. Reyes changes offices.  If we are incautious, we will update the tuple listing Mr. Reyes as the teacher of Algebra I, but forget to update the tuple for Geometry, leaving our data internally consistent.  Mr. Reyes will be listed as having two different offices, without any indication which is correct.  To avoid trouble, we must remember to always update *all* classes for which Mr. Reyes is the instructor.

Solution
--------

Normalizing the **classes** relation will prevent each of the situations above.

A primary goal is structuring relations such that the data is expressed in a very simple and consistent form.  In a given relation, some thing or concept is uniquely identified by a primary key composed of one or more attributes; every other attribute should represent a *single-valued* fact about the thing or concept.  For example, a relation about persons might associate each person with a unique identifier, and include attributes for the person's name and other relevant facts about the person (this assumes all persons have one name - not always a correct assumption!).  The relation should *not* contain multi-valued facts about the person, such as employment history or the names of the person's children.  The relation should *not* contain extended facts about some related thing or concept; the person relation might include the name of the country the person resides in (or some key representing the country), but no other facts about the country.  Extended facts belong in another relation.


redundancy, data consistency, null

Other goals include reducing or eliminating redundancy, and ensuring data integrity.   Continuing our person example, we would not want the person's name spelled differently in one place in the database than in another; we would not want the person's country of residence to be different in one place than in another.  This goal is largely met when our primary goal is met; when data is in its simplest form, each fact is expressed once and only once, eliminating

Example
-------





.. _normalization-by-example:

Normalization by example
::::::::::::::::::::::::


Data modeling and normalization
:::::::::::::::::::::::::::::::

Databases can be created using a number of approaches.  The approach taken depends greatly on the circumstances which have led to the need for a database.  Data modeling (:numref:`Part {number} <data-modeling-part`) and normalization both have a role to play.

In some cases, data have been previously collected and stored in some fashion, but not organized into something we would consider a database.  Many scientific, industrial, and business processes produce large amounts of data in the form of sensor readings, application logs, reports, and form responses.  This data may exist in electronic form or on paper.  There may be little structure to the data; it may exist in a *flat* form in which there is only one type of record which stores every piece of information relevant to some event.  Creating a database to more efficiently work with such data may be best accomplished using a *top-down* approach, in which relations (or tables) are broken down into multiple, smaller relations.

In contrast, when creating a new software application, a *bottom-up* approach may be preferred.  The application developers and other interested parties work to identify the data that need to be collected and stored.  A multitude of relations will arise naturally, corresponding to different parts of the application.

Whatever the approach, data modeling can provide useful insights, and will typically lead to a better database design; we strongly recommend this as a first step.  Data modeling is very effective at producing relations that accurately represent independent concepts and the relationships between them.

Normalization provides a different perspective on database design.  As with data modeling, our understanding of the real world and our data informs our design choices.  However, while data modeling focuses on mapping concepts in the real world to relations, normalization works to produce a database structure that is more resistant to data errors.  Data modeling ensures our database accurately captures the data we need, while normalization ensures our database can be used effectively.  The two activities are thus complementary.  Whether or not normalization is applied formally, an understanding of normalization and its trade-offs is important for any database designer.


Trade-offs
----------

- efficiency (space vs time)



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
