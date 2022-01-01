.. _normalization-chapter:

.. |zero-width-space| unicode:: &+200B

=============
Normalization
=============

In this chapter we discuss some aspects of how a database is structured - what relation schemas should we use to store our data?  While the material in this chapter is deeply rooted in relational database theory, the basic concepts can be understood without the theoretical foundation, and are important for all database practitioners.  If you are reading this chapter without having read earlier chapters on the relational model of the database, note we use the terms relation, attribute, and tuple in discussing the objects in our database; these terms closely correspond to the terms table, column, and row in relational database systems.

Introduction
::::::::::::

Normalization is the process of modifying a database structure to meet certain requirements. These requirements are defined by a series of *normal forms*, which we will define shortly.

A primary goal of normalization is to make it easier to maintain a correct collection of data.  Correct data is complete and self-consistent.  The database should not contain data contradicting what we understand to be true.

There are a few clues that can indicate a database is susceptible to common kinds of data corruption.  A very big clue is when a database exhibits redundancy, that is, when the same facts are recorded multiple times.  Another clue is when the database contains NULL in many places.  When these issues are present in a relation, it is usually also the case that the relation is doing too many things.  Relations work best when they contain data regarding a single thing or concept.  It can sometimes be difficult to detect that a relation is doing too much, though, which is why redundancy and excessive NULLs are such useful clues.

Modification anomalies
----------------------

There are three common types of errors that we can consider to better understand the need for normalization.  Each of these *modification anomalies* arise from one of the data modification operations available to us: insert, update, and delete.  We can illustrate each of these with the relation pictured below, which lists information about some classes and teachers at a fictional school.  The relation gives the class name, starting time, room number, instructor name, instructor office number, and instructor department.  Right away, we should be concerned that there is information in the relation about both *classes* and *instructors*.

.. table:: **classes**
    :class: lined-table

    ==================== ===== ========= =========== ====== ==========
    class_name           time  classroom instructor  office department
    ==================== ===== ========= =========== ====== ==========
    Algebra I            8:00  C01       Mr. Reyes   B24    Math
    Geometry             10:00 C01       Mr. Reyes   B24    Math
    World History        9:00  C15       Ms. Tan     A11    Humanities
    English Literature   10:00 C09       Ms. Larsen  A05    Humanities
    Physics              11:00 C06       Ms. Musa    B22    Science
    Chemistry            13:00 C17       Ms. Musa    B22    Science
    Music                9:00  C25       Mr. Pal     A03    Humanities
    ==================== ===== ========= =========== ====== ==========

This relation also exhibits redundancy.  We are given the facts that Mr. Reyes is in the Math department and his office is in room 124 multiple times, for example.  Note that Mr. Reyes appearing multiple times is not itself redundancy, as each appearance is a new fact: Mr. Reyes teaches Algebra I, and Mr. Reyes teaches Geometry.  We do not have any NULLs in this example, but we will see shortly how they might appear when we add or remove data.

Insert anomaly
##############

Consider what happens when a new instructor joins the faculty at the school.  Mr. Hassan is a new faculty member in the department of Math, but he has not yet been assigned any classes to teach.  How should we add Mr. Hassan to the database?  There are a few options, but none of them are good - whatever we do, we must provide values for **class_name**, **classroom**, and **time** in our new tuple.  We might think that NULL is the best choice for each of these attributes, as otherwise we must create fake class information.  Either way, we are making trouble for ourselves later on - our database now contains a tuple that must be handled in a special fashion, different from the other tuples in the relation.

Delete anomaly
##############

Now, consider what happens when Ms. Tan takes a job at another school.  If we are incautious, we will delete the tuple for Ms. Tan - removing World History from our list of classes altogether.  Similarly, we cannot remove English Literature from the list of classes without removing all information about Ms. Larsen.  The only way to avoid these issues with our current database design is to replace the existing values with NULLs.  As with the insert anomaly example, this leaves us with tuples that require special handling.

Update anomaly
##############

Update anomalies are a direct consequence of the redundancy in our database.  Consider what happens when Mr. Reyes changes offices.  If we are incautious, we will update the tuple listing Mr. Reyes as the teacher of Algebra I, but forget to update the tuple for Geometry, leaving our data internally consistent.  Mr. Reyes will be listed as having two different offices, without any indication which is correct.  To avoid trouble, we must remember to always update *all* classes for which Mr. Reyes is the instructor.

Example solution
----------------

