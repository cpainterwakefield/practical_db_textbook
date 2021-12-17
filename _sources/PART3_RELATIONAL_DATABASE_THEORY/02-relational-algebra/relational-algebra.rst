.. _relational-algebra-chapter:

==================
Relational algebra
==================

In the last chapter, we introduced the relational model of the database, and defined the fundamental mathematical object in the model, the *relation*.  In this chapter, we discuss the *relational algebra*, which is the set of algebraic operations that can be performed on relations.  The relational algebra can be viewed as one mechanism for expressing queries on data stored in relations, and an understanding of relational algebra is important in understanding how relational databases represent and optimize queries.

A related topic, which we do not cover in this book, is the *relational calculus*.  The relational calculus provides another mathematical expression of queries on relations, and is equivalent in expressiveness to the relational algebra.

Unary operations
::::::::::::::::

The unary operations in the relational algebra act on one relation and result in another relation.  The unary operations are *selection*, *projection*, and *renaming*, and their associated operators are typically written as the Greek letters which match the starting letters of the operation:

- :math:`\sigma` (sigma): selection
- :math:`\pi` (pi): projection
- :math:`\rho` (rho): renaming

We will explore each of these unary operators in application to the relation **books** shown below:

.. table:: **books**
    :class: lined-table

    ======= ========= ============================ ====
    book_id author_id title                        year
    ======= ========= ============================ ====
    1       3         *The House of the Spirits*   1982
    2       1         *Invisible Man*              1952
    3       6         *The Hobbit*                 1937
    4       2         *Unaccustomed Earth*         2008
    5       6         *The Fellowship of the Ring* 1954
    6       4         *House Made of Dawn*         1968
    7       5         *A Wizard of Earthsea*       1968
    ======= ========= ============================ ====

Selection
---------

Selection applies a Boolean condition to the tuples in a relation.  The result of a selection operation is a relation containing exactly those tuples for which the selection condition evaluated to true.  For example, if we are interested in books published after 1960, we can write the selection operation to retrieve just those books as:

.. math::

    \sigma_{\text{year} > 1960}(\text{books})

The operator is written with the Boolean condition as a subscript, and then the operand (the input relation) is given in parentheses.  Note that the Boolean condition refers to an attribute of the **books** relation, comparing it to a constant value.  The result of this operation is a relation with the same schema as **books**, but with no name:

.. table::
    :class: lined-table

    ======= ========= ============================ ====
    book_id author_id title                        year
    ======= ========= ============================ ====
    1       3         *The House of the Spirits*   1982
    4       2         *Unaccustomed Earth*         2008
    6       4         *House Made of Dawn*         1968
    7       5         *A Wizard of Earthsea*       1968
    ======= ========= ============================ ====

Simple Boolean expressions in the relational algebra usually involve comparisons of an attribute with a constant, using any comparison operator.  More complex Boolean expressions can be constructed from simple expressions using **AND**, **OR**, and **NOT**.  For instance, if we are interested in books published after 1960 as well as books by the author with **author_id** equal to 6, we could write:

.. math::

    \sigma_{\text{year} > 1960 \text{ OR } \text{author_id} = 6}(\text{books})

Selection can result in a relation that has all of the tuples from the original (a relation equivalent to the original), some of the tuples from the original, or no tuples at all (an empty set).  In the case of a empty relation, we still consider the relation to have the same schema as the original relation.

Since the result of a selection is a relation, we can apply another selection to the result.  For example, we could find the books published after 1950, and then select from that result the books with **author_id** equal to 6:

.. math::

    \sigma_{\text{author_id} = 6}(\sigma_{\text{year} > 1950}(\text{books}))

This would give us a result with one tuple:

.. table::
    :class: lined-table

    ======= ========= ============================ ====
    book_id author_id title                        year
    ======= ========= ============================ ====
    5       6         *The Fellowship of the Ring* 1954
    ======= ========= ============================ ====

This composition of selection operations is equivalent to a single selection operation using a *conjunction* (AND) of the selection conditions:

.. math::

    \sigma_{\text{author_id} = 6 \text{ AND } \text{year} > 1950}(\text{books})

Projection
----------

