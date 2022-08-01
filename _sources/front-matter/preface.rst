=======
Preface
=======

    *In God we trust, all others bring data.*

    -- attributed to W. Edwards Deming

This book, *A Practical Introduction to Databases*, was created to fill the need for a low-cost, up-to-date textbook suitable for an introductory course on databases:

- The book is free to students and instructors (although we encourage you to support the Runestone Academy platform to help ensure its future sustainability and growth).
- The book is open. [#]_  Its contents are licensed under the `Creative Commons Attribution-ShareAlike 4.0 International License`_, meaning that anyone is free to share, adapt, and redistribute them for any purpose whatsoever, as long as appropriate credit is given and the license terms persist.  The  source for this book is available on github; you may fork, extend, and re-release this book in any form you wish (pull requests welcome!).  This book is intended as a living document, to be updated and expanded regularly as technology changes, by any interested author.
- The book is oriented to students encountering databases for the first time, with a focus on key skills needed by future software engineers.

.. _`Creative Commons Attribution-ShareAlike 4.0 International License`: https://creativecommons.org/licenses/by-sa/4.0/

In this first release, the book covers three core topics: SQL, data modeling, and relational database theory. [#]_  In determining which material to include or exclude, choices were made to prefer material with high impact and practical application over exhaustive coverage.  Instructors may therefore wish to use this book as a foundation, supplemented with additional material as required for their course.  Instructors of more advanced courses in databases may also find value in providing this book as a supplement for students encountering the subject for the first time.  To the extent possible, the content is presented in a way that allows different orderings of topics, although some dependencies are unavoidable.

In keeping with the applied focus of the book, SQL is offered as the starting point in :numref:`Part {number} <sql-part>` of the text.  At a time when academia and industry alike are developing new advances in database technology seemingly on a daily basis, the relational database model as mediated by SQL remains the gravitational center for practitioners.  SQL is the common point of reference used to evaluate job applicants on their database skills.  The online book uses an embedded SQLite database engine to provide interactive examples and facilitate exploration (scripts are provided in :ref:`Appendix A <appendix-a>` to create the textbook's database on several other major database systems, if preferred).

:numref:`Part {number} <data-modeling-part>` discusses data modeling.  Here, the emphasis is on database design using entity-relationship diagrams (ERDs) and related tools.  The coverage includes a detailed example demonstrating ERD construction along with step-by-step instructions for converting ERDs to a relational database schema.

Finally, :numref:`Part {number} <relational-theory-part>` of the text provides coverage of relational database theory, including relational algebra and normalization (through fourth normal form).


|chapter-end|

----

**Notes**

.. [#] This book is an Open Education Resource (OER), and is aligned with the goals laid out in the `UNESCO Recommendation on Open Educational Resources`_, among others.

.. [#] Future releases are planned, covering software programming with databases, NoSQL, and other topics of practical interest.

.. _`UNESCO Recommendation on Open Educational Resources`: https://www.unesco.org/en/legal-affairs/recommendation-open-educational-resources-oer

|license-notice|
