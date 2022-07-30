.. _sql-vs-theory-chapter:

================================================
Differences between SQL and the relational model
================================================

The relational model of the database inspires and informs relational database systems.  However, the model differs from the actual implementation of most such systems.  The structured query language (SQL, covered in :numref:`Part {number} <sql-part>`) largely dictates the behavior of relational database systems, and therefore we frame this chapter as a comparison between SQL and the relational model.  This chapter is intended as a practical guide to these differences for students who have learned the relational model and must now learn SQL, or users of SQL learning the relational model.

Relations as sets
:::::::::::::::::

A key difference between SQL and the relational model regards the treatment of duplicate tuples in a relation.  Since relations are sets, in the model there can be no notion of "duplicate" tuples.  A tuple either is or is not in the relation, without multiplicity.  By contrast, SQL permits duplicates.  A SQL relation (or *table*) may contain multiple identical tuples (or *rows*) if it is not constrained with a primary key - we say that SQL relations are *multisets* rather than sets.  SQL queries may also return duplicate rows, for example, if a query asks for some subset of attributes (a projection) from a relation or a join of relations.

Duplicate tuples can be the source of serious errors if care is not taken.  As an example, consider a hypothetical database with the following relations: the first relation lists employees of a company with attributes including employee salary, the second lists projects in the company, while the third relation connects the first two relations in a many-to-many relationship.  Assuming that the primary key for the employee table is **employee_id** and the primary key for the project table is **project_id**, then the third relation contains only the attributes named **project_id** and **employee_id**, which are foreign keys on the attributes of the same name in the first two tables.  The existence of a tuple in this relation indicates that a particular employee works on a particular project.

Now, consider a query asking for the total employee salary costs to the company for a list of projects.  One approach to this problem using relational algebra or SQL involves a simple join of the three tables followed by projection onto the employee id and salary attributes.  The final answer is obtained by summing the salaries of the employees.  Using relational algebra, this produces the correct result, because each employee engaged in any of the projects of interest is listed only once.  Using SQL, however, the sum is incorrect if any employees work on more than one of the projects of interest, because those employees will be listed more than once in the join result.

There are, of course, multiple solutions to the hypothetical problem described above.  If the SQL query is restructured to use subqueries (using **IN**) instead of a join, or if the cross-reference relation is redesigned to include, say, a percentage of effort for an employee attributable to a particular project, then a more accurate result can occur.  However, this requires some effort on the designers and programmers of the SQL database to prevent a result that cannot happen using relational algebra.  The counter argument is that care must also be taken when applying relational algebra; suppose the join result is projected solely onto the employee salary attribute, which is, after all, the only value we need to sum.  In this case, we will obtain an incorrect result if two employees have the same salary!

The choice to permit duplicate tuples in query results is perhaps based on practical performance considerations.  To ensure uniqueness in a query result (e.g., when using the SQL **DISTINCT** operator), database systems typically sort or index the result, which adds cost to the operation.  If a query has a very large result (not uncommon in the modern era of "big data") the cost may become prohibitive.  While there are situations which permit the database engine to utilize indexes already available on the original tables involved in a query in order to produce a unique result efficiently, this is not possible for all queries in the general case. (One can posit that, had early database designers chosen to enforce uniqueness, technology and practical use would have evolved to largely erase these concerns; however, performance was a serious concern for the early relational databases in competition with more mature non-relational systems.)

Tuples and attributes
:::::::::::::::::::::

SQL also differs from the relational model regarding some aspects of tuples (or rows).  In the relational model, tuples are composed of values which are each associated with a specific named attribute and which are members of the domain of values for the attribute.  Tuples in the model may sometimes be treated as ordered lists of values, but only with the implicit understanding that each value is associated with an attribute as defined in some relation schema.  Finally, attribute names within a tuple must be unique.

In SQL, rows are treated somewhat more flexibly.  Rows stored in a relation (or table) generally conform to the above requirements on tuples.  However, rows returned from a query may not have well defined attributes associated with each value.  Where the query returns column values from some table, the values can be considered to be associated with a well defined attribute, following the definition of the original table.  SQL also permits queries to return values that are the results of operations or functions invoked on values; in most SQL database systems, those values will at least have an associated type, such as "integer" or "character string".  However, unless the query specifically provides a name for each such value, the database system may choose to provide a generic name or no name at all.  These anonymous values can be identified solely by their position in the row.  In some SQL systems, it is also possible to have multiple values associated with the same name.

These differences may not present serious difficulties in usual practice.  However, there is a sense in which such values lack the context necessary to give them meaning, and it is therefore possible to contrive scenarios in which problems arise.  For example, consider a company which awards bonuses as well as pay raises to its employees at the end of each year based on some complex mathematical formulas.  A SQL query may be used to calculate both values.  It would take only a momentary distraction for a developer programming the company's accounting software to reverse the intended application of each number if each number is identified solely by its columnar position within the returned rows.

Logical confusions
::::::::::::::::::

While not specifically a conflict between the relational model and SQL, SQL's implementation of three-valued logic using NULL can lead to some puzzling situations.  One of the more cited issues relates to *tautological* expressions - expressions which are logically always true.  For example, assuming some integer variable *x*, in the absence of NULL, the statement that "*x* is less than 17 or *x* is greater than or equal to 17" must be true.

However, if NULL is permitted and *x* is NULL, then comparing *x* to 17 using three-valued logic gives us the result *unknown*.  In SQL, the **OR** of two unknown values yields another unknown, rather than the expected true.  This outcome is partially a result of practical limitations; asking the SQL interpreter to reason about arbitrary logical expressions in order to reveal tautologies would impact the speed with which queries can be answered.  If our logical expectation is that the expression must be true in the relational model, then this represents a difference between SQL and the relational model.

It can be argued, though, that this is really a problem in how we assign meaning to NULL.  NULL may mean that *x* is some integer, but we do not know which integer it is.  However, NULL is also used for other meanings, such as that *x* is *not applicable*.  Consider a relation storing data on the inhabitants of some country where the relation includes an attribute for the person's passport number.  If a person never travels outside the country, they may not have a passport, and the passport number attribute has no meaning for the person.  In this case it is difficult to argue that an expression such as "the first digit of the person's passport number is less than 5 OR the first digit is greater than or equal to 5" should evaluate to true.

As an even simpler example of a problem with tautology, consider the SQL query below, which returns no rows in which NULL is assigned to the column *x*:

::

    SELECT * FROM some_table WHERE x = x;

The inclusion of NULL in the relational model is controversial at least in part due to confusions of this type.


|chapter-end|

|license-notice|
