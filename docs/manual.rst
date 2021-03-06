===============================
NMAP2DB - NMAP scans management
===============================

|
| Version-1.0.0
|
| Author: Rafael Martinez Guerrero (University of Oslo)
| E-mail: rafael@postgresql.org.es
| Source: https://github.com/rafaelma/nmap2db
|

.. contents::


Introduction
============

NMAP2DB is a tool for managing nmap scans and save the result in a
PostgreSQL database.

It is designed to run thousands of scans a day , save the results in a
database and use the nmap2db shell to interact with the system.

The NMAP2DB code is distributed under the GNU General Public License 3
and it is written in Python and PL/PgSQL. 



Main features
=============

The main features of NMAP2DB are:

* Central database with metadata and raw information.
* NMAP2DB shell for interaction with the system.
* Management of multiple networks
* Management of multiple scan types
* Scans scheduling
* Host reports
* Scan reports
* OS reports
* Ports reports
* Host without DNS entry.
* Written in Python and PL/PgSQL 
* Distributed under the GNU General Public License 3


Architecture and components
===========================

The components forming part of NMAP2DB could be listed as follows:

* **Scan servers:** One or several servers running NMAP2DB. They will
  use nmap to execute the scans defined in the system and will access
  via ``libpq`` the nmap2db database to save and retrieve the data.

* **nmap2db DB**: Central postgreSQL database used by NMAP2DB. All
  scan servers need access to this database.

* **NMAP2DB shell:** This is a program that must be run in a text
  terminal. It can be run in any of the scan servers. It is the
  console used to manage NMAP2DB.


Installation
============

You will have to install the NMAP2DB software in all the servers
that are going to be used to run nmap scans.

System requirements
-------------------

* Linux/Unix
* Python 2.6 or 2.7
* Python modules:
  
  * psycopg2
  * argparse
    
* PostgreSQL >= 9.2 for the ``nmap2db`` database
* NMAP >= xxxx 

Before you install NMAP2DB you have to install the software needed by
this tool

In systems using ``yum``, e.g. Centos, RHEL, ...::

  yum install python-psycopg2 python-argparse nmap

In system using ``apt-get``, e.g. Debian, Ubuntu, ...::

  apt-get install python-psycopg2 python-argparse nmap

If you are going to install from source, you need to install also
these packages: ``python-dev(el), python-setuptools, git, make, rst2pdf``

In systems using ``yum``::

  yum install python-devel python-setuptools git make rst2pdf

In system using ``apt-get``::

  apt-get install python-dev python-setuptools git make rst2pdf


Installing from source
----------------------

The easiest way to install nmap2db from source is to get the last
version from the master branch at the GitHub repository.

::

 [root@server]# cd
 [root@server]# git clone https://github.com/rafaelma/nmap2db.git

 [root@server]# cd nmap2db
 [root@server]# ./setup2.py install
 .....

This will install all users, groups, programs, configuration files, logfiles and the
nmap2db module in your system.


Installing via RPM packages
---------------------------

RPM packages for CentOS 6 and RHEL6 are available at
https://github.com/rafaelma/nmap2db/releases

Install the RPM package with::

  [root@server]# rpm -Uvh nmap2db-<version>.rpm

Or::

  [root@server]# yum install nmap2db-<version>.rpm
  

Installing the nmap2db database
---------------------------------

After the requirements and the NMAP2DB software are installed, you
have to install the ``nmap2db`` database in a server running
PostgreSQL. This database is the core of the NMAP2DB tool and it is
used to save all the metadata needed to manage the system.

You can get this database from the directory ``sql/`` in the source
code or under the directory ``/usr/share/nmap2db`` if you have
installed NNAMP2DB via ``source``, ``rpm`` or ``deb`` packages.

::

   psql -h <dbhost.domain> -f /usr/share/nmap2db/nmap2db.sql

There is another file in this directory named
``nmap2pg_table_partition.sql``. This file can be used to install and
configure partitioning of the main tables used by NMAP2DB. We
recommend to use table partitioning when using NMAP2DB. The nmap2db
database can became very large if you have a large network and you
want to keep some historic data. Partitioning will help to have a good
performance when searching for data in the database.

Run this command to install partitioning support.

::

   psql -h <dbhost.domain> -f /usr/share/nmap2db/nmap2db_table_partition.sql


Configuration
=============

Scan servers
------------

A scan server needs to have access to the ``nmap2db`` database. This
can be done like this:

#. Update ``/etc/nmap2db/nmap2db.conf`` with the database parameters
   needed by NMAP2DB to access the central database. You need to
   define ``host`` or ``hostaddr``, ``port``, ``dbname``, ``database``
   under the section ``[nmap2db_database]``.

   You can also define a ``password`` in this section but we discourage
   to do this and recommend to define a ``.pgpass`` file in the home
   directory of the users ``root`` and ``nmap2db`` with this
   information, e.g.::

     <dbhost.domain>:5432:nmap2db:nmap2db_role_rw:PASSWORD

   and set the privileges of this file with ``chmod 400 ~/.pgpass``.

   An even better solution will be to use ``cert`` autentication for
   the nmap2db database user, so we do not need to save passwords
   values.

#. Update and reload the ``pg_hba.conf`` file in the postgreSQL server
   running the ``nmap2db`` database, with a line that gives access to
   the nmap2db database from the new backup server. We recommend to
   use a SSL connection to encrypt all the traffic between the database
   server and the backup server, e.g.::

     hostssl   nmap2db   nmap2db_role_rw    <scan_server_IP>/32     md5 



System administration and maintenance
=====================================

If NMAP2DB is using table partitioning we have to run a job every
month to maintain all the tables, triggers and indexes we use for
this.

This job can be executed via cron everty month. Create this file
``/etc/crond.d/nmap2db`` with this content.

::

   SHELL=/bin/bash
   PATH=/sbin:/bin:/usr/sbin:/usr/bin
   MAILTO=your@email_address

   01 00 01 * * root /usr/bin/psql -h <your.dbhost> -U nmap2db_role_rw nmap2db -c "SELECT create_nmap2db_partitions_tables()"

The script ``/etc/init.d/nmap2db_ctrl.sh`` can be used to start or
stop new nmap2db scan processes. This is a simple bash script that
does not follow or implement any System V requirements and can not be
used to start/stop nmap2db automatically when the server running
NMAP2DB boots or shutdowns.

To start e.g. 40 nmap2db scan processes:

::

   /etc/init.d/nmap2db_ctrl.sh -n 40 -c start

To stop all nmap2db scan processed::

::

   /etc/init.d/nmap2db_ctrl.sh -c stop


NMAP2DB shell
===============

The NMAP2DB interactive shell can be started by running the program
``/usr/bin/nmap2db``

::

   [nmap2db@scan_server]# nmap2db
   ########################################################
   Welcome to the Nmap2DB shell (v.1.0.0)
   ########################################################
   Type help or \? to list commands.
   
   [nmap2db]$ help
   
   Documented commands (type help <topic>):
   ========================================
   EOF                shell                       show_os              
   clear              show_history                show_port            
   quit               show_host_reports           show_report_details  
   register_network   show_host_without_hostname  show_scan_definitions
   register_scan_job  show_network_definitions    show_scan_jobs       
   
   Miscellaneous help topics:
   ==========================
   shortcuts
   
   Undocumented commands:
   ======================
   help
   

**NOTE:** It is possible to use the NMAP2DB shell in a non-interactive
modus by running ``/usr/bin/nmap2db`` with a command as a parameter in
the OS shell. This can be used to run NMAP2DB commands from shell
scripts.e.g.::

  -bash-4.1$ nmap2db show_network_definitions
  +------------------+---------+
  | Network          | Remarks |
  +------------------+---------+
  | 46.105.18.177/32 |         |
  +------------------+---------+

Use help <command_name> to get some information about a command.


Submitting a bug
================

NMAP2DB has been extensively tested, and is currently being used in
production. However, as any software, NMAP2DB is not bug free.

If you discover a bug, please file a bug through the GitHub Issue page
for the project at: https://github.com/rafaelma/nmap2db/issues


Authors
=======

In alphabetical order:

|
| Rafael Martinez Guerrero
| E-mail: rafael@postgresql.org.es / rafael@usit.uio.no
| PostgreSQL-es / University Center for Information Technology (USIT), University of Oslo, Norway
|

License and Contributions
=========================

NMAP2DB is the property of Rafael Martinez Guerrero / PostgreSQL-es
and USIT-University of Oslo, and its code is distributed under GNU
General Public License 3.

| Copyright © 2012-2014 Rafael Martinez Guerrero / PostgreSQL-es
| Copyright © 2014 USIT-University of Oslo.
