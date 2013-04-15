---
layout: post
title: "Postgresql installation on debian Wheezy"
date: 2011-10-05 13:47
comments: true
categories: [PostgreSQL, Debian GNU/Linux, Old Enki Blog]
---

Packages installation
---------------------

    $ sudo apt-get update && sudo apt-get install postgresql postgresql-client postgresql-contrib postgresql-doc

The installation process takes care of the `postgres` user creation as of the starting of the server.

PostgreSQL configuration
------------------------

We now have to set up the administrative password:

    $ sudo passwd postgres


Then, we could create a new super-user:

    $ su - postgres
    $ create user

and define a password for that user using

    $ psql -d template1 -c "alter user MyNewUser with password 'TheNewPassword'"


Remote administration
---------------------

On the server side, you may configure PostgreSQL by editing the `/etc/postgresql/9.1/main/pg_hba.conf`. Check the [pgi_hba.conf File official documentation](http://developer.postgresql.org/pgdocs/postgres/auth-pg-hba-conf.html) for further information.

On the client-side, we could install pgadmin3

    $ sudo apt-get install pgadmin3

Et voilà !