Normalizing the **classes** relation will prevent each of the situations above.  In effect, normalization requires us to structure relations such that the data is expressed in a very simple and consistent form.  We typically achieve normalization by *decomposing* a relation into multiple smaller relations.

Informally, in a normalized relation, some thing or concept is uniquely identified by a primary key composed of one or more attributes and every other attribute represents a *single-valued* fact about the thing or concept only. For our example, the **classes** relation describes classes; it has **class_name** as a primary key, and **classroom**, **time**, and **instructor** as single-valued attributes.  (An example of an attribute that is not single-valued would be a list of students in the class.  We call such an attribute *multi-valued*.)  However, **office** and **department** are not really facts about a class; instead, they are facts about instructors.  These extended facts need to be removed to another relation through decomposition of the **classes** relation:

.. table:: **classes**
    :class: lined-table

    ==================== ===== ========= ===========
    class_name           time  classroom instructor
    ==================== ===== ========= ===========
    Algebra I            8:00  C01       Mr. Reyes
    Geometry             10:00 C01       Mr. Reyes
    World History        9:00  C15       Ms. Tan
    English Literature   10:00 C09       Ms. Larsen
    Physics              11:00 C06       Ms. Musa
    Chemistry            13:00 C17       Ms. Musa
    Music                9:00  C25       Mr. Pal
    ==================== ===== ========= ===========

.. table:: **instructors**
    :class: lined-table

    =========== ====== ==========
    name        office department
    =========== ====== ==========
    Mr. Reyes   B24    Math
    Ms. Tan     A11    Humanities
    Ms. Larsen  A05    Humanities
    Ms. Musa    B22    Science
    Mr. Pal     A03    Humanities
    =========== ====== ==========