The projection operation creates a new relation which has a subset of the attributes of the input relation.  We could use projection, for example, to get a set of tuples expressing just the title and publication year of our books.  We write the projection operator with the list of attribute names in the subscript, followed by the operand in parentheses:

.. math::

    \pi_{\text{title, year}}(\text{books})

This result contains a tuple for each tuple in **books**, but the tuples in the result only have the attributes specified by the projection operation, thus the result relation has a different schema from the original:

.. table::
    :class: lined-table

    ============================ ====
    title                        year
    ============================ ====
    *The House of the Spirits*   1982
    *Invisible Man*              1952
    *The Hobbit*                 1937
    *Unaccustomed Earth*         2008
    *The Fellowship of the Ring* 1954
    *House Made of Dawn*         1968
    *A Wizard of Earthsea*       1968
    ============================ ====

At first glance, it might seem the result of a projection will always have the same number of tuples as the input relation, but this is not the case.  Consider what happens if we project **books** onto the single attribute **year**.  There are two tuples in **books** with the same **year** value of 1968; when we project onto this single attribute, we get two identical tuples.  Since relations cannot contain duplicates, the result only has one tuple with **year** equal to 1968.  Thus, the result has *fewer* tuples than the input relation:

.. table:: :math:`\pi_{\text{year}}(\text{books})`
    :class: lined-table

    +------+
    | year |
    +======+
    | 1982 |
    +------+
    | 1952 |
    +------+
    | 1937 |
    +------+
    | 2008 |
    +------+
    | 1954 |
    +------+
    | 1968 |
    +------+

Since the result of projection is a relation, we can apply selection to the result:

.. math::

    \sigma_{\text{year}=1968}(\pi_{\text{title, year}}(\text{books}))

Note the order of operations here: first, we supply **books** as an input to the projection operation; second, the result of the projection is given as the input to the selection operation.

Similarly, since the result of a selection is a relation, we can apply projection after selection.  The above expression is equivalent to:

.. math::

    \pi_{\text{title, year}}(\sigma_{\text{year}=1968}(\text{books}))

The result in both cases is:

.. table::
    :class: lined-table

    ======================== ====
    title                    year
    ======================== ====
    *House Made of Dawn*     1968
    *A Wizard of Earthsea*   1968
    ======================== ====

It is important to note, however, that you cannot always change the order of projection and selection for an equivalent result.  Consider the following expressions:

.. math::

    \pi_{\text{title}}(\sigma_{\text{year}=1968}(\text{books}))

.. math::

    \sigma_{\text{year}=1968}(\pi_{\text{title}}(\text{books}))

In the first expression, we select the books which were published in 1968, and then project the resulting tuples onto the **title** attribute.  This result is:

.. table::
    :class: lined-table

    +-------------------------+
    | title                   |
    +=========================+
    | *House Made of Dawn*    |
    +-------------------------+
    | *A Wizard of Earthsea*  |
    +-------------------------+

However, the second expression is not a correct expression.  The projection occurs first, yielding a relation with just one attribute named **title**.  The following selection is then incorrect, because it makes reference to an attribute, **year**, which does not exist in the input relation.

Projection can also by applied to the result of another projection; however, the result is equivalent to just performing the second projection.  Compare:

.. math::

    \pi_{\text{title}}(\pi_{\text{title, year}}(\text{books}))

.. math::

    \pi_{\text{title}}(\text{books})

Note that we cannot change the order of the two projection operations in the first expression above, as the expression would then be incorrect.

Renaming
--------

The final unary operation allows for relations and their attributes to be renamed.  As we will see, this operation is primarily useful in eliminating name conflicts in certain binary operations - that is, in expressions involving two relations in which the name of some attribute is the same in both relations.  The general form of the renaming operator lets us provide new names for the relation and all of its attributes:

.. math::

    \rho_{\text{mybooks(b_id, a_id, title, year)}}(\text{books})

This results in a relation with the name **mybooks** with attributes **b_id**, **a_id**, **title**, and **year**.  The tuples of the new relation have the same values as the tuples of the old relation, but the values are associated with the new attribute names.

