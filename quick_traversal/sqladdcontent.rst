===================================
8: SQL Traversal and Adding Content
===================================

Traverse through a resource tree of data stored in an RDBMS,
adding folders and documents at any point.

Background
==========

We now have SQLAlchemy providing us a persistent root. How do we
arrange an infinitely-nested URL space where URL segments point to
instances of our classes, nested inside of other instances?

SQLAlchemy, as mentioned previously, uses the adjacency list
relationship to allow self-joining in a table. This allows a resource
to store the identifier of its parent. With this we can make a generic
"Node" model in SQLAlchemy which holds the parts needed by Pyramid's
traversal.

In a nutshell, we are giving RDBMS data Python dictionary behavior,
using built-in SQLAlchemy relationships. This lets us define our own
kinds of containers and own kinds of types, nested in any way we like.

Goals
=====

- Recreate the :doc:`addcontent` and :doc:`zodb` steps, where you can
  add folders inside folders

- Extend traversal/dictionary behavior to SQLAlchemy models


Steps
=====

#. We are going to use the previous step as our starting point:

   .. code-block:: bash

    $ cd ..; cp -r sqlroot sqladdcontent; cd sqladdcontent
    $ $VENV/bin/python setup.py develop


#. Make a Python module for a generic ``Node`` base class that gives us
   traversal-y behavior in ``sqladdcontent/tutorial/sqltraveral.py``:

   .. literalinclude:: sqladdcontent/tutorial/sqltraversal.py
      :linenos:

#. ``sqladdcontent/tutorial/models.py`` is very simple,
   with the heavy lifting moved to the common module:

   .. literalinclude:: sqladdcontent/tutorial/models.py
      :linenos:

#. ``sqladdcontent/tutorial/__init__.py`` needs to import classes from
   ``sqladdcontent/tutorial/sqltraversal.py``:

   .. literalinclude:: sqladdcontent/tutorial/__init__.py
      :linenos:

#. Initialization script ``sqladdcontent/tutorial/initialize_db.py``
   is also changed :

   .. literalinclude:: sqladdcontent/tutorial/initialize_db.py
      :linenos:

#. Delete existing database and produce a new database.

   .. code-block:: bash
   rm sqltutorial.sqlite
   $ $VENV/bin/initialize_tutorial_db development.ini

   2015-01-23 19:29:36,398 INFO  [sqlalchemy.engine.base.Engine][MainThread] SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
   2015-01-23 19:29:36,398 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,399 INFO  [sqlalchemy.engine.base.Engine][MainThread] SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
   2015-01-23 19:29:36,399 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,400 INFO  [sqlalchemy.engine.base.Engine][MainThread] PRAGMA table_info("node")
   2015-01-23 19:29:36,400 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,400 INFO  [sqlalchemy.engine.base.Engine][MainThread] PRAGMA table_info("document")
   2015-01-23 19:29:36,400 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,401 INFO  [sqlalchemy.engine.base.Engine][MainThread] PRAGMA table_info("folder")
   2015-01-23 19:29:36,401 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,402 INFO  [sqlalchemy.engine.base.Engine][MainThread] 
   CREATE TABLE node (
         id INTEGER NOT NULL, 
         name VARCHAR(50) NOT NULL, 
         parent_id INTEGER, 
         type VARCHAR(50), 
         PRIMARY KEY (id), 
         FOREIGN KEY(parent_id) REFERENCES node (id)
   )


   2015-01-23 19:29:36,402 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,406 INFO  [sqlalchemy.engine.base.Engine][MainThread] COMMIT
   2015-01-23 19:29:36,407 INFO  [sqlalchemy.engine.base.Engine][MainThread] 
   CREATE TABLE document (
         id INTEGER NOT NULL, 
         title TEXT, 
         PRIMARY KEY (id), 
         FOREIGN KEY(id) REFERENCES node (id)
   )


   2015-01-23 19:29:36,407 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,411 INFO  [sqlalchemy.engine.base.Engine][MainThread] COMMIT
   2015-01-23 19:29:36,412 INFO  [sqlalchemy.engine.base.Engine][MainThread] 
   CREATE TABLE folder (
         id INTEGER NOT NULL, 
         title TEXT, 
         PRIMARY KEY (id), 
         FOREIGN KEY(id) REFERENCES node (id)
   )


   2015-01-23 19:29:36,412 INFO  [sqlalchemy.engine.base.Engine][MainThread] ()
   2015-01-23 19:29:36,416 INFO  [sqlalchemy.engine.base.Engine][MainThread] COMMIT
   2015-01-23 19:29:36,427 INFO  [sqlalchemy.engine.base.Engine][MainThread] BEGIN (implicit)
   2015-01-23 19:29:36,428 INFO  [sqlalchemy.engine.base.Engine][MainThread] INSERT INTO node (name, parent_id, type) VALUES (?, ?, ?)
   2015-01-23 19:29:36,428 INFO  [sqlalchemy.engine.base.Engine][MainThread] ('', None, 'folder')
   2015-01-23 19:29:36,429 INFO  [sqlalchemy.engine.base.Engine][MainThread] INSERT INTO folder (id, title) VALUES (?, ?)
   2015-01-23 19:29:36,429 INFO  [sqlalchemy.engine.base.Engine][MainThread] (1, 'My SQLTraversal Root')
   2015-01-23 19:29:36,430 INFO  [sqlalchemy.engine.base.Engine][MainThread] INSERT INTO node (name, parent_id, type) VALUES (?, ?, ?)
   2015-01-23 19:29:36,430 INFO  [sqlalchemy.engine.base.Engine][MainThread] ('f1', 1, 'folder')
   2015-01-23 19:29:36,430 INFO  [sqlalchemy.engine.base.Engine][MainThread] INSERT INTO folder (id, title) VALUES (?, ?)
   2015-01-23 19:29:36,431 INFO  [sqlalchemy.engine.base.Engine][MainThread] (2, 'Folder 1')
   2015-01-23 19:29:36,431 INFO  [sqlalchemy.engine.base.Engine][MainThread] INSERT INTO node (name, parent_id, type) VALUES (?, ?, ?)
   2015-01-23 19:29:36,432 INFO  [sqlalchemy.engine.base.Engine][MainThread] ('da', 2, 'document')
   2015-01-23 19:29:36,432 INFO  [sqlalchemy.engine.base.Engine][MainThread] INSERT INTO document (id, title) VALUES (?, ?)
   2015-01-23 19:29:36,433 INFO  [sqlalchemy.engine.base.Engine][MainThread] (3, 'Document A')
   2015-01-23 19:29:36,433 INFO  [sqlalchemy.engine.base.Engine][MainThread] COMMIT

#. Our ``sqladdcontent/tutorial/views.py`` is almost unchanged from the
   version in the ``addcontent`` step:

   .. literalinclude:: sqladdcontent/tutorial/views.py
      :linenos:

#. Our templates are all unchanged from addcontent. Let's bring them
   back. Make a re-usable snippet in
   ``sqladdcontent/tutorial/templates/addform.jinja2`` for adding content:

   .. literalinclude:: sqladdcontent/tutorial/templates/addform.jinja2
      :language: html
      :linenos:

#. Need this snippet added to
   ``sqladdcontent/tutorial/templates/root.jinja2``:

   .. literalinclude:: sqladdcontent/tutorial/templates/root.jinja2
      :language: html
      :linenos:

#. Need a view template for ``folder`` at
   ``sqladdcontent/tutorial/templates/folder.jinja2``:

   .. literalinclude:: sqladdcontent/tutorial/templates/folder.jinja2
      :language: html
      :linenos:

#. Also need a view template for ``document`` at
   ``sqladdcontent/tutorial/templates/document.jinja2``:

   .. literalinclude:: sqladdcontent/tutorial/templates/document.jinja2
      :language: html
      :linenos:


#. Run your Pyramid application with:

   .. code-block:: bash

    $ $VENV/bin/pserve development.ini --reload

#. Open ``http://localhost:6543/`` in your browser.

Analysis
========

If we consider our views and templates as the bulk of our business
logic when handling web interactions, then this was an intriguing step.
We had no changes to our templates from the ``addcontent`` and
``zodb`` steps, and almost no change to the views. We made a one-line
change when creating a new object. We also had to "stack" an extra
``@view_config`` (although that can be solved in other ways.)

We gained a resource tree that gave us hierarchies. And for the most
part, these are already full-fledged "resources" in Pyramid:

- Traverse through a tree and match a view on a content type

- Know how to get to the parents of any resource (even if outside the
  current URL)

- All the traversal-oriented view predicates apply

- Ability to generate full URLs for any resource in the system

Even better, the data for the resource tree is stored in a table
separate from the core business data. Equally, the ORM code for moving
through the tree is in a separate module. You can stare at the data and
the code for your business objects and ignore the the Pyramid part.

This is most useful for projects starting with a blank slate,
with no existing data or schemas they have to adhere to. Retrofitting a
tree on non-tree data is possible, but harder.

