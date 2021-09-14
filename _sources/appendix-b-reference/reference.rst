.. _appendix-b:

=========================
Appendix B: SQL Reference
=========================

This appendix contains documentation on various aspects of the SQL standard (2016).  This is not an exhaustive guide to the standard.  Where possible, variations from the standard by various database implementations (SQLite, PostgreSQL, MySQL, Oracle, Microsoft SQL Server) are shown.  Note that documentation of this sort is a moving target, so the information below may be out-of-date when you read it.  Consult your database vendor's documentation for current information.

In the documentation below, options may be indicated using square brackets.  For example, usage of the **SUBSTRING** function is documented as

::

    SUBSTRING(*s* FROM *start* [FOR *length*])

meaning both of the following expressions are valid:

::

    SUBSTRING('hello, world' FROM 1 FOR 5)
    SUBSTRING('hello, world' FROM 8)

(The first expression evaluates to ``'hello'``, and the second to ``'world'``.)

.. _appendix-b-data-types:

Data types
::::::::::

.. _appendix-b-number-types:

Number types
------------

The SQL standard provides for both exact (integers and fixed-precision decimal) and inexact (floating point) numbers.  Details of these types (e.g., the number of bits used in the number representation) are left up to individual implementations, so consult your database's documentation to fully understand its capabilities.

Integers
########

SQL defines three integer types: **INTEGER**, **SMALLINT**, and **BIGINT**.  Implementations of these types vary, but it is not uncommon for **INTEGER** (often abbreviated as **INT**) to store 32-bit integers, **SMALLINT** 16-bit integers, and **BIGINT** 64-bit integers.  Not all databases recognize all of these types, but **INTEGER** is recognized by all of the databases considered for this book.  Additional integer types may be available for your database system.

Exact decimal numbers
#####################

Decimal number types allow for exact storage of numbers that have digits to the right of the decimal point, e.g., 1234.56789.  These numbers are exact (compare to the floating point types below), and permit exact mathematical operations where possible (addition, subtraction, and multiplication).  The two defined types for SQL are **NUMERIC** and **DECIMAL**, which are synonyms of each other.  These types may be defined with parameters representing *precision* and *scale*, where precision is the number of significant digits that can be stored, and scale is the number of digits following the decimal point.  If the precision is given, but not the scale, the scale defaults to zero.

For example, in most implementations:

- **NUMERIC(3, 2)** defines a type that can store the values between -999.99 and 999.99, with a maximum of 2 digits past the decimal point.
- **NUMERIC(4)** defines a type that can store integers between -9999 and 9999.
- **NUMERIC** defines a type that can exactly store decimal values with implementation-defined precision and scale.

Different implementations behave differently when an attempt is made to store values with more digits than are allowed by the specified precision and scale.  This may result in an error, or (in the case of too many digits to the right of the decimal point), it may result in rounding or truncation of the value.

Floating point numbers
######################

Floating point number types allow for (possibly inexact) storage of real numbers, similar (or sometimes identical to) the `IEEE 754`_ specification.  The SQL standard defines the types **FLOAT**, **REAL**, and **DOUBLE PRECISION** (often abbreviated **DOUBLE**), but implementation of these types vary.

.. _`IEEE 754`: https://en.wikipedia.org/wiki/IEEE_754

Database support for number types
#################################

A summary of database support for number types is shown below (for the five databases this textbook attempts to cover):

================  ===================== ============================== ======== ================================== ================
Type              SQLite                PostgreSQL                     MySQL    Oracle                             SQL Server
================  ===================== ============================== ======== ================================== ================
INTEGER           yes                   yes                            yes      use NUMBER                         yes
SMALLINT          equivalent to INTEGER yes                            yes      use NUMBER                         yes
BIGINT            equivalent to INTEGER yes                            yes      use NUMBER                         yes
NUMERIC/DECIMAL   yes                   yes                            yes      use NUMBER                         yes
FLOAT             equivalent to REAL    equivalent to DOUBLE PRECISION yes      yes; but BINARY_DOUBLE recommended yes
REAL              yes                   yes                            yes      yes; but BINARY_DOUBLE recommended yes
DOUBLE PRECISION  equivalent to REAL    yes                            yes      yes; but BINARY_DOUBLE recommended use FLOAT
================  ===================== ============================== ======== ================================== ================


.. _appendix-b-string-types:

Character string types
----------------------

