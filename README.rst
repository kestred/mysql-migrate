*************
mysql-migrate
*************

MySQL Migrate is a set of shell scripts to manage changes to a database schema.
The scripts are supported on Linux and macOS.

To the extent possible under law, I waive all copyright and related or neighboring rights to git-consistent.

.. image:: http://i.creativecommons.org/p/zero/1.0/88x31.png


Usage
=====
The "setup.sql" file sources all of the .sql files (eg. `source version/migrate.sql`)
necessary to prepare the database for the current version of your software.

The "vX.Y.Z" directories should each contain a "migrate.sql" script which
updates the schema to that version from the previous version.


Tools
=====

Tools for developing the database live in the "tools" directory:

tools/init DBNAME
 This tool prepares a new directory to keep your schema SQL files in.
 This command should be run to setup a new project for MySQL Migrate.

tools/add DBNAME VERSION
 This tool creates a new empty migration file for lazy typists.

 Here is an example use:

     tools/add example v1.3.0

tools/create DBNAME
 This tool creates a new database with the given name, dropping it if already existed.

tools/apply DBNAME SQL_FILE
 This tool allows you to run a SQL file and update the "latest.sql" file with
 a compact form of the database schema.

 This allows a local database to be prepared by running::

     tools/create example && tools/apply example example/setup.sql

 Then as migrations are written, they can be applied with::

     tools/apply example vX.Y.Z/migrate.sql

tools/dump DBNAME
 This tool dumps the current schema of the database for the purposes of local
 development, or to export it from another migration system.


Use Cases
=========
SQL Migrate was built for a system that works with traditional vertically-scaled
databases and not "Google" or "Facebook" scale database requirements. Luckily,
despite the aspirations of most application authors, the vast majority of web
applications in 2016 satisfy that requirement.

In particular, SQL Migrate was designed to replace migration systems like
Python's SQLAlchemy or Ruby's ActiveRecord with a pure SQL solution.

If you have very strict requirements for how long the database can be locked
during an upgrade, look at an online schema change tool like GitHub's gh-ost_.
Typically, I avoid those tools unless I really need them because every online
schema migration tool I've seen doesn't work on tables with foreign key
constraints -- and I'm quite fond of those!


Best Practices
==============
In general, migrations should be completely backwards compatible with any code
that may currently be running in your production environment.

This can usually be accomplished by adhering to a few rules:

1) Do not DROP tables or columns until the release after the final release
   in which they are used by code.
2) When adding a new column, whose value can be derived from an old column,
   create a TRIGGER to keep the new column up to date with the old column,
   then UPDATE the new column in the same way.
3) To rename a column, instead add a new column in one release and drop the old
   column in the next release (see #1 and #2).