As in this example, it is not necessary to alter the name of every attribute (we left unchanged the attribute names **title** and **year**), but some name must be provided for every attribute.  A non-standard alternative notation allows us to rename only the attributes we want to change:

.. math::

    \rho_{\text{mybooks(book_id} \rightarrow \text{b_id, author_id} \rightarrow \text{a_id)}}(\text{books})

We can optionally leave out either the relation name or the list of attributes.  For example, the following expression is correct and results in a relation named **books** with attributes **book_id**, **author_id**, **title**, and **publication_year**:

.. math::

    \rho_{\text{(year} \rightarrow \text{publication_year)}}(\text{books})


Cross product and joins
:::::::::::::::::::::::

We now turn our attention to operations which extend tuples in one relation with tuples from another relation.  For this section, we will be using **books** and a second relation, **authors**:

.. table:: **books**
    :class: lined-table

    ========== ================== =========== ============
    author_id  name               birth       death
    ========== ================== =========== ============
    1          Ralph Ellison      1914-03-01  1994-04-16
    2          Jhumpa Lahiri      1967-07-11
    3          Isabel Allende     1942-08-02
    4          N\. Scott Momaday  1934-02-27
    5          Ursula K. Le Guin  1929-10-21  2018-01-22
    6          J.R.R. Tolkien     1892-01-03  1973-09-02
    7          Kazuo Ishiguro     1954-11-08
    ========== ================== =========== ============

Cross product
-------------

