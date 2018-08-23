# Guide to deal with sqitch development and production version control management



#### Steps to follow when creating a new project
---
* Create a new separate repository which will be used to version control database changes.
* Initialize empty sqitch project in that repository by using following command
```bash
# here we have assumed the db to be used is postgres. Please change engine parameter according to your requirement
# change project name to your required string
sqitch init PROJECTNAME --uri https://github.com/team-avesta/sqitch-startup/ --engine pg**
```
* What above commands does is that it clones a existing git repo created by us which contains basic folder structure that is required for sqitch projects and creates a `sqitch.plan` & `sqitch.conf` file which are required.
* After init sqitch project set username and email which will be used by sqitch to show who changed what in database
```bash
sqitch config --user user.name 'Marge N. O’Vera'
sqitch config --user user.email 'marge@example.com'
```

## Example use (Create new table)

* create users table query through pgadmin.
* After created table, run below command
```bash
sqitch add users -n 'Creates table to track our users'
```
* Execution of above command will create three files in deploy, revert and verify folder with users.sql name
* Copy create table query copy from pgadmin and paste into deploy/users.sql file.
* Copy drop table query and paste into revert/user.sql file. This file is used by sqitch when you perform rollbacks
* This query is for confirming if the required change was executed or not hence query in this file should be such that if it executes successfully then you can be sure that your change has been persisted to db. In this case we use a select query to check if users table is created. Put this query in verify/users.sql file.
* To run verify query, run below command 
```bash
sqitch verify
```
* Once changes are verified we need to commit the changes in our repo using below commands
```bash
git add .
git commit -am 'Add users table.'
```

## Updating production database
---

* When Database changes are to upload into production database, add tag into sqitch and git. For this use following command:
```bash   
#add new sqitch tag
sqitch tag v1.0.0-dev1 -n 'Tag v1.0.0-dev1.'**
#commit changes made by sqitch tag
git commit -am 'Tag the database with v1.0.0-dev1'
#add git tag (must when releasing )
git tag v1.0.0-dev1 -am 'Tag v1.0.0-dev1
```  
* Before release changes into production, tag is required. Otherwise we can not modify files.

## Changing existing function (which is already in production)
---

* After upload changes into production database, if change is required in any table/function then we need to run below command :
```bash
sqitch rework insert_user -n 'Change insert_user function'
```
* Now make the required changes in deploy/insert_user.sql, revert/insert_user.sql and verify/insert_user.sql

## How to download/use sqitch
- Sqitch is a command line utility which needs to be be present in your system's `PATH` to be used directly from command line.
- Instead of downloading the binary we will be using a docker container which provides us with sqitch binary. Run the command below to download the relevant docker image
```bash
docker run -it --network="host"  -v PATH_TO_YOUR_PROJECT_DATABASE_REPO  docteurklein/sqitch:pgsql
```
- In above command it is important to pass in the correct `volume (-v)` flag to docker run command as that is where all the sqitch actions will be executed.
- We can alias the above command as shown below so that it is easy to use during development
```bash
alias sqitchRnd='sudo docker run -it --network="host"  -v /home/upc1/MyProj/RND/sqitchRnd:/src  docteurklein/sqitch:pgsql'
# Now you can simply type sqitchRnd to access sqitch from terminal !
```
---

## Important 
* **NEVER** run `sqitch deploy` locally. It is meant to be used only when deploying production database
* You should not use **sqitch rework** for making changes to existing tables in postgres. It should be used only for altering functions