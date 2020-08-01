====================
SQL Reference Tables
====================

This appendix contains tables providing documentation on various aspects of the SQL standard.  Where possible, variations from the standard by various database implementations (SQLite, PostgreSQL, MySQL, Oracle, Microsoft SQL Server) are shown.  Note that documentation of this sort is a moving target, so the information below may be out-of-date when you read it.  Consult your database vendor's documentation for current information.


Operators and functions
:::::::::::::::::::::::

Mathematical operators and functions
------------------------------------

Unless otherwise noted, the operands or parameters below can be any numeric type.

======== ===================== ================================ ===========================================
operator meaning               usage
======== ===================== ================================ ===========================================
\+       addition              *x* + *y*
\-       subtraction           *x* - *y*
\*       multiplication        *x* * *y*
/        division              *x* / *y*
ABS      absolute value        ABS(*x*)
MOD      modulus (remainder)   MOD(*x*, *divisor*)              integers only in standard SQL
LOG      logarithm             LOG(*base*, *x*)                 in SQL Server, use LOG(*x*, *base*)
LN       natural logarithm     LN(*x*)                          in SQL Server, use LOG(*x*)
LOG10    base-10 logarithm     LOG10(*x*)                       in Oracle, use LOG(10, *x*)
EXP      exponential function  EXP(*x*)
POWER    raise to the power    POWER(*base*, *x*)
SQRT     square root           SQRT(*x*)
FLOOR    floor function        FLOOR(*x*)
CEIL     ceiling function      CEIL(*x*)                     same as CEILING
SIN      sine function         SIN(*x*)                     argument in radians
COS      cosine function       COS(*x*)
TAN      tangent function      TAN(*x*)
ASIN     inverse sine          ASIN(*x*)
ACOS     inverse cosine        ACOS(*x*)
ATAN     inverse tangent       ATAN(*x*)
SINH     hyperbolic sine       SINH(*x*)
COSH     hyperbolic cosine     COSH(*x*)
TANH     hyperbolic tangent    TANH(*x*)
============ ===================== ================================ ===========================================

Most distributions provide additional non-standard functions and operators; for example, most include some mechanism for generating random numbers.  Note that SQLite provides very few of the functions listed above "out of the box", but there is an extension library available that implements them; most are available in the SQLite implementation used by this book.


Miscellaneous functions
-----------------------

WIDTH_BUCKET
