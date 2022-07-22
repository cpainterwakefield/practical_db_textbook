.. _other-notations-chapter:

===============================
ERD alternatives and variations
===============================

In this chapter we explore alternatives to Chen notation entity-relationship data modeling as well as other variations.  We start with *crow's foot* notation, a popular notation for entity-relationship modeling.  We will show how the basic crow's foot notation can be used to model relational databases at lower levels of abstraction.  We will finish with an overview of some of the more common variations you are likely to encounter in both Chen and crow's foot notation.  The examples used throughout correspond to the computer manufacturer data model developed in :numref:`Chapter {number} <erd-chapter>`.

Crow's foot notation
::::::::::::::::::::

"Crow's foot" is the nickname given to a family of modeling notations that originated in a :ref:`paper by Gordon Everest <data-modeling-references>` in 1976.  The notation has been widely adopted, expanded, and modified since.  The name comes from the symbol used to represent the "many" cardinality in relationships, which resembles a crow's foot (Everest originally called the symbol an "inverted arrow", and later a "fork").

Entity-relationship modeling
----------------------------

In crow's foot notation, entities are represented with simple rectangles.  Entities can be labeled simply with a name for a higher level of abstraction, or they can contain a name and a list of attributes.  Key attributes can be indicated in various ways; we will style keys with bold type and underlining.  In our example data model, we have an entity modeling data regarding employees of our fictional computer manufacturer:

.. image:: crows_foot_entity.svg
    :alt: A rectangle with the header "employee" and a list of the attributes of the employee entity; the ID attribute is boldface and underlined

Relationships are represented with lines connecting entities.  Cardinality ratios are specified with different line ending symbols.  The possible cardinalities are:

.. |cf_zero_or_one| image:: crows_foot_zero_or_one.svg
    :alt: A horizontal line ending in a symbol composed of an open circle and a vertical line

.. |cf_one_exactly| image:: crows_foot_one_exactly.svg
    :alt: A horizontal line ending in a symbol composed of two vertical lines

.. |cf_zero_or_more| image:: crows_foot_zero_or_more.svg
    :alt: A horizontal line ending in a symbol composed of an open circle and three branching lines

.. |cf_one_or_more| image:: crows_foot_one_or_more.svg
    :alt: A horizontal line ending in a symbol composed of a vertical line and three branching lines

=================== ============
|cf_zero_or_one|    Zero or one
|cf_one_exactly|    One exactly
|cf_zero_or_more|   Zero or more
|cf_one_or_more|    One or more
=================== ============

These cardinality symbols have two parts.  The symbol closest to the entity indicates the maximum cardinality - either *one* (represented by a vertical line) or *many* (represented by the branching "crow's foot").  The symbol further from the entity indicates minimum cardinality - either *zero* (represented by the open circle) or *one*.  A minimum cardinality of one is sometimes stated in terms of the entity being *mandatory*, while a minimum cardinality of zero indicates an *optional* entity.

Consider the relationship pictured below:

.. image:: crows_foot_cardinality_example.svg
    :alt: A rectangle labeled "A" connected to a rectangle labeled "B"; the connector on the A side is the "one exactly" connector, while the connector on the "B" side is the "zero or more" connector

This diagram tells us that an instance of the **A** entity can be associated with zero or more instances of **B**.  An instance of **B** must be associated with exactly one instance of **A**.

Relationships may be further annotated with text to name or describe the relationship.

In our example data model, the **employee** entity participates in several relationships with itself and the **factory** entity:

.. image:: crows_foot_relationships.svg
    :alt: Entities employee and factory and their relationships; the employee entity has attributes ID (key), name, position, pay rate, and pay type; the factory entity has the key attribute city; the relationships are the many-to-one (optional on both sides) relationship between employee and factory labeled "works at", the one-to-one (exactly one on the employee side) relationship between employee and factory labeled "manages", and the many-to-one (optional on both sides) relationship between employee and itself labeled "supervises".

