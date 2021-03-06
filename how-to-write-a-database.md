---
title: How to write [A] Database
date: 2017-07-14 03:47:10
description:
featured_image:
issue: https://github.com/RyougiNevermore/ryouginevermore.github.io/issues/6
tags:
 - 搬运
 - 数据库
 - 程序设计
---

Its a great question, and deserves a long answer.
Most database servers are built in C, and store data using B-tree type constructs.
In the old days there was a product called C-Isam (c library for an indexed sequential access method) which is a low level library to help C programmers write data in B-tree format.
So you need to know about btrees and understand what these are.

Most databases store data separate to indexes. Lets assume a record (or row) is 800 bytes long and you write 5 rows of data to a file.
If the row contains columns such as first name, last name, address etc. and you want to search for a specific record by last name, you can open the file and sequentially search through each record but this is very slow.

Instead you open an index file which just contains the lastname and the position of the record in the data file.
Then when you have the position you open the data file,
lseek to that position and read the data.
Because index data is very small it is much quicker to search through index files.

Also as the index files are stored in btrees in it very quick to effectively do a quicksearch (divide and conquer) to find the record you are looking for.

So you understand for one "table" you will have a data file with the data and one (or many) index files.

The first index file could be for lastname, the next could be to search by SS number etc.

When the user defines their query to get some data, they decide which index file to search through.

If you can find any info on C-ISAM (there used to be an open source version (or cheap commercial) called D-ISAM) you will understand this concept quite well.

Once you have stored data and have index files, using an ISAM type approach allows you to GET a record based on a value, or PUT a new record.

However modern database servers all support SQL, so you need an SQL parser that translates the SQL statement into a sequence of related GETs.

SQL may join 2 tables so an optimizer is also needed to decide which table to read first (normally based on number of rows in each table and indexes available) and how to relate it to the next table.

SQL can INSERT data so you need to parse that into PUT statements but it can also combine multiple INSERTS into transactions so you need a transaction manager to control this, and you will need transaction logs to store wip/completed transactions.
It is possible you will need some backup/restore commands to backup your data files and index files and maybe also your transaction log files, and if you really want to go for it you could write some replication tools to read your transaction log and replicate the transactions to a backup database on a different server. Note if you want your client programs (for example an SQL UI like phpmyadmin) to reside on separate machine than your database server you will need to write a connection manager that sends the SQL requests over TCP/IP to your server, then authenticate it using some credentials, parse the request, run your GETS and send back the data to the client.
So these database servers can be a lot of work, especially for one person. But you can create simple versions of these tools one at a time. Start with how to store data and indexes, and how to retrieve data using an ISAM type interface.
There are books out there - look for older books on mysql and msql, look for anything on google re btrees and isam, look for open source C libraries that already do isam. Get a good understanding on file IO on a linux machine using C. Many commercial databases now dont even use the filesystem for their data files because of cacheing issues - they write directly to raw disk. You want to just write to files initially.
I hope this helps a little bit.