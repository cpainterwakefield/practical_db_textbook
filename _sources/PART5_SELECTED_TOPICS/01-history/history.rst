.. _history-chapter:

===================================
A brief history of database systems
===================================

Prior to the invention of the hard drive in the 1950's, collections of data intended for computer processing were stored on media such as paper tape, paper punch cards, and magnetic tape.  These media allowed *sequential access* to data; data input started from the beginning of the collection and proceeded linearly through.  In general, processing proceeded by first reading a large amount of data into a computer's main memory, where it could be accessed and manipulated efficiently.  Processed data could then be stored on similar media for future use.

In contrast, a modern storage device such as a hard drive or solid-state drive allows for *random access*; the computer can quickly move to an arbitrary location in a data store without having to read through all of the data in earlier locations.  If data is carefully organized on a random access medium, it becomes possible to efficiently access or modify individual pieces of data on demand.

Early software applications used this approach to create organized data stores that were application-specific; the data store could only be understood or managed by the specific software application.  If it became necessary to modify the structure of the data (for instance, by storing new attributes for every piece of data), the software would have to be rewritten, and the existing data store would have to be converted to the new structure.


A brief history
:::::::::::::::

The first commercial database systems appeared in the 1960's, not long after the commercial introduction of hard disk drive storage.  These early database systems allowed software developers to organize data in structures similar to trees or graphs.  Individual pieces of data were stored in fixed-size *records*, which were often direct mappings of a software application's data structures in memory.  Connections between records were also stored in the database, often using references which could be directly translated to addresses on the storage device, allowing for fast navigation between related pieces of information.  These *navigational* databases were very successful, and their successors are still sold today.

Navigational databases had certain drawbacks in interfacing with software application code.  In particular, navigational databases coupled program code to the structures stored in the database.  Changes to the database could require extensive changes to program code.  Changes in the needs of the software could require the addition of more structures to the database, leading to an accumulation of structures to be maintained over time.

Edgar F. Codd, a computer scientist working for computer company IBM, is credited [#]_ with proposing a new database model based on mathematical set theory, the relational model.  The new approach eliminates many (although not all) of the dependencies coupling application code to the structure of the database.  Some of the structures built into navigational databases (those storing connections between records) are no longer needed, while others (such as indexes) are effectively hidden from application code.  Data queries can be expressed in a simple and consistent fashion using mathematical operations.  The relational model thus raises the level of abstraction in interfacing application code with the database system.

Codd shared his relational model internally at IBM, and then publicly in :ref:`a paper <relational-theory-references>` in 1970.  Codd and other authors further developed the relational model in subsequent papers and books.  The first relational database systems based on Codd's work appeared in the mid-to-late 1970's.  Relational database systems came into heavy use in subsequent decades.  While it is difficult to measure the prevalence of relational databases relative to other types in use today, the relational model remains hugely influential and important.

Relational model beginnings
:::::::::::::::::::::::::::

The first commercial database systems appeared in the 1960's, not long after the commercial introduction of hard disk drive storage.  These early database systems allowed software developers to organize data in structures similar to trees or graphs.  Individual pieces of data were stored in fixed-size *records*, which were often direct mappings of a software application's data structures in memory.  Connections between records were also stored in the database, often using references which could be directly translated to addresses on the storage device, allowing for fast navigation between related pieces of information.  These *navigational* databases were very successful, and their successors are still sold today.

Edgar F. Codd, a computer scientist working for computer company IBM, is credited [#]_ with proposing a new database model based on mathematical set theory.  This new approach differs from navigational database systems in that links between records are not explicitly stored in the database.  Rather than following predetermined access paths provided by the database designer, relational database users and application code are free to write arbitrary queries that translate to simple mathematical operations on sets of records.  Responsibility for determining proper and efficient execution largely shifts from application code to the database system itself.  The result is a higher level of abstraction in the interface to the database compared to earlier database systems.

Codd shared his relational model internally at IBM, and then publicly in :ref:`a paper <relational-theory-references>` in 1970.  Codd and other authors further developed the relational model in subsequent papers and books.  The first database systems based on Codd's work appeared in the mid-to-late 1970's.  While it is difficult to measure the prevalence of relational databases relative to other types in use today, the relational model remains hugely influential and important.

For more on the history of database systems, see :numref:`Chapter {number} <history-chapter>`.

|chapter-end|

----

**Notes**

.. [#] :ref:`An earlier paper by David L. Childs <relational-theory-references>` in 1968 on the application of set theory to data systems is a precursor cited by Codd.  However, it was Codd's 1970 paper that became widely circulated and influential in later work on the relational model, and in the creation of early relational database systems such as System R from IBM and the INGRES database developed at the University of California, Berkeley.

.. [#] :ref:`An earlier paper by David L. Childs <relational-theory-references>` in 1968 on the application of set theory to data systems is a precursor cited by Codd.  However, it was Codd's 1970 paper that became widely circulated and influential in later work on the relational model, and in the creation of early relational database systems such as System R from IBM and the INGRES database developed at the University of California, Berkeley.

|license-notice|
