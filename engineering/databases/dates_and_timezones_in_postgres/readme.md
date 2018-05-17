# Guide to deal with dates and timezones in Postgres and Node.js

Dates and timezones are a tricky thing to deal with when working with postgres which we discovered after going through a lot of pain. Hence based on our learnings we are documenting below points which should be always followed when working with postgres for making everyone's life a lot easy.

#### General
---
* Always use **timestamp with timezone data type column** for storing timestamp information.
* **Postgres db timezone should always be set to 'UTC'**. Timezone information can be set using postgres configuration parameters. If you are using AWS RDS then too it provides such configuration options to set pg timezone
* Unless absolutely required **DO NOT ACCEPT timestamps from client**. Timestamps should always be on the backend side most preferably in postgres SP's (i.e functions)
#### Things to watch out for on Client side
---
* In case of UI which have grids with date filters such as By Date, By month, Date Range etc. the dates that should be passed to server should always contain timezone information and **MUST ALWAYS BE IN UTC** timezone format. Date string in UTC can be obtained using `new Date().toISOString()`Date string examples:
     * 2018-02-02T14:07:40.340Z (valid)
     * Fri Feb 02 2018 19:39:18 GMT+0530 (IST) (invalid)
     * 02/02/2018 (invalid)
     * 02-02-2018 (invalid)
     * Any other date string which does not have time and timezone information (invalid)
* When working with just date filter where we are dealing explicitly with dates and not time ensure that the hours, minutes and seconds value in the date that is to sent to server are set to 0 (assuming client is in IST). So if you are filtering for records on say 2nd Feb 2018 the following values are valid
  * 2018-02-01T18:30:00.000Z (12 AM on 2nd Feb is 18:30 on 1st Feb in UTC as it runs -5.30 from IST)
  * 2018-02-02T18:30:00.000Z (above date + 24 hours )
* If using Angular material the datepicker component that is inbuilt gives date string with timezone information in UTC and that too with hours, minutes and seconds set to 0. If you select 15th Jan 2018 in picker the date value you get in your model will be `2018-01-14T18:30:00.000Z`. Hence in this case it requires no more effort on the frontend while sending params in API
* If you need to filter data on date and time both then the format of date will be the same as mentioned below (i.e UTC) but as expected do not set hh/mm/yy bits to zero.

#### Things to watch out for on Node
---
* At times there would be requirement to generate monthly or weekly reports automatically on scheduled intervals via cron jobs on the server or any other mechanism. This means that client would not be involved here in supplying the date information and the date that is required to be sent for querying would be generated from node itself.
* In such cases the knowlegde of timezone of client to whom these reports are going to be delivered matters becuase the UTC date string will have to be created based upon the client's time zone. For example if we need to query records for say 2nd Feb 2018 then UTC string that should be sent to pg for two clients in IST and PDT at the time of this writing is as follows
``` sql
--IST (+5.30 UTC)
from : '2018-02-01T18:30:00.000Z' to : '2018-02-02T18:30:00.000Z'
--PDT (-7 UTC)
from : '2018-02-02T07:00:00.000Z' to : '2018-02-03T07:00:00.000Z'
```
* In order to generate such kind of dates on node it may be recommended to use moment.js

#### Things to watch out for in Postgres
---
* In postgres all date or time or date/time queries will be range queries as we no longer compare just the date from date string. We need the exact date string with timezone information for all date related queries. Examples are shown below
```sql
--query for a particular date for ex 2nd feb 2018
SELECT  * FROM  your_table WHERE your_column between '2018-02-01T18:30:00.000Z' and timestamp '2018-02-01T18:30:00.000Z'  + interval '1' day;
--query for all records of January 2018
SELECT  * FROM  your_table WHERE your_column between '2017-12-31T18:30:00.000Z' and timestamp '2017-12-31T18:30:00.000Z'  + interval '1' month;
```
