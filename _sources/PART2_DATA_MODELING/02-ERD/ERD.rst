.. _erd-chapter:

============================
Entity-relationship diagrams
============================

This chapter introduces *entity-relationship diagrams*, or ERDs.  ERDs define a graphical language for data modeling, at a high level of abstraction.  The level of abstraction facilitates communication between the database designers, project users, and data domain experts.

ERDs were conceived of by Peter Chen and described in a 1976 paper.  While various approaches to data modeling existed before ERD, Chen's ERD has stood the test of time to become one of the preferred methods and is in wide use today.  Many authors have expanded upon Chen's basic model, extending the notation in different directions.  As a result, there are many different ERD notations in use.  For this book, we adopt the notation of Elmasri and Navathe (see :ref:`references`); some popular alternative notations are discussed in :numref:`Chapter {number} <alternative-notations-chapter>`.

As we cover the various elements of the ERD notation, our examples will build pieces of a data model for a fictional computer manufacturer.  The complete model is given at the end of the chapter.  You can also find ERDs for some of the databases used in :numref:`Part {number} <sql-part>` in :ref:`appendix-a`.

Basic model
:::::::::::

As the name suggests, ERDs are concerned with *entities* and the *relationships* between them.  Each of these elements and related elements are denoted using simple shapes, containing text labels, and connected with straight lines.

Entities
--------

Entities represent things or objects with independent existence; persons, products, and companies are some examples.  Entities act much like the "nouns" of our visual modeling language.  Entities are denoted by rectangles; the rectangle is labeled with a name indicative of the concept being modeled:

.. image:: entity.svg

It is important to distinguish between an entity, which is a model of the thing or object being modeled, and instances of the entity, e.g., a particular thing or object.  The entity **employee** models all employees, not a specific employee of a company.

Attributes
----------

Entities are further described by their properties, or *attributes*.  Attributes are denoted by ovals, and attached to their entities with straight lines.  For example, employees have names:

.. image:: attribute.svg

You can attach as many attributes as necessary to an entity.

Keys
----

Every entity has at least one attribute that uniquely identifies instances of the entity.  These *key attributes* are indicated by underlining the attribute label.  For our computer company, each employee is given an ID number for unique identification:

.. image:: key_attribute.svg

ERD allows for multiple key attributes; for example, we might wish to also store a government issued identification number (such as the SSN used in the United States) for each employee.  Note that this is not the same as a key attribute composed of multiple parts!  An employee can be uniquely identified by either their company ID or by their government issued identification number - both are not needed.  Composite keys will be discussed in a later section.

Relationships
-------------

Two or more entities may participate in a relationship.  Relationships act like the "verbs" in our modeling language.  Relationships are denoted by diamonds, and are connected to the participating entities by straight lines:

.. image:: relationship_with_entities.svg

This diagram reads like a sentence: "employee works at factory".  Note that no direction is implied by the layout of the diagram; you have to use your knowledge of the data domain to know that the diagram probably does not mean "factory works at employee".

Cardinality ratios and participation
-------------------------------------

How many employees work at a factory, and how many factories can an employee work at?  This is important information for our model (and for the database we will create from it).

*Cardinality ratios* let us indicate the general number of instances of an entity that map to an entity on the other side of the relationship, and vice versa.  The cardinalities defined by the basic model are 1 and N (or n).  A cardinality of 1 actually means "zero or one"; a cardinality of N means "zero, one, or many".  As most relationships are binary having only two participating entities, this gives rise to a small number of commonly occurring cardinality ratios:

- 1:1, read as "one-to-one"
- 1:N, read as "one-to-many" (equivalently, N:1, or "many-to-one")
- N:M or N:N, read as "many-to-many"

We show the cardinalities on our model next to the line connecting the relationship to the entity:

.. image:: relationship_with_cardinalities.svg

This diagram's cardinality ratio implies two statements about the relationship between employees and factories.  First, "each employee works at zero or one factories".  Second, "each factory has zero or more employees working at it".

*Participation* is a closely related topic.  An entity is said to have *total participation* in a relationship if every instance of the entity must be in relationship with instances of the other entity in the relationship.  In a sense, this provides a minimum cardinality for the entity on the other side of the relationship.  Here is an example - note this is a second relationship between **employee** and **factory**:

.. image:: relationship_with_participation.svg

The double line between **factory** and **manages** says that **factory** has total participation in the relationship.  This diagram's cardinality ratio and participation imply two subtly different statements: "each employee manages *zero or one* factories" and "each factory has *exactly one* employee managing it".  That is, every factory is expected to have a manager, but only some employees manage factories.

The opposite of total participation, denoted using a single line, is *partial participation*.

While indicating total participation on an ERD provides useful information, it is not as critical as cardinality ratios.  As we will see in :numref:`Chapter {number} <erd-to-relational-chapter>`, total participation can influence some decisions when converting our diagram to a relational database (particularly for 1:1 relationships), but its absence is generally not harmful.

Putting it together
-------------------

Below is a diagram incorporating the examples above, and with some additional attributes to fill out the **employee** entity:

.. image:: subset_of_ERD.svg

While this is only part of the complete model that we will ultimately develop, it is a valid ERD from which we can build a database.  All of the necessary detail is in place.

There is also no unnecessary duplication of information in our model.


More complex modeling options
:::::::::::::::::::::::::::::

Recursive relationships
-----------------------

Weak entities
-------------


Composite, derived, and multivalued attributes
----------------------------------------------

- composite keys

Relationship attributes
-----------------------


Higher-arity relationships
--------------------------


Beyond notation
:::::::::::::::

- other annotations
- communication - don't get hung up on notation in early stages


Using ERD to design a database
:::::::::::::::::::::::::::::::

Note the abstract nature of this model.  Although we will examine how to turn our ERD into a relational database in :numref:`Chapter {number} <erd-to-relational-chapter>`, the ERD contains no details specific to SQL or relational databases.  In fact, we could as easily create other types of database from it.


Complete model
::::::::::::::



.. image:: complete_ERD.svg


.. |chapter-end| unicode:: U+274F

|chapter-end|


.. raw:: html

   <div style="width: 520px; margin-left: auto; margin-right: auto;">
   <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">
   <img alt="Creative Commons License" style="border-width:0; display:block; margin-left:
   auto; margin-right:auto;" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a>
   <br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/InteractiveResource"
   property="dct:title" rel="dct:type"><i>A Practical Introduction to Databases</i></span> by
   <span xmlns:cc="http://creativecommons.org/ns#" property="cc:attributionName">
   Christopher Painter-Wakefield</span> is licensed under a
   <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">
   Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.</div>