The basic crow's foot notation can be extended to encompass the same advanced elements as Chen's notation, such as composite, derived, and multivalued attributes, weak entities and partial keys, relationship attributes, and higher arity relationships.  Different drawing and modeling tools may or may not provide direct support for these, and notations vary widely.

Some database designers prefer crow's foot notation over Chen's notation for entity-relationship modeling, in part due to its relatively compact form.  We can compare the above diagram side-by-side with its equivalent in Chen notation:

.. image:: comparison.svg
    :alt: Side-by-side diagrams in crow's foot and Chen's notation showing the employee and factory entities and their attributes and relationships

Ultimately which notation you use will depend on your preferences and the preferences of the people you are working with on a given project.


Lower level models
------------------

We have so far been discussing data modeling at an abstract level, which we might call the *conceptual* level.  At the conceptual level, the emphasis is on the fundamental data entities and their relationships, rather than the relational database constructs necessary to implement them.  For example, in conceptual modeling we typically do not include depictions of cross-reference tables, which are necessary in the implementation of many-to-many relationships, but which do not themselves represent entities of interest.  We might say that the conceptual model is focused on the *data* rather than the *database*.

At the next level of abstraction lives the *logical* model, which includes all of the relational database structure that results from applying the techniques described in :numref:`Chapter {number} <erd-to-relational-chapter>`.  Crow's foot notation is well suited to modeling at this level.

Below we show the conceptual and logical versions of two parts of our example data model.  Rectangles in the logical model now represent actual tables and list all columns in the table.  We show columns participating in primary keys in boldface and underlined; foreign key columns are italicized.  Logical models often include data types, but we have omitted those details for now.

.. figure:: crows_foot_conceptual_1.svg

    A conceptual model showing entities **employee** and **factory** and their relationships.

.. figure:: crows_foot_logical_1.svg

    The logical model constructed from the above conceptual model.  Note the addition of foreign key columns in both tables.

.. figure:: crows_foot_conceptual_2.svg

    A conceptual model showing entities **part** and **vendor** and the many-to-many relationship between them.  The relationship has an attribute, which we have shown as a rectangle connected to the relationship line.

.. figure:: crows_foot_logical_2.svg

    The logical model constructed from the above conceptual model.  The many-to-many relationship has been realized as a cross-reference table.

If we choose, we can add even more detail to create a *physical* model.  The physical model would definitely include data types as well as any constraints on columns or tables, and might include details such as indexes or even where a table lives on disk or on the network.

Each level of abstraction has value, but whether or not you create models at a particular level will depend on your needs.  As discussed in :numref:`Chapter {number} <erd-chapter>`, models at the highest levels of abstraction are particularly valuable in the early stages of developing a database, and in communicating with all of the various stakeholders in a project.  The conceptual model can be used to produce a database directly, or you may prefer to create a logical model as an intermediate stage.  On the other hand, for some projects you may skip the conceptual level and start with a logical model.  It can be very useful to maintain a logical model as documentation for a database; with large and complex databases, even regular users of the database can forget the names of tables and columns!  Physical models are mostly used by database administrators (DBAs) on very complex projects and are usually created in software tools that can also generate the SQL code to create the database.


Common variations
::::::::::::::::::

Most visual languages for data modeling derive in greater or lesser degree from Chen's notation or crow's foot notation, although alternatives exist.  One popular alternative is the *unified modeling language* (UML).  While UML is not specifically intended for database design, it has been adapted for the purpose.  UML is especially applicable in more advanced settings involving inheritance hierarchies for entities.  Chen's notation has also been extended for these settings.  We do not cover inheritance in this book.

All data modeling languages share certain commonalities, such as entities, attributes, keys, relationships, and cardinality ratios.  Most have some notion of participation or minimum cardinality.  The basic concepts are the same, but the notations vary.  We give an overview of the most common variations you are likely to encounter below.

Cardinality ratios and participation
------------------------------------

