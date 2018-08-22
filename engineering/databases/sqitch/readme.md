# Guide to deal with sqith development and production version control management


#### General
---
* Usually the first thing to do when starting a new project is to create a source code git repository.
* Every Sqitch project must have a name associated with it, and, optionally, a unique URI. We recommend including the URI, as it increases the uniqueness of object identifiers internally. 
  * **sqitch init flipr --uri https://github.com/theory/sqitch-intro/ --engine pg**
* After init sqitch project set username and email
  * **sqitch config --user user.name 'Marge N. O’Vera'**
  * **sqitch config --user user.email 'marge@example.com'**


#### Things to watch out
---
* Example **Add users table** Please follow steps:
     * create users table query through pgadmin.
     * After created table, run command **sqitch add users -n 'Creates table to track our users.'**
       - after execution of above command created three files in deploy, revert and verify folder with users.sql name.
       - Create table query copy from pgadmin and paste into deploy/users.sql file.
       - Drop table query into direct revert/user.sql file.
       - Table is created or not, Put select query in verify/users.sql file.
     * To run verify query, run command **sqitch verify**
     * git add .
     * git commit -am 'Add users table.'
* When Database changes upload into production database, add tag into sqitch and git. For this use following command:
  * **sqitch tag v1.0.0-dev1 -n 'Tag v1.0.0-dev1.'**
  * **git commit -am 'Tag the database with v1.0.0-dev1.'**
  * *g*it tag v1.0.0-dev1 -am 'Tag v1.0.0-dev1'**
* Before release changes into production, tag is required. Otherwise we can not modify files.
* After upload changes into production database, if change in any file run this command :
  * **sqitch rework insert_user -n 'Change insert_user function.'**
  * Now you have to change in deploy/insert_user.sql, revert/insert_user.sql and verify/insert_user.sql.

```