There are three main data types for storing character data in SQL.  Officially, these are named **CHARACTER**, **CHARACTER VARYING**, and **CHARACTER LARGE OBJECT**.  In addition, the modifier **NATIONAL** may be used to indicate strings containing data from locale-dependent character sets.  These names are fairly long and clunky, so databases typically use abbreviations or even completely different names for the same concepts.

The type **CHARACTER**, usually abbreviated as **CHAR**, is used for fixed-length strings.  The type **CHAR** is followed by parentheses enclosing the length of the string.  All values in a column of type **CHAR(4)**, for example, must contain exactly 4 characters.  In practice, many databases relax the "exactly" part of the definition and allow for shorter strings to be stored, although they may "pad" the value with trailing space characters.  Attempting to store strings longer than *n* usually results in an error.

**CHARACTER VARYING** is usually abbreviated as **VARCHAR**, and is used for strings of varying length up to some maximum, which must be specified just as with the **CHAR** type.  It is usually an error to attempt to store strings longer than the maximum.

**CHARACTER LARGE OBJECT** goes by many names, and is used to store strings of arbitrary length, up to some implementation-defined maximum (for example, Oracle's **CLOB** type allows strings of up to 128TB in some cases).  In many implementations, this type is limited in the operations or functions that may be used, and may not allow indexing.

A summary of database support for character strings is shown below:

=======================  ===================== ========== ======== =============== ================
Type                     SQLite                PostgreSQL MySQL    Oracle          SQL Server
=======================  ===================== ========== ======== =============== ================
CHARACTER(n)             equivalent to TEXT    yes        yes      yes             yes
CHARACTER VARYING(n)     equivalent to TEXT    yes        yes      use VARCHAR2(n) yes
CHARACTER LARGE OBJECT   equivalent to TEXT    use TEXT   use TEXT use CLOB        use VARCHAR(MAX)
=======================  ===================== ========== ======== =============== ================


.. _appendix-b-datetime-types:

Date and time types
-------------------

The SQL standard defines three or five principal types, depending on how you count.  The types are **DATE**, **TIME** (with or without time zone), and **TIMESTAMP** (with or without time zone).  If you specify simply **TIME** or **TIMESTAMP**, you get the version without time zones; append **WITH TIME ZONE** to additionally store time zone information.

- **DATE** values store dates in such a way that any particular day in history can be accurately recorded.  Typically the Gregorian calendar is supported, but some implementations will convert to and from Julian dates or other calendars.
- **TIME** represents a time of day, without reference to the date.  **TIME WITH TIME ZONE** includes information specifying the time zone relative to which the time should be evaluated.
- **TIMESTAMP** represents a precise moment in time, incorporating both the date and the time of day (with or without time zone).

A summary of database support for date and time types is shown below:

========================  ========================== ========== ======== ================================ ================
Type                      SQLite                     PostgreSQL MySQL    Oracle                           SQL Server
========================  ========================== ========== ======== ================================ ================
DATE                      use TEXT, REAL, or INTEGER yes        yes      yes                              yes
TIME                      use TEXT, REAL, or INTEGER yes        yes      no, use TIMESTAMP                yes
TIME WITH TIME ZONE       use TEXT, REAL, or INTEGER yes        no       no, use TIMESTAMP WITH TIME ZONE no
TIMESTAMP                 use TEXT, REAL, or INTEGER yes        yes      yes                              use DATETIME2
TIMESTAMP WITH TIME ZONE  use TEXT, REAL, or INTEGER yes        no       yes                              no
========================  ========================== ========== ======== ================================ ================

In addition to the date and time types, SQL defines a set of types known as *interval* types, where an interval represents a span of days or time between two date or time values.  Intervals are not covered in this book.


Operators and functions
:::::::::::::::::::::::

.. _appendix-b-comparison-operators:

Comparison operators
--------------------

Generally speaking, two non-NULL values of the same type can be compared, resulting in a Boolean value.  In certain cases, comparisons can made between different types, e.g., when both are numbers.  Numeric values are compared according to their algebraic values.  Date, time, and timestamp values are compared chronologically.  The Boolean value ``True`` is greater than ``False``.

Character string comparison is somewhat complex, as the comparison done depends on the *collation* rules in effect for the values; collation may depend on many factors including: the DBMS implementation, DBMS configuration parameters (such as the *locale*), operating system parameters, and any explicit collation settings for a given database table.  Collations may be used to implement proper sorting, for example, in a particular language context.  In general, if string *s* would appear in sorted (ascending) order prior to string *t*, then *s* \< *t*.

A comparison of any value with ``NULL`` results in ``NULL`` when using any of the operators in the table below.

=========== ========================= =========================== =============================================
operator    meaning                   usage                       notes
=========== ========================= =========================== =============================================
\=          equal to                  *x* \= *y*
\<\>        not equal to              *x* \<\> *y*                can also use != in most DBMSes (nonstandard)
\<          less than                 *x* \< *y*
\>          greater than              *x* \> *y*
\<=         less than or equal to     *x* \<= *y*
\>=         greater than or equal to  *x* \>= *y*
BETWEEN     range comparison          *x* BETWEEN *y* AND *z*     equivalent to *x* \>= *y* AND *x* \<= *z*
NOT BETWEEN exterior range comparison *x* NOT BETWEEN *y* AND *z* equivalent to NOT(*x* BETWEEN *y* AND *z*)
=========== ========================= =========================== =============================================

Comparison of ``NULL`` values requires special treatment; the expression ``NULL = NULL`` results in ``NULL``, not ``True``, and thus is not useful in testing for ``NULL``.  The **IS NULL** operator is provided for this purpose.  **IS NULL** (and the inverse, **IS NOT NULL**) expressions always result in ``True`` or ``False``.

Another standard SQL operator that has utility in the presence of ``NULL`` values are the binary operators **IS DISTINCT FROM** and **IS NOT DISTINCT FROM**.  These operators compare two values, treating ``NULL`` as if it were a special, distinct value, and always return ``True`` or ``False``.  Thus, the expression ``x IS NOT DISTINCT FROM y`` returns ``True`` if ``x = y`` evaluates to ``True`` or if *x* and *y* are both ``NULL``.  Of the databases considered for this book, only PostgreSQL implements **IS DISTINCT FROM** and **IS NOT DISTINCT FROM**.

The table below summarizes these operators.

==================== ============================ =========================================================== ================
operator             usage                        result                                                      notes
==================== ============================ =========================================================== ================
IS NULL              *x* IS NULL                  True if and only if *x* evaluates to NULL
IS NOT NULL          *x* IS NOT NULL              equivalent to NOT (*x* IS NULL)
IS DISTINCT FROM     *x* IS DISTINCT FROM *y*     equivalent to NOT (*x* IS NOT DISTINCT FROM *y*)            PostgreSQL only
IS NOT DISTINCT FROM *x* IS NOT DISTINCT FROM *y* True if *x* = *y* is true, or if *x* and *y* are both NULL  PostgreSQL only
==================== ============================ =========================================================== ================

Also see the `Boolean operators`_ section below for comparison operators that only apply to Boolean values.

.. _appendix-b-math-operators:

Mathematical operators and functions
------------------------------------

Unless otherwise noted, the operands or parameters below can be any numeric type.

================== ===================== ================================ ===========================================
operator/ function meaning               usage                            notes
================== ===================== ================================ ===========================================
\+                 addition              *x* + *y*
\-                 subtraction           *x* - *y*
\*                 multiplication        *x* * *y*
\/                 division              *x* / *y*
ABS                absolute value        ABS(*x*)
MOD                modulus (remainder)   MOD(*x*, *divisor*)              integers only in standard SQL
LOG                logarithm             LOG(*base*, *x*)                 in SQL Server, use LOG(*x*, *base*)
LN                 natural logarithm     LN(*x*)                          in SQL Server, use LOG(*x*)
LOG10              base-10 logarithm     LOG10(*x*)                       in Oracle, use LOG(10, *x*)
EXP                exponential function  EXP(*x*)
POWER              raise to power        POWER(*base*, *exponent*)
SQRT               square root           SQRT(*x*)
FLOOR              floor function        FLOOR(*x*)
CEILING            ceiling function      CEILING(*x*) or CEIL(*x*)
SIN                sine function         SIN(*x*)                         argument in radians
COS                cosine function       COS(*x*)
TAN                tangent function      TAN(*x*)
ASIN               inverse sine          ASIN(*x*)
ACOS               inverse cosine        ACOS(*x*)
ATAN               inverse tangent       ATAN(*x*)
SINH               hyperbolic sine       SINH(*x*)
COSH               hyperbolic cosine     COSH(*x*)
TANH               hyperbolic tangent    TANH(*x*)
================== ===================== ================================ ===========================================

Most database implementations provide additional non-standard functions and operators; for example, most include some mechanism for generating random numbers.

Mathematical expressions where one or more operands or inputs are ``NULL`` evaluate to ``NULL``.


.. _appendix-b-string-operators:

Character string operators and functions
----------------------------------------

Below is a partial listing of operators and functions acting on character strings, omitting some less frequently implemented functions and some less frequently used optional parameters.

The SQL standard defines several operators and functions making use of three different pattern-matching languages: the one used by the operator **LIKE** (discussed in `Chapter 3`_), and two different regular expression (regex) languages; however the databases considered for this book mostly do not conform to the standard with respect to these operators and functions.  Many implementations provide functions with similar effect, but under different names and using different regex languages.  These functions are therefore omitted, but you are encouraged to read the documentation for your database to see what options are available to you.

.. _`Chapter 3`: ../03-expressions/expressions.html

================== ================================== ================================================== ===========================================
operator/ function meaning                            usage                                              notes
================== ================================== ================================================== ===========================================
\||                concatenation                      *s* || *t*                                         in MySQL, use CONCAT(*s*, *t*); in SQL Server, use *s* + *t*
LIKE               pattern comparison                 *s* LIKE *pattern*                                 see `Chapter 3`_
NOT LIKE           inverse of LIKE                    *s* NOT LIKE *pattern*                             equivalent to NOT (*s* LIKE *pattern*)
CHAR_LENGTH        length of string                   CHARACTER_LENGTH(*s*) or CHAR_LENGTH(*s*)          in SQLite and Oracle, use LENGTH(*s*); in SQL Server, use LEN(*s*)
POSITION           index of substring                 POSITION(*t* IN *s*)                               in SQLite and Oracle, use INSTR(*s*, *t*)
SUBSTRING          substring extraction               SUBSTRING(*s* FROM *start* [FOR *length*])         in SQLite and Oracle, use SUBSTR(*s*, *start*, *length*); in SQL Server, use SUBSTRING(*s*, *start*, *length*)
UPPER              convert to uppercase               UPPER(*s*)
LOWER              convert to lowercase               LOWER(*s*)
TRIM               remove leading/trailing characters TRIM([[LEADING|TRAILING|BOTH] [*t*] FROM] *s*)     If *t* is omitted, whitespace is trimmed; BOTH is the default if LEADING etc. are omitted; in SQLite, Oracle, and SQL Server use LTRIM, RTRIM and TRIM (varying usage)
OVERLAY            substring replacement              OVERLAY(*s* PLACING *t* FROM *start* FOR *length*) not in SQLite, Oracle, or SQL Server, but see REPLACE
================== ================================== ================================================== ===========================================

Most database implementations provide additional non-standard functions and operators.

String operator or function expressions where the operands or inputs are ``NULL`` result in ``NULL``.


.. _appendix-b-boolean-operators:

Boolean operators
-----------------

The principal Boolean operators in SQL are **AND**, **OR**, and **NOT**.  Given operands that are strictly truth valued, i.e., not ``NULL``, these operators result in the logic operations they are named for.  That is, ``a AND b`` evaluates to ``True`` if and only if ``a`` and ``b`` are both ``True``, ``c OR d`` evaluates to ``True`` if either ``c`` or ``d`` are ``True``, and ``NOT e`` inverts the value ``e``.

However, since expressions resulting in Boolean values may also result in NULL (e.g., ``4 > NULL``), ``NULL`` is also a valid operand for the Boolean operators, and we can think of SQL as therefore having a 3-valued (rather than truly Boolean) logic.  The truth tables for **AND**, **OR**, and **NOT** are given below.  Treating ``NULL`` as meaning "unknown" in Boolean expressions, we can generally infer the result of a Boolean expression involving ``NULL``.  For example, ``True AND NULL`` must evaluate to ``NULL`` (meaning unknown), because the truth of the second operand is unknown.  On the other hand, ``True OR NULL`` must evaluate to ``True``, as it doesn't matter whether the second operand represents a true or a false value.

===== ===== =========== ==========
*a*   *b*   *a* AND *b* *a* OR *b*
===== ===== =========== ==========
True  True  True        True
True  False False       True
True  NULL  NULL        True
False True  False       True
False False False       False
False NULL  False       NULL
NULL  True  NULL        True
NULL  False False       NULL
NULL  NULL  NULL        NULL
===== ===== =========== ==========

===== =======
*a*   NOT *a*
===== =======
True  False
False True
NULL  NULL
===== =======

The SQL standard defines some less frequently used unary operators on Boolean values:  **IS [NOT] TRUE**, **IS [NOT] FALSE**, and **IS [NOT] UNKNOWN**, with **IS UNKNOWN** equivalent to **IS NULL** except that it only applies to the result of a Boolean expression.  So for example, SQL allows us to write ``NULL < 7 IS FALSE``, which would evaluate to ``False``.

SQL Server and Oracle do not implement **IS [NOT] TRUE**, **IS [NOT] FALSE**, and **IS [NOT] UNKNOWN**.  SQLite does not implement **IS [NOT] UNKNOWN**.

Some database implementations provide additional non-standard operators, such as **XOR**, **&** as an alternative to **AND**, etc.


.. _appendix-b-datetime-operators:

Date and time operators and functions
-------------------------------------

The SQL standard defines several basic operations relating **DATE**, **TIME** (with and without timezone), **TIMESTAMP** (with and without timezone), and **INTERVAL** data types.  (For a description of these data types, consult the section on `Data types`_ above.)

Comparison of like types is accomplished using the `Comparison operators`_ previously documented.  For example, **DATE** values can be compared with other **DATE** values, but not with **TIME**, **TIMESTAMP**, or **INTERVAL** values. (Behavior varies widely among the different database implementations - some do allow comparisons between types not allowed in the SQL standard.  However, it is generally inadvisable to compare different types, unless you know exactly how the comparison is being made!)

In addition, the mathematical operators *+*, *-*, *\**, and */* may be used as follows:

======== ========================= ======================== =====================
operator left operand              right operand            result type
======== ========================= ======================== =====================
\-       DATE, TIME, or TIMESTAMP  DATE, TIME, or TIMESTAMP INTERVAL
\+ or \- DATE, TIME, or TIMESTAMP  INTERVAL                 DATE, TIME, or TIMESTAMP
\+       INTERVAL                  DATE, TIME, or TIMESTAMP DATE, TIME, or TIMESTAMP
\+ or \- INTERVAL                  INTERVAL                 INTERVAL
\* or \/ INTERVAL                  number (INTEGER, etc.)   INTERVAL
\*       number (INTEGER, etc.)    INTERVAL                 INTERVAL
======== ========================= ======================== =====================

So, for example, a subtraction of one **TIMESTAMP** from another yields an **INTERVAL** representing the difference in days, hours, minutes, and seconds.

Other operators and functions involving dates and times:

===================== ============================================ ======================================
operator or function  meaning                                      usage
===================== ============================================ ======================================
CURRENT_DATE          evaluates to the current date                CURRENT_DATE
CURRENT_TIME          evaluates to the current time                CURRENT_TIME
CURRENT_TIMESTAMP     evaluates to the current date and time       CURRENT_TIMESTAMP
EXTRACT               get a date or time field from a date or time EXTRACT(*field* FROM *date/time/interval*), where *field* is e.g., 'YEAR', 'HOUR', etc.
OVERLAPS              test if one span of time overlaps another    *period1* OVERLAPS *period2*, where each *period* can be (*start date/time*, *end date/time*) or (*start date/time*, *interval*)
===================== ============================================ ======================================

Examples:

``EXTRACT('HOUR' FROM TIME '10:03:21')`` results in the integer ``10``.

``(DATE '2002-07-19', DATE '2003-01-31') OVERLAPS (DATE '2002-12-31', DATE '2005-05-05')`` results in a ``True``.

In actual practice, the databases considered for this book vary widely in their implementation of the SQL standard in regards to date and time types and operations on those types.  The variations are so great, we have not attempted to list departures from the standard in the above tables.  In most implementations, similar types can be compared, date and time types can be subtracted to yield intervals, and intervals can be added or subtracted to date and time types to yield a modified date or time.  Most databases implement **CURRENT_DATE**, **CURRENT_TIME**, and **CURRENT_TIMESTAMP**, or something similar.  Most implementations provide some function or functions replicating some of the functionality of **EXTRACT**.


Miscellaneous operators and functions
-------------------------------------

NULLIF

COALESCE

CASE

CAST

WIDTH_BUCKET