Participation and minimum cardinality can be equated when working with binary relationships.  If an entity has total participation in a binary relationship, then the minimum cardinality of the other entity is one (or at least, not zero).  Conversely, partial participation of an entity implies a minimum cardinality of zero for the other entity.  Typically you will use either participation or minimum cardinality, but not both.

.. |single-line| image:: single_line.svg
    :alt: A single line

.. |double-line| image:: double_line.svg
    :alt: A double line

.. |dotted-line| image:: dotted_line.svg
    :alt: A dashed line

.. table:: Participation

    +-----------------------+----------------------------+----------------------------+
    |                       | This book                  |  Alternative notation      |
    +=======================+============================+============================+
    | Partial participation | |single-line|              | |dotted-line|              |
    +-----------------------+----------------------------+----------------------------+
    | Total participation   | |double-line|              | |single-line|              |
    +-----------------------+----------------------------+----------------------------+

.. |zero_or_one_p| image:: zero_or_one_parenthetical.svg
    :alt: A line annotated with "(0,1)" at one end

.. |one_exactly_p| image:: one_exactly_parenthetical.svg
    :alt: A line annotated with "(1,1)" at one end

.. |zero_or_more_p| image:: zero_or_more_parenthetical.svg
    :alt: A line annotated with "(0,N)" at one end

.. |one_or_more_p| image:: one_or_more_parenthetical.svg
    :alt: A line annotated with "(1,N)" at one end

.. |two_or_three_p| image:: two_or_three_parenthetical.svg
    :alt: A line annotated with "(2,3)" at one end

.. |zero_or_one_r| image:: zero_or_one_range.svg
    :alt: A line annotated with "0..1" at one end

.. |one_exactly_r| image:: one_exactly_range.svg
    :alt: A line annotated with "1..1" at one end

.. |zero_or_more_r| image:: zero_or_more_range.svg
    :alt: A line annotated with "0..N" at one end

.. |one_or_more_r| image:: one_or_more_range.svg
    :alt: A line annotated with "1..N" at one end

.. |two_or_three_r| image:: two_or_three_range.svg
    :alt: A line annotated with "2..3" at one end

.. table:: Minimum and maximum cardinality

    +-----------------------+----------------------------------------+---------------------------------------------------------------------------------+
    |                       | Crow's foot notation                   |  Alternative notations                                                          |
    +=======================+========================================+============================================+====================================+
    | Zero or one           | |cf_zero_or_one|                       | |zero_or_one_p|                            | |zero_or_one_r|                    |
    +-----------------------+----------------------------------------+--------------------------------------------+------------------------------------+
    | Exactly one           | |cf_one_exactly|                       | |one_exactly_p|                            | |one_exactly_r|                    |
    +-----------------------+----------------------------------------+--------------------------------------------+------------------------------------+
    | Zero or more          | |cf_zero_or_more|                      | |zero_or_more_p|                           | |zero_or_more_r|                   |
    +-----------------------+----------------------------------------+--------------------------------------------+------------------------------------+
    | One or more           | |cf_one_or_more|                       | |one_or_more_p|                            | |one_or_more_r|                    |
    +-----------------------+----------------------------------------+--------------------------------------------+------------------------------------+
    | Specified min/max     |                                        | |two_or_three_p|                           | |two_or_three_r|                   |
    +-----------------------+----------------------------------------+--------------------------------------------+------------------------------------+


Attributes
----------

In our presentation of crow's foot logical models above, we used text styling (boldface and underlining) to indicate primary keys.  We used italics to indicate foreign keys.  Many drawing and modeling tools similarly use text styling to indicate keys, although not necessarily the styling we used.  Tools may also or instead use background or foreground colors to indicate keys and other properties of columns.

Many tools will also (or instead) indicate primary and foreign key columns with text indicators, usually "PK" and "FK".  Some will highlight primary keys by separating them from the other columns:

.. image:: entity_alternative.svg
    :alt: The entity employee with the primary key attribute labeled with "PK" and with the foreign key attributes labeled with "FK"



|chapter-end|


|license-notice|
