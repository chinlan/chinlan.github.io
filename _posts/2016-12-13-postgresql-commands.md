---
layout: post
title: "Postgresql commands"
description: ""
categories: [command]
tags: [postgresql]
redirect_from:
  - /2016/12/13/
---
# Initial setting
~~~
CREATE USER postgres;

ALTER USER postgres PASSWORD 'postgres';

ALTER USER postgres WITH SUPERUSER;
~~~
To have launchd start postgresql now and restart at login:
~~~
brew services start postgresql
~~~
Or, if you don't want/need a background service you can just run:
~~~
pg_ctl -D /usr/local/var/postgres start
~~~

Change the port when starting up:
~~~
pg_ctl -o "-F -p 5433" start
~~~
or:
~~~
postgres -p 5433
~~~

Delete specific migration file from db directly:
~~~
\d schema_migrations;

select * from schema_migrations;

delete from schema_migrations where filename = '20170508122200_create_users.rb';
~~~

Switch pager function:
~~~
db=# \pset pager

Pager is used for long output.

db=# \pset pager

Pager usage is off.
~~~

Stop manually:
~~~
pg_ctl -D /usr/local/var/postgres stop -s -m fast
~~~

Start manually:
~~~
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
~~~

To import the backup data into recent db, it is needed to empty the db at first:
~~~
pg_restore -vcaOxd databasename restorefilepath

psql
~~~

list all the psql database:
~~~
\l   
~~~
~~~
drop database databasename
~~~

If exited psql shell:
~~~
dropdb databasename
~~~

# About index:
If there is already an index on (a,b), then don't need to create another index on just (a). But (b,a) does not work like (a).

# Function & Trigger
- show all the functions stored in db:
~~~
\df <schema>.* 
~~~
- Get a list of all triggers in the system :
~~~
select * from pg_trigger;
~~~

# Using '-' in column name:

Avoid using '-' in column name if you can. If you can't, using double quotes.
~~~
ALTER DATABASE one RENAME TO "foo-bar";
~~~
ref: https://stackoverflow.com/questions/3942759/whats-the-escape-sequence-for-hyphen-in-postgresql

# Enum
- Select a specific enum (in the example it is event_type_enum), list its all values:
~~~
SELECT enum_range(NULL::event_type_enum);
~~~

- List all enums and their values:
~~~
\dT+
~~~

- List all the reserved words:
~~~
select * from pg_get_keywords()
~~~


