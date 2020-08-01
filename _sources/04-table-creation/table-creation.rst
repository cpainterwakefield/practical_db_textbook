====================
Basic Table Creation
====================

- very brief introduction to table creation
    - datatypes: stick with strings, integers, numeric, and dates?
    - no keys, defaults, auto increments, etc.

- reminder of how to display created tables (select name, sql from sqlite_master)


Data types in SQL
:::::::::::::::::

The SQL standard defines several basic data types with which columns can be associated.  While most of the basic types exist in all RDBMSes, actual implementation of the types varies quite a bit.  In particular, SQLite is dynamically typed, and you can store values of any type in any column no matter how the column is defined.  Please read the documentation for your RDBMS for details on data types supported and their implementations.

Numbers
-------

SQL provides support for several different types of number values, explained below.

Integers
########

SQL defines three integer types, which differ only in the range of values they can store: **INTEGER**, **SMALLINT**, and **BIGINT**.  Implementations of these types vary, but it is not uncommon for **INTEGER** (often abbreviated as **INT**) to store 32-bit integers, **SMALLINT** 16-bit integers, and **BIGINT** 64-bit integers.  Additional integer types may be available for your RDBMS.

Exact decimal numbers
#####################

Decimal number types allow for exact storage of numbers that have digits to the right of the decimal point, e.g., 1234.56789.  These numbers are exact (compare to the floating point types below), and permit exact mathematical operations where possible (e.g., for addition and subtraction).  The two defined types for SQL are **NUMERIC** and **DECIMAL**, which are typically just synonyms for each other.  These types may be defined with parameters representing *precision* and *scale*, where precision is the number of significant digits that can be stored, and scale is the number of digits following the decimal point.  If the precision is given, but not the scale, the scale defaults to zero.  Some implementations may also allow the omission of precision and scale, allowing for arbitrary numbers of digits before or after the decimal point (up to implementation-defined limits).

For example, in most implementations:

- **NUMERIC(3, 2)** defines a type that can store the values between -999.99 and 999.99, with a maximum of 2 digits past the decimal point.
- **NUMERIC(4)** defines a type that can store integers between -9999 and 9999.
- **NUMERIC** defines a type that can exactly store decimal values with arbitrary numbers of digits before and after the decimal point.

Different implementations behave differently when an attempt is made to store values with more digits than are allowed by the specified precision and scale.  This may result in an error, or (in the case of too many digits to the right of the decimal point), it may result in rounding or truncation of the value.

Decimal number types are particularly important for the storage of monetary records, where exact addition, subtraction, and multiplication is necessary.

Floating point numbers
######################

Floating point number types allow for (possibly inexact) storage of real numbers, similar (or sometimes identical to) the `IEEE 754`_ specification.  The SQL standard defines the types **FLOAT**, **REAL**, and **DOUBLE PRECISION** (often abbreviated **DOUBLE**), but implementation of these types vary.  These types can support extremely large and extremely small numbers, and are most useful in scientific and mathematical applications.

.. _`IEEE 754`: https://en.wikipedia.org/wiki/IEEE_754


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

Character string types
----------------------

There are three main data types for storing character data in SQL.  Officially, these are named **CHARACTER**, **CHARACTER VARYING**, and **CHARACTER LARGE OBJECT**.  In addition, the modifier **NATIONAL** may be used to indicate strings containing data from locale-dependent character sets.  These names are fairly long and clunky, so databases typically use abbreviations or even completely different names for the same concepts.

The type **CHARACTER**, usually abbreviated as **CHAR**, is used for fixed-length strings.  The type **CHAR** is followed by parentheses enclosing the length of the string.  All values in a column of type **CHAR(4)**, for example, must contain exactly 4 characters.  In practice, many databases relax the "exactly" part of the definition and allow for shorter strings to be stored, although they may "pad" the value with trailing space characters.  Attempting to store strings longer than *n* usually results in an error.

**CHARACTER VARYING** is usually abbreviated as **VARCHAR**, and is used for strings of varying length up to some maximum, which must be specified just as with the **CHAR** type.  It is usually an error to attempt to store strings longer than the maximum.

**CHARACTER LARGE OBJECT** goes by many names, and is used to store strings of arbitrary length, up to some implementation-defined maximum (for example, Oracle's **CLOB** type allows strings of up to 128TB in some cases).

A summary of database support for character strings is shown below:

=======================  ===================== ========== ======== =============== ================
Type                     SQLite                PostgreSQL MySQL    Oracle          SQL Server
=======================  ===================== ========== ======== =============== ================
CHARACTER(n)             equivalent to TEXT    yes        yes      yes             yes
CHARACTER VARYING(n)     equivalent to TEXT    yes        yes      use VARCHAR2(n) yes
CHARACTER LARGE OBJECT   equivalent to TEXT    use TEXT   use TEXT use CLOB        use VARCHAR(MAX)
=======================  ===================== ========== ======== =============== ================


Date and time types
-------------------

Management of date and time data is a very complicated affair.  Calendars change and differ among cultures, time zones vary widely, and "leap" adjustments to the calendar and clock occur irregularly.  SQL provides very robust data and time types along with operations on these types that allow for very precise storage and management of these values.  However, here again, implementations vary, and you should read your database system's documentation to understand the fine points.

The SQL standard defines three or five principal types, depending on how you count.  The types are **DATE**, **TIME** (with or without time zone), and **TIMESTAMP** (with or without time zone).  If you specify simply **TIME** or **TIMESTAMP**, you get the version without time zones; append **WITH TIME ZONE** to additionally store time zone information.

- **DATE** values store dates in such a way that any particular day in history can be accurately recorded.  Typically the Gregorian calendar is supported, but some implementations will convert to and from Julian dates.
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

In addition to the date and time types, SQL defines a useful set of types known a *interval* types, where an interval represents a span of days or time between two date or time values.  These are not covered in this book.

Additional data types
---------------------

Below is a list of some other data types you might encounter or wish to use in a SQL setting.  These are not supported by all RDBMSes.

- SQL defines a Boolean data type (**BOOLEAN**) which can store the literal values **True** and **False**.
- SQL also defines types designed to hold binary data.  This can sometimes be useful, although binary data such as images or music files take up a great deal of space; it is often preferable to store them externally, and only store in the database information about how to retrieve the files (e.g., a file path or URL).  The SQL standard includes the types **BINARY**, **BINARY VARYING**, and **BINARY LARGE OBJECT**; most implementations provide something analogous to **BINARY LARGE OBJECT**, usually under a different name.
- SQL provides for user-defined types; that is, custom data types created by the database user for specific applications.
- Many RBDMSes support types not defined in the SQL standard, or defined as optional extensions, such as types for storing and working with JSON and XML documents, geometric objects, geographical or spatial coordinates, arrays, and more.

