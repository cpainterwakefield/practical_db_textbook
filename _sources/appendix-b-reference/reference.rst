====================
SQL Reference Tables
====================

This appendix contains tables providing documentation on various aspects of the SQL standard (2016).  Where possible, variations from the standard by various database implementations (SQLite, PostgreSQL, MySQL, Oracle, Microsoft SQL Server) are shown.  Note that documentation of this sort is a moving target, so the information below may be out-of-date when you read it.  Consult your database vendor's documentation for current information.


Operators and functions
:::::::::::::::::::::::

Mathematical operators and functions
------------------------------------

Unless otherwise noted, the operands or parameters below can be any numeric type.

================== ===================== ================================ ===========================================
operator/ function meaning               usage                            notes
================== ===================== ================================ ===========================================
\+                 addition              *x* + *y*
\-                 subtraction           *x* - *y*
\*                 multiplication        *x* * *y*
/                  division              *x* / *y*
ABS                absolute value        ABS(*x*)
MOD                modulus (remainder)   MOD(*x*, *divisor*)              integers only in standard SQL
LOG                logarithm             LOG(*base*, *x*)                 in SQL Server, use LOG(*x*, *base*)
LN                 natural logarithm     LN(*x*)                          in SQL Server, use LOG(*x*)
LOG10              base-10 logarithm     LOG10(*x*)                       in Oracle, use LOG(10, *x*)
EXP                exponential function  EXP(*x*)
POWER              raise to power        POWER(*base*, *x*)
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

Most distributions provide additional non-standard functions and operators; for example, most include some mechanism for generating random numbers.  Note that SQLite provides very few of the functions listed above "out of the box", but there is an extension library available that implements them; most are available in the SQLite implementation used by this book.



Character string operators and functions
----------------------------------------

Below is a partial listing of operators and functions acting on character strings, omitting some less frequently implemented functions and some less frequently used optional parameters.  The 2016 SQL standard defines functions using three different pattern-matching languages: the one used by the operator **LIKE** (discussed in `Chapter 2`_), and two different regular expression (regex) languages; however none of the databases considered for this book conform to the 2016 standard with respect to most of these functions.  Many implementations do, however, provide functions with similar effect, but under a different name and using different regex languages.  These functions are therefore omitted, but you are encouraged to read the documentation for your database to see what options are available to you.

.. _`Chapter 2`: ../02-data-retrieval/data-retrieval.html

================== ================================== ================================================== ===========================================
operator/ function meaning                            usage                                              notes
================== ================================== ================================================== ===========================================
||                 concatenation                      *s* || *t*                                         in MySQL, use CONCAT(*s*, *t*); in SQL Server, use *s* + *t*
[NOT] LIKE         pattern comparison                 *s* [NOT] LIKE *pattern*
CHAR_LENGTH        length of string                   CHARACTER_LENGTH(*s*) or CHAR_LENGTH(*s*)          in SQLite and Oracle, use LENGTH(*s*); in SQL Server, use LEN(*s*)
POSITION           index of substring                 POSITION(*t* IN *s*)                               in SQLite and Oracle, use INSTR(*s*, *t*)
SUBSTRING          substring extraction               SUBSTRING(*s* FROM *start* FOR *length*)           in SQLite and Oracle, use SUBSTR(*s*, *start*, *length*); in SQL Server, use SUBSTRING(*s*, *start*, *length*)
UPPER              convert to uppercase               UPPER(*s*)                                         
LOWER              convert to lowercase               LOWER(*s*)
TRIM               remove leading/trailing characters TRIM([[LEADING|TRAILING|BOTH] [*t*] FROM] *s*)     If *t* is omitted, whitespace is trimmed; in SQLite, Oracle, and SQL Server use LTRIM, RTRIM and TRIM (varying usage)
OVERLAY            substring replacement              OVERLAY(*s* PLACING *t* FROM *start* FOR *length*) not in SQLite, Oracle, or SQL Server, but see REPLACE
================== ================================== ================================================== ===========================================

Most distributions provide additional non-standard functions and operators.


Miscellaneous functions
-----------------------

WIDTH_BUCKET