The cross product (or *Cartesian product*) of two relations **A** and **B** is a new relation containing all tuples that can be created by concatenating some tuple from **B** onto some tuple from **A** [#]_.  Here we are using the definition of tuple as an ordered list of values.  The attributes of the new relation are the attributes of **A** and **B** concatenated (however, if there is a name collision, e.g., if both **A** and **B** have some attribute **x**, we will disambiguate the attributes in the new relation by prepending the relation names, that is, the cross product will have attributes **A.x** and **B.x**; we can avoid having to do this if we first apply renaming to one relation or the other).

The cross product operator is denoted :math:`\times`, and is written between its two operands. To start, consider two rather abstract relations **S** and **T**:

.. table:: **S**
    :class: lined-table

    == ===
    u  v
    == ===
    1  one
    2  two
    == ===

.. table:: **T**
    :class: lined-table

    ======= ======== ======
    x       y        z
    ======= ======== ======
    green   3.1415   apple
    blue    2.71828  pear
    yellow  1.618    mango
    ======= ======== ======

We write the cross product of **S** and **T** as:

.. math::

    \text{S} \times \text{T}

which gives us the (unnamed) relation containing every pairing of a tuple from **S** with every tuple from **T**:

.. table::
    :class: lined-table

    == === ======= ======== =======
    u  v   x       y        z
    == === ======= ======== =======
    1  one green   3.1415   apple
    1  one blue    2.71828  pear
    1  one yellow  1.618    mango
    2  two green   3.1415   apple
    2  two blue    2.71828  pear
    2  two yellow  1.618    mango
    == === ======= ======== =======

From the definition, it is trivial to determine that the size of the cross product is the product of the sizes of the operands.

Join
----

The cross product is a fundamental operation in the relational algebra, but not a generally useful one when we consider actual data.  Consider the cross product of **books** and **authors**:

.. math::

    \text{books} \times \text{authors}

The full set of tuples in this relation is large (the number of books multiplied by the number of authors), so we only show a subset below:

.. table::
    :class: lined-table

    ======= =============== ============================ ===== ================== ================== =========== ============
    book_id books.author_id title                        year  authors.author_id  name               birth       death
    ======= =============== ============================ ===== ================== ================== =========== ============
    1       3               *The House of the Spirits*   1982  1                  Ralph Ellison      1914-03-01  1994-04-16
    1       3               *The House of the Spirits*   1982  2                  Jhumpa Lahiri      1967-07-11
    1       3               *The House of the Spirits*   1982  3                  Isabel Allende     1942-08-02
    2       1               *Invisible Man*              1952  1                  Ralph Ellison      1914-03-01  1994-04-16
    2       1               *Invisible Man*              1952  2                  Jhumpa Lahiri      1967-07-11
    2       1               *Invisible Man*              1952  3                  Isabel Allende     1942-08-02
    ======= =============== ============================ ===== ================== ================== =========== ============

The author of *The House of the Spirits* is Isabel Allende.  What meaning, then, can we make of a tuple that pairs *The House of the Spirits* with the author Ralph Ellison (the author of *Invisible Man*)?

We are typically interested in pairing only certain tuples of a relation with certain tuples of another.  In the above example, we are interested in tuples where the **author_id** attribute from **books** agrees with the **author_id** attribute from **authors**.  This is easily accomplished by applying the appropriate selection operation to the result of our cross product:

.. math::

    \sigma_{\text{books.author_id}=\text{authors.author_id}}(\text{books} \times \text{authors})

This yields a useful result:

.. table::
    :class: lined-table

    ======= =============== ============================ ===== ================== ================== =========== ============
    book_id books.author_id title                        year  authors.author_id  name               birth       death
    ======= =============== ============================ ===== ================== ================== =========== ============
    1       3               *The House of the Spirits*   1982  3                  Isabel Allende     1942-08-02
    2       1               *Invisible Man*              1952  1                  Ralph Ellison      1914-03-01  1994-04-16
    3       6               *The Hobbit*                 1937  6                  J.R.R. Tolkien     1892-01-03  1973-09-02
    4       2               *Unaccustomed Earth*         2008  2                  Jhumpa Lahiri      1967-07-11
    5       6               *The Fellowship of the Ring* 1954  6                  J.R.R. Tolkien     1892-01-03  1973-09-02
    6       4               *House Made of Dawn*         1968  4                  N\. Scott Momaday  1934-02-27
    7       5               *A Wizard of Earthsea*       1968  5                  Ursula K. Le Guin  1929-10-21  2018-01-22
    ======= =============== ============================ ===== ================== ================== =========== ============

Since this pattern of applying a selection after a cross product is so common, we have an operator that combines the two into an operation known as a *join* [#]_.  Using the join operator, the above expression becomes:

.. math::

    \text{books} \Join_{\text{books.author_id}=\text{authors.author_id}} \text{authors}

or, you can instead format the expression as:

.. math::

    \text{books} \underset{\text{books.author_id}=\text{authors.author_id}}\Join \text{authors}


Theta-join and equijoin
-----------------------

While an equality condition is typically used in joins, more generally any condition of the form

.. math::

    \text{A.x } \Theta \text{ B.y}

where **A.x** is an attribute from one relation, **B.y** is an attribute from the other relation, and :math:`\Theta` is a comparison operator (such as =, <, etc.) can be used.  A condition of this form is known as a *theta condition*, and a join using such a condition or a conjunction (AND) of such conditions is known as a *theta-join*.

A theta-join using only equality comparisons (as in our example above) is further known as an *equijoin*.

This terminology is not especially important in understanding the algebra, but is something you may encounter if you intend a deeper study of the relational algebra.


Natural join
------------

When we join **books** with **authors** we run into the issue that both relations contain an attribute named **author_id**.  Since a relation cannot have more than one attribute with the same name, joining (or taking a cross product of) these two relations requires us to rename the attributes in some fashion, either by an explicit renaming operation prior to joining, or by prepending the original relation name as we did in our example.  Because our join condition was equality on the **author_id** attributes, both the **books.author_id** and **authors.author_id** in the resulting relation always agree.  This unnecessary redundancy can be removed using projection and renaming.

In this special situation in which we wish to join specifically by equating the attributes with the same names in both relations, and subsequently remove the "duplicate" attributes, we can instead do a *natural join*.  We can indicate a natural join using the join operator with no conditions [#]_ as follows:

.. math::

    \text{books} \Join \text{authors}

which yields the simplified relation:

.. table::
    :class: lined-table

    ======= ========= ============================ ===== ================== =========== ============
    book_id author_id title                        year  name               birth       death
    ======= ========= ============================ ===== ================== =========== ============
    1       3         *The House of the Spirits*   1982  Isabel Allende     1942-08-02
    2       1         *Invisible Man*              1952  Ralph Ellison      1914-03-01  1994-04-16
    3       6         *The Hobbit*                 1937  J.R.R. Tolkien     1892-01-03  1973-09-02
    4       2         *Unaccustomed Earth*         2008  Jhumpa Lahiri      1967-07-11
    5       6         *The Fellowship of the Ring* 1954  J.R.R. Tolkien     1892-01-03  1973-09-02
    6       4         *House Made of Dawn*         1968  N\. Scott Momaday  1934-02-27
    7       5         *A Wizard of Earthsea*       1968  Ursula K. Le Guin  1929-10-21  2018-01-22
    ======= ========= ============================ ===== ================== =========== ============


Set operations
::::::::::::::

Unsurprisingly, given that relations are sets, the relational algebra includes the usual set operations *union*, *intersection*, and *set difference*, with some restrictions.  These binary operations are denoted by:

- :math:`\cup`: union
- :math:`\cap`: intersection
- :math:`-`: set difference

Given two relations **A** and **B**, the union :math:`\text{A} \cup \text{B}` is the set of all tuples that exist in **A**, or exist in **B**, or both.  The intersection :math:`\text{A} \cap \text{B}` is the set of all tuples that exist in both **A** and **B**.  Finally, the set difference :math:`\text{A} - \text{B}` is the set of all tuples that exist in **A** but do not exist in **B**.

For example, let **A** and **B** be the relations below:

.. table:: **A**
    :class: lined-table

    ======= ===
    x       y
    ======= ===
    apple   42
    orange  19
    cherry  77
    ======= ===

.. table:: **B**
    :class: lined-table

    ======== ===
    x        y
    ======== ===
    banana   8
    apple    42
    coconut  17
    ======== ===

Then we have:

.. table:: :math:`\text{A} \cup \text{B}`
    :class: lined-table

    ======== ===
    x        y
    ======== ===
    apple    42
    orange   19
    cherry   77
    banana   8
    coconut  17
    ======== ===

.. table:: :math:`\text{A} \cap \text{B}`
    :class: lined-table

    ======== ===
    x        y
    ======== ===
    apple    42
    ======== ===

.. table:: :math:`\text{A} - \text{B}`
    :class: lined-table

    ======== ===
    x        y
    ======== ===
    orange   19
    cherry   77
    ======== ===

.. table:: :math:`\text{B} - \text{A}`
    :class: lined-table

    ======== ===
    x        y
    ======== ===
    banana   8
    coconut  17
    ======== ===

Note that union and intersection are commutative, but set difference is not.

The important restriction on set operations in the relational algebra is that the relations must be compatible in terms of their schemas.  The meaning of "compatible" varies, but for our purposes, assume we view the tuples in a relation as ordered lists, where each position in the list is associated with a particular attribute and type domain.  Then, if we have two relations, we require that, for a given position in the tuples in either relation, the attribute and type domain are the same.  For **A** and **B** shown above, we might assert that the first position corresponds to attribute **x** and contains character strings, while the second position (**y**) contains integers.

A looser requirement allows attribute names (but not type domains) to differ between relations.  This requirement is less compatible with the second definition of tuple given in the previous chapter, while eliminating the occasional need for renaming operations prior to applying set operations. If the attribute names do not match in the two relations, we adopt the attribute names from the left-hand operand for the result relation.

While intersection is a useful operation, it is not strictly needed for the algebra, as the same result can be obtained using only union and difference:

.. math::

    \text{A} \cap \text{B} \equiv \text{A} - (\text{A} - \text{B})

Other operations and extensions
:::::::::::::::::::::::::::::::

Division
--------

Grouping and aggregation
------------------------

Outer joins
-----------

Operation sequences
:::::::::::::::::::


Expression trees
::::::::::::::::


  - division
- completeness of operators?
- mention of group/aggregation operations, outer joins
- expression trees


|chapter-end|

----

**Notes**

.. [#] This is consistent with the definition of the Cartesian product of sets of tuples in general mathematics.

.. [#] In fact, Codd's original relational model paper discusses joins and not cross products.  However, the cross product is now recognized as a more fundamental operation in the relational algebra.

.. [#] Some authors use * instead of the join operator without conditions.

|license-notice|
