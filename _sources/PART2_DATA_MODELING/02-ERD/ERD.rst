.. _erd-chapter:

============================
Entity-relationship diagrams
============================

This chapter introduces *entity-relationship diagrams*, or ERDs.  ERDs define a graphical language for data modeling, at a high level of abstraction.  The level of abstraction facilitates communication between the database designers, project users, and data domain experts.

ERDs were conceived of by Peter Chen and described in a 1976 paper.  While various approaches to data modeling existed before ERD, Chen's ERD has stood the test of time to become one of the preferred methods and is in wide use today.  Many authors have expanded upon Chen's basic model, extending the notation in different directions.  As a result, there are many different ERD notations in use.  For this book, we adopt the notation of Elmasri and Navathe (see :ref:`references`); some popular alternative notations are discussed in :numref:`Chapter {number} <alternative-notations-chapter>`.

As we cover the various elements of the ERD notation, our examples will build pieces of a data model for a fictional computer manufacturer.  The complete model is given at the end of the chapter.  You can also find ERDs for some of the datasets used in :numref:`Part {number} <sql-part>` in :ref:`appendix-a`.

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

ERD allows for multiple key attributes; for example, we might wish to also store a government issued identification number (such as the SSN used in the United States) for each employee.  In this case, we would have two attributes with underlined labels.  Note that this is not the same as a key attribute composed of multiple parts!  An employee can be uniquely identified by either their company ID or by their government issued identification number - you do not need to know both.  Composite keys will be discussed in a later section.

Relationships
-------------

Two or more entities may participate in a relationship.  Relationships act like the "verbs" in our modeling language.  Relationships are denoted by diamonds, and are connected to the participating entities by straight lines:

.. image:: relationship_with_entities.svg

This diagram reads like a sentence: "employee works at factory".  Note that no direction is implied by the layout of the diagram; you have to use your knowledge of the data domain to know that the diagram probably does not mean "factory works at employee".

Cardinality ratios and participation
-------------------------------------

How many employees work at a factory, and how many factories can an employee work at?  This is important information for our model (and for the database we will create from it).

*Cardinality ratios* let us indicate the general number of instances of an entity that map to an entity on the other side of the relationship, and vice versa.  The cardinalities defined by the basic model are **1** and **N** (or **n**).  A cardinality of **1** actually means "zero or one"; a cardinality of **N** means "zero, one, or many".  As most relationships are binary (involving only two participating entities), there are a small number of commonly occurring cardinality ratios:

- 1:1, read as "one-to-one"
- 1:N, read as "one-to-many" (equivalently, N:1, or "many-to-one")
- N:M or N:N, read as "many-to-many"

We show the cardinalities on our model next to the line connecting the relationship to the entity:

.. image:: relationship_with_cardinalities.svg

This diagram's cardinality ratio implies two statements about the relationship between employees and factories.  First, "each employee works at zero or one factories".  Second, "each factory has zero or more employees working at it".

*Participation* is a closely related topic.  An entity is said to have *total participation* in a relationship if every instance of the entity must be matched with instances of the other entity in the relationship.  In effect, this provides a minimum cardinality for the entity on the other side of the relationship.  Here is an example - note this is a second relationship between **employee** and **factory**:

.. image:: relationship_with_participation.svg

The double line between **factory** and **manages** says that **factory** has total participation in the relationship.  This diagram's cardinality ratio and participation imply two subtly different statements: "each employee manages *zero or one* factories" and "each factory has *exactly one* employee managing it".  That is, every factory is expected to have a manager, but only some employees manage factories.

The opposite of total participation, denoted using a single line, is *partial participation*.

While indicating total participation on an ERD provides useful information, it is not as critical as cardinality ratios.  As we will see in :numref:`Chapter {number} <erd-to-relational-chapter>`, total participation can influence some decisions when converting our diagram to a relational database (particularly for 1:1 relationships), but its absence is generally not harmful.

Putting it together
-------------------

Below is a diagram incorporating the examples above, and with some additional attributes to fill out the entities:

.. image:: subset_of_ERD.svg

Note that the **factory** entity does not use a generated key, but a "natural" one - the city in which the factory is located.  (This only works if our company has no more than one factory in a city!)

While this is only part of the complete model that we will ultimately develop, it is a valid ERD from which we could build a database.  All of the necessary detail is in place.

There is also no unnecessary duplication of information in our model.  It is tempting to add attributes or other features that anticipate the database to come; for example, we might think that a employees should have an attribute indicating at which factory they work.  However, the fact that (at least some) employees work at a factory is already implicit in the relationship* **works at**.  This relationship will give rise to the necessary database structures connecting employees to factories.


More complex modeling options
:::::::::::::::::::::::::::::

This section will look at some cases not covered in the examples above, and also reveal some additional notation covering situations not addressed by the basic model above.

Recursive relationships
-----------------------

Relationships can be between an entity and itself.  This is frequently useful, especially in modeling hierarchical relationships.  In our fictional computer company, each employee (except for the head of the company) has a supervisor, who is another employee.  This is easily modeled as a one-to-many relationship connecting **employee** to **employee**:

.. image:: recursive_relationship.svg

For added clarity, we have annotated the lines connecting the relationship with the roles that employees play in the relationship: one supervisor supervises many supervisees.

Weak entities
-------------

In some situations, we may want to model an entity that we do not have a unique identifier for, but which can be uniquely identified in relationship with another entity.  As an example, each of the factories of our computer manufacturer will contain assembly lines.  We wish to track certain information about each assembly line in our database, such as the daily *throughput* of the assembly line (the number of computers it can produce in a day).  We wish to model these as an entity in our data model, but it is not immediately clear what property of an assembly line would make a good identifier.

We could, of course, give every assembly line a generated unique identifier, but there is a more natural way to identify assembly lines.  In each factory, assembly lines are simply numbered starting from 1, most likely in order by their position on the factory floor.  To identify a particular assembly line, we first state which factory it is in, and then its number within the factory.

When an entity is dependent on another entity for full identification, the dependent entity is called a *weak entity*, and notate it using a rectangle with doubled outline.  The weak entity has only a partial, or weak, key - in our example, the number of the assembly line within the factory.  We note the weak key using a dashed underline.  We also call out the relationship that the weak entity depends on for its identity, to distinguish it from any other relationships the weak entity participates in.  We call this relationship the *identifying relationship*, and draw it as a diamond with doubled outline.  The key of the parent entity together with the weak key of the weak entity constitutes a unique identifier for instances of the weak entity.

Here is the diagram of our assembly line example:

.. image:: weak_entity.svg

Composite attributes
--------------------

.. image:: composite_attribute.svg

- composite keys

Multivalued attributes
----------------------

.. image:: multivalued_attribute.svg

Derived attributes
------------------

.. image:: derived_attribute.svg

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