Note how we have eliminated redundancy through this decomposition.  If we need to update office information for an instructor, there is exactly one tuple to update.  We also no longer need to worry about modification anomalies.  Adding or removing an instructor is completely independent of adding or removing classes; this also removes any need for excessive NULLs in the **classes** relation [#]_.

Normal forms
::::::::::::

The concept of normalization originates with the relational model itself.  Additional refinements have been added over time, leading to a series of normal forms, which mostly build on earlier normal forms.  We will not study every normal form that has been proposed, but focus on the forms which are most useful and most likely to be of value in most applications.  The first form we consider is appropriately named the *first normal form*, abbreviated 1NF.  We proceed with the second, third, and fourth normal forms (2NF, 3NF, 4NF) as well as Boyce-Codd normal form (BCNF), which fits in between 3NF and 4NF.  We briefly mention fifth normal form.

When a database meets the requirement for a normal form, we say that the database is *in* the form.  As commonly defined, most normal forms include a requirement that earlier normal forms are also met.  Therefore, any database that is in 4NF is necessarily also in 1NF, 2NF, 3NF, and BCNF; a database in BCNF is also in 3NF and below; and so forth.  However, it is also true that higher forms address less frequently occurring situations, so, for example, a database that has been restructured to be in 3NF is very likely to also be in BCNF or even 4NF or 5NF.  3NF is generally considered the minimum requirement a database must meet to be considered "normalized".

To explain most of the normal forms, we first need to provide some additional foundation, covered in the next few sections.  However, we can explain 1NF immediately.  1NF requires that the domain of an attribute of a relation contains *atomic* values only.  Atomic here simply means that we cannot usefully break the value down into smaller parts.  Non-atomic elements include compound values, arrays of values, and relations.  For example, a character string containing an author's name may be atomic [#]_, but a string identifying a book by author and title is probably compound; a list of authors would be an array; and a table of values giving a book's publication history (including publisher, year, ISBN, etc. for each publication) would be a relation.  To meet the 1NF requirements, compound values should be broken into separate attributes, while arrays and relations should be broken out into their own relations (with a foreign key referencing the original relation).

1NF is often described as simply part of the definition of a relational database, and early relational database systems indeed provided no capabilities that would permit violations of 1NF.  Some modern database systems now provide support for compound values, in the form of user-defined types, and array values.  While 1NF technically remains a requirement for all higher normal forms, for certain applications these violations of 1NF may be highly useful.  Some authors have argued for permitting relation-valued attributes as well.

Keys and superkeys
::::::::::::::::::

This section reiterates some material from :numref:`Chapter {number} <relational-model-chapter>` in which we defined the term *key*, but in a bit more detail.  We start by defining a more general term, *superkey*.

A superkey of a relation is some subset of attributes of the relation which uniquely identifies any tuple in the relation.  Consider the **books** relation below:

.. table:: **books**
    :class: lined-table

    =============== ========================== ==== ================ ============ ============
    author          title                      year genre            author_birth author_death
    =============== ========================== ==== ================ ============ ============
    Ralph Ellison   Invisible Man              1952 fiction          1914-03-01   1994-04-16
    Jhumpa Lahiri   Unaccustomed Earth         2008 fiction          1967-07-11
    J.R.R. Tolkien  The Hobbit                 1937 fantasy          1892-01-03   1973-09-02
    Isabel Allende  The House of the Spirits   1982 magical realism  1942-08-02
    J.R.R. Tolkien  The Fellowship of the Ring 1954 fantasy          1892-01-03   1973-09-02
    =============== ========================== ==== ================ ============ ============

(The blank entries for **author_death** in this table represent NULLs.  The authors are still living at the time of this writing.)

We assert that the set of attributes {**author**, **title**, and **year**} is a superkey for the **books** relation.

The definition of superkey applies not just to the current data in the relation, but to *any data we might possibly store in the relation*.  That is, a superkey is not a transitory property of the data, but a constraint we impose on the data.  For example, although each publication year listed in the **year** column above is unique to its book, that cannot be guaranteed for future books we might add to the relation.  Therefore the set {**year**} is not a superkey for **books**.

A second, and equivalent definition of superkey is as a subset of attributes of the relation that are guaranteed to contain a unique setting of values for any tuple in the relation.  For our example, this means there can never be two books in the **books** relation which share the same author, title, and year.  From this second definition and the definition of a relation, we note that *every* relation has at least one superkey: the set of all attributes of the relation.  The set {**author**, **title**, **year**, **genre**, **author_birth**, **author_death**} is a superkey for the **books** relation simply because every tuple in the relation must be unique.

We can further state that any subset of attributes of the relation which is a superset of some superkey of the relation is also a superkey of the relation.  For our example, {**author**, **title**, **year**, **author_birth**} must be a superkey because it is a superset of a known superkey.

A *key* of a relation is a superkey of the relation from which we cannot subtract any attributes and get a superkey.  For the **books** relation, we assert that {**author**, **title**} is a superkey of the relation; furthermore, both **author** and **title** are needed.  That is, neither {**author**} nor {**title**} is a superkey of **books**.  Therefore, {**author**, **title**} is a key of **books**; the set {**author**, **title**, **year**} is a superkey but not a key because we can remove **year** and still have a superkey [#]_.

Identifying the keys of a relation is a key step in analyzing whether or not a relation is already normalized with respect to 2NF or higher.

Functional dependencies
:::::::::::::::::::::::

Now we turn to the topic of functional dependencies, which are closely related to superkeys.

A *functional dependency* (FD) is a statement about two sets of attributes of a relation.  Consider two sets of attributes, which we will label *X* and *Y*.  We say that *X* *functionally determines* *Y*, or *Y* is *functionally dependent on* *X*, if, whenever two tuples in the relation agree on the values in *X*, they must also agree on the values in *Y*.  The notation for this is:

.. math::
    X \rightarrow Y

As with keys, FDs are constraints that we impose on the data.  Another way of thinking about a functional dependency is, if you had a relation such that the relation contains only the attributes that are in *X* or *Y*, then *X* would be a superkey for that relation.  That is, *X* uniquely determines everything in the union of *X* and *Y*.  (We can now provide another defintiion of superkey as a subset of attributes of a relation that functionally determines the set of all attributes of the relation.)

Another way of thinking about FDs is, if *X* functionally determines *Y*, then if we know the values in *X*, we know or can determine the values in *Y*, because *Y* just contains single-valued facts about *X*.  In our **books** relation, the set {**author**, **title**} functionally determines the set {**year**}, because if we know the author and title of the book, then we should be able to find out what the publication year is; and whatever sources we consult to find the year should all give us the same answer.  The dependency is "functional" in this sense; there exists some *function* between the domain of (author, title) pairs and the domain of publication years that yields the correct answer for every valid input.  The function in this case is simply a mapping between domains, not something we can analytically derive.

Here are some more FDs for the **books** relation:

.. math::

    \begin{eqnarray*}
    \text{\{author, title\}} & \rightarrow & \text{\{genre\}} \\
    \text{\{author\}} & \rightarrow & \text{\{author_birth, author_death\}} \\
    \text{\{author, title\}} & \rightarrow & \text{\{title, year\}} \\
    \text{\{title, genre\}} & \rightarrow & \text{\{title\}} \\
    \end{eqnarray*}

The first FD tells us that each book is categorized into exactly one genre in our database.  The second tells us that an author's dates of birth and death should be the same every time the author appears in the database.  The last two FDs are different from the previous ones; in these, there is an overlap between the set on the left-hand side and the set on the right-hand side.  We will give special names to these in a moment.  For now, the third FD tells us that, if we know the author and title of a book, then we know the title and the publication year.  The final FD simply tells us that any two tuples having the same title and genre, have the same title!

Types of functional dependency
------------------------------

FDs are categorized into three types: *trivial*, *non-trivial*, and *completely non-trivial*.

A trivial FD is one in which the right-hand side of the FD is a subset of the left-hand side.  The last FD in our example above is a trivial FD.  A trivial FD conveys no useful information - it tells us "we know what we know" - but they still have some use to us in our normalization procedures.  Every trivial FD we can write down for a relation is true, as long as the left-hand side of the FD is a subset of the attributes of the relation.

A non-trivial FD is one in which some part of the right-hand side of the FD is not in the left-hand side.  The intersection of the left-hand side and the right-hand side is not empty, but the right-hand side is not a subset of the left-hand side.  The third FD above is a non-trival FD.  These FDs convey some new information.

As you might guess by now, a completely non-trivial FD is one for which there is no overlap between the left-hand side and the right-hand side - the intersection of the two sets is the empty set.  The first two FDs above are completely non-trivial.  Identifying completely non-trivial FDs in our relations is a crucial step in normalization.

Inference rules
---------------

Many FDs can be inferred or derived from other FDs.  We are particularly interested in non-trivial FDs which have a maximal set on the right-hand side, that is, a set which cannot be added to without making the FD false.  There is a straightforward algorithm to infer such FDs from a set of FDs, which we discuss in the next section.  We need the five inference rules below for the algorithm.  The first three inference rules are known as *Armstrong's axioms*, and can be used to prove the remaining rules.

We present these without proof, but the intuition behind these should be clear.  Let *X*, *Y*, and *Z* be subsets of the attributes of the same relation.  Let the union of *Y* and *Z* be denoted *YZ*.  Then we have:

*Reflexive rule*
    If Y is a subset of X, then

    .. math::

        X \rightarrow Y

    This is simply a statement that all trivial FDs are true.

*Augmentation rule*
    If

    .. math::

        X \rightarrow Y

    then

    .. math::

        XZ \rightarrow YZ

    also holds.

    This rule says we can add the same attributes to both the left-hand and right-hand sides of an FD.  Trivially, if we add *Z* to what we know (left-hand side), then we should be able to determine *Z* in addition to what we could determine previously (right-hand side).

    In our **books** example, we are given

    .. math::

        \text{\{author\}} \rightarrow \text{\{author_birth, author_death\}}

    therefore, it is also true that

    .. math::

        \text{\{author, genre\}} \rightarrow \text{\{author_birth, author_death, genre\}}

    A special case of this is that we can add the left-hand side to both sides; this leaves the left-hand side unchanged, since the union of any set with itself is just the set:

    .. math::

        X \rightarrow Y

    implies

    .. math::

        X \rightarrow XY


*Transitive rule*
    If we have both of

    .. math::

        X \rightarrow Y \\
        Y \rightarrow Z

    then

    .. math::

        X \rightarrow Z

    also holds.  That is, if knowing *X* tells us *Y*, and from *Y* we can know *Z*, then knowing *X* also tells us *Z*.

    We do not have a direct example from our **books** relation to use; but imagine that we augment our relation with a **store_section** attribute, used by some bookstore.  The **store_section** attribute indicates the part of the store in which a book can be found.  If we then assert that

    .. math::

        \text{\{genre\}} \rightarrow \text{\{store_section\}} \\
        \text{\{author, title\}} \rightarrow \text{\{genre\}}

    then

    .. math::

        \text{\{author, title\}} \rightarrow \text{\{store_section\}}

*Splitting rule (or decomposition, or projective, rule)*
    If

    .. math::

        X \rightarrow YZ

    holds, then so do

    .. math::

        X \rightarrow Y \\
        X \rightarrow Z

    Plainly stated, if knowing the values for *X* tells us the values for *Y* **and** *Z*, then knowing the values for *X* tells us the values for *Y*, and likewise for *Z*.  In our **books** example, we have

    .. math::

        \text{\{author\}} \rightarrow \text{\{author_birth, author_death\}}

    therefore, it is also true that

    .. math::

        \text{\{author\}} \rightarrow \text{\{author_birth\}} \\
        \text{\{author\}} \rightarrow \text{\{author_death\}}

    Note that we can "split" the right-hand side only.  For example, given :math:`\text{\{author, title\}} \rightarrow \text{\{year\}}`, it is **not** true that :math:`\text{\{author\}} \rightarrow \text{\{year\}}`.

*Combining rule (or union, or additive, rule)*
    This is the splitting rule in reverse.  If we have both of

    .. math::

        X \rightarrow Y \\
        X \rightarrow Z

    then

    .. math::

        X \rightarrow YZ

    also holds. In our **books** example, we have

    .. math::

        \text{\{author, title\}} \rightarrow \text{\{year\}} \\
        \text{\{author, title\}} \rightarrow \text{\{genre\}}

    thus

    .. math::

        \text{\{author, title\}} \rightarrow \text{\{year, genre\}}

While any FDs that can be inferred from a given collection of FDs on a relation can be inferred using the above rules, there is unfortunately no way of deciding that some collection of FDs is, in fact, *complete* - that is, that the collection of FDs lets us infer every possible true FD on the relation.  FDs come from the minds of the database designer and others involved in analysis and design, a process which requires some "trial and error", i.e., iterative improvement.

Closure
-------

As mentioned, we are going to be particularly interested in non-trivial FDs which have a maximal set on the right-hand side.  The *closure* of a subset *X* of relation *R* given some collection of FDs is the union of all sets {*a*} such that *a* is an attribute of *R* and we can infer :math:`X \rightarrow {a}`.  Informally, the closure of *X* is the set of attributes which are functionally determined by *X*.  The closure of *X* is denoted *X*:sup:`+`.

We are interested in closure for a couple of reasons.  First, note from this definition that the closure of a superkey of a relation is the set of all attributes of the relation.  We can use this fact to test whether or not some set of attributes is a superkey; further, we could in theory find all superkeys of a relation by examining the closure of every subset of attributes (in practice this can become too much work fairly quickly as the number of attributes increases).  Second, closure will be useful in the decomposition step of our normalization algorithms.

The closure of a set of attributes can be determined using the following algorithm.

*Closure algorithm*
    Given a collection *F* of FDs and a set of attributes *X*:

    1. Let *C* = *X*.  Trivially, :math:`X \rightarrow C`.
    2. While there exists some functional dependency :math:`Y \rightarrow Z` in *F* such that *Y* is a subset of *C* and *Z* contains some attributes not in *C*, add the attributes in *Z* to *C* to create *C\'*.  Then,

       .. math::

          \begin{eqnarray*}
          & & C \rightarrow Y    ~~\text{(reflexive rule)} \\
          & & C \rightarrow Z    ~~\text{(transitive rule)} \\
          & & X \rightarrow Z    ~~\text{(transitive rule)} \\
          & & X \rightarrow C'   ~~\text{(combining rule)} \\
          \end{eqnarray*}

       Let *C* = *C\'*.

    3. When no more FDs meet the criteria above, *C* = *X*:sup:`+`.

We previously asserted that the set {author, title} is a superkey for our example **books** relation, so the closure {author, title}\ :sup:`+` should be the set of all attributes of **books**.  We now show that this follows from our inference rules, and from the FDs given previously:

.. math::

    \begin{eqnarray*}
    \text{\{author, title\}} & \rightarrow & \text{\{year\}} \\
    \text{\{author, title\}} & \rightarrow & \text{\{genre\}} \\
    \text{\{author\}} & \rightarrow & \text{\{author_birth, author_death\}} \\
    \text{\{author, title\}} & \rightarrow & \text{\{title, year\}} \\
    \text{\{title, genre\}} & \rightarrow & \text{\{title\}} \\
    \end{eqnarray*}

1. Let *C* = {author, title}.
2. We have :math:`\text{\{author, title\}} \rightarrow \text{\{year\}}`, and {author, title} is a subset of *C*, so add **year** to *C*: *C* = {author, title, year}.
3. Similarly, :math:`\text{\{author, title\}} \rightarrow \text{\{genre\}}`, so let *C* = {author, title, year, genre}.
4. We have :math:`\text{\{author\}} \rightarrow \text{\{author_birth, author_death\}}`, and {author} is a subset of *C*.  Let *C* = {author, title, year, genre, author_birth, author_death}.
5. The algorithm completes at this point because the right-hand sides of all of the unused FDs are already subsets of *C*; and in any case, *C* already has all attributes of **books**.

Thus, {author, title}\ :sup:`+` = {author, title, year, genre, author_birth, author_death}.


Second, third, and Boyce-Codd normal forms
::::::::::::::::::::::::::::::::::::::::::

- 2NF
- 3NF/BCNF
- when to relax


Decomposition requirements
::::::::::::::::::::::::::

- lossless join
- dependency preservation


- 1NF - author, author awards, book title, ...
- 2NF - author, book title, author birth/death, ... (author -> author dates)
- 3NF & BCNF - book_id, author, author dates, book title, ...?
- relaxation: award, year, format -> book and book -> format
- 4NF - author, books, author awards?
- higher


Multivalue dependencies and fourth normal form
::::::::::::::::::::::::::::::::::::::::::::::

Normalization in database design
::::::::::::::::::::::::::::::::

Databases can be created using a number of approaches.  The approach taken depends greatly on the circumstances which have led to the need for a database.

In some cases, data have been previously collected and stored in some fashion, but not organized into something we would consider a database.  Many scientific, industrial, and business processes produce large amounts of data in the form of sensor readings, application logs, reports, and form responses.  This data may exist in electronic form or on paper.  There may be little structure to the data; it may exist in a *flat* form in which there is only one type of record which stores every piece of information relevant to some event.  Creating a database to more efficiently work with such data may be best accomplished using a top-down approach, in which relations (or tables) are systematically decomposed.  Data modeling (:numref:`Part {number} <data-modeling-part`) may be used as part of this process, to document, communicate, and reason about the evolving database.

In contrast, when creating a new software application, a bottom-up approach may be preferred.  The application developers and other interested parties work to identify the data that need to be collected and stored.  A multitude of relations will arise naturally, corresponding to different parts of the application.  Data modeling should almost always occur early in this process.

Data modeling is very effective at producing relations that accurately represent independent concepts and the relationships between them.  However, some relations may still require normalization.  Normalization provides a different perspective on database design.  As with data modeling, our understanding of the real world and our data informs our choices.  However, while data modeling focuses on mapping concepts in the real world to relations, normalization works to produce a database structure that is more resistant to data errors.  Data modeling ensures our database accurately captures the data we need, while normalization ensures our database can be used effectively.  The two activities are thus complementary.  Whether or not normalization is applied formally, an understanding of normalization and its trade-offs is important for any database designer.

Trade-offs
::::::::::

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

----

**Notes**

.. [#] We may need to use NULLs when information is truly unknown or absent; for example, we would set the **instructor** attribute NULL for classes which have no instructor assigned at the current time.  Similarly, we might set the **office** attribute NULL for new instructors who do not yet have an office.  Neither of these cases requires special handling in our software, so we consider these NULLs acceptable.  While it is possible to design a database that avoids even these NULLs, it would complicate the database (with more relations) for little gain.

.. [#] It is common practice in some countries where English is the primary language to break a name into first (or given), middle, and last name (or surname).  However, this naming scheme is by no means universal, even for English speakers.  Unless there is a compelling need to break a name into components for your application, we recommend a single name attribute.  For more on this topic, see https://www.w3.org/International/questions/qa-personal-names.

.. [#] We reiterate that a superkey is a constraint *we impose* on the data.  When designing a database, we of course hope to create a structure that accommodates true facts from the world, but a) we sometimes fail due to incomplete information about the world, and b) we sometimes compromise on a simpler design that accommodates *most* facts from the world.  For our **simple_books** relation, we are unaware of any books by the same author with the same title in the same year, so we are comfortable asserting that {**author**, **title**, **year**} is a valid superkey (but we could be wrong).  On the other hand, this design intentionally fails to capture any number of the complexities of books in the real world.  For one example, authors occasionally re-publish a book with small changes, under the same title, years after the original publication.  Is this the same "book" (in which case **year** really stands for "year of first publication"), or a different book (in which case {**author**, **title**} is *not* a superkey)?  For another example, we are completely ignoring the fact that some books have multiple authors, and some have no known authors.  Databases about books can be very complex!

|license-notice|
