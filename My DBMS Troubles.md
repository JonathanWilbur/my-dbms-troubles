# My DBMS Troubles

By Jonathan M. Wilbur \<[jonathan@wilbur.space](mailto:jonathan@wilbur.space)\>.
Copyright (C) 2019 by Jonathan M. Wilbur.

## Preface

Please forgive the awkward transitions in tense, if there are any: I wrote this over the course of several months.

## The Problem

For a personal project, it seems as though I have tried every DBMS known to man to find a suitable one. This personal project involves probably around 300 million records initially and about 15 tables if the data is stored in relational tables, and it has potential for several hundred million more records. Needless to say, 300 million records of nearly any size is going to be so big that:

1. I will probably need to buy a separate hard drive for it alone, if I store it locally.
2. Loading over the Internet is going to mean that my computer will have to be uploading at full steam for hours, if not days or weeks.
3. Any of the following mistakes will be a catastrophe:
   1. A flaw in a table's schema.
   2. A flaw in the overall architecture.
   3. Incorrect data is loaded.
   4. Data is only partially loaded.
   5. I realize later on down the road that the capabilities of the DBMS do not suit my needs.

Further, I have these constraints:

1. I am financially crunched for a few reasons, so spending even just a few hundred dollars on this project is daunting. I believe this will let up in the next few months.
2. I am unsure of the benefits of this project. It may prove to be more expensive than its worth, although, because this data is pretty general purpose, I have a lot of ways I can use it; since there are so many things I can do with it, I am not too worried about this, but it is on my mind.
3. I have a lot of hobbies, a full-time job, a lot of obligations, a lot of friends and social activity, and a lot of goals. I am a busy man. Time is scarce for me. I spent over 500 hours in 2018 trying to get this project up and running, and I cannot afford to spend another 500 hours with no payoff in 2019.
4. I would like to wrap this up in the next few months. I do not want to wait for a new release of a DBMS that has the features I need to come out in an indefinite amount of time from now. 

Just thinking about this issue has had me pacing around my apartment for hours. I feel like I need psychotherapeutic help at times from the anxiety this induces in me. Let's review my story of this project.

## MongoDB

I like MongoDB. It really feels like it should have been invented first. Relational databases are so much more complicated by comparison. It is easy to see why so many companies are transitioning over to some kind of NoSQL data store.

I always try to be ahead of the curve technologically, so MongoDB was my first choice of DBMS for this project. MongoDB was really nice and easy to use, but I had (and still have) my doubts about its value for my particular use case, and so, nearly as quickly as I began, I made the switch to a relational DBMS.

## Why Relational Databases

Here are the reasons why I chose relational databases for this project:

1. My data comes in tabular form. It is not that hard to translate it into MongoDB documents, but the many ETL tools available for RDBMSs makes relational databases preferred in this regard.
2. My data is innately relational. MongoDB is good for storing mostly independent documents that do not require many other documents for meaningful analysis. One use case that comes to mind is X.509 certificates: an X.509 certificate makes sense being a single document, and X.509 version 3 extensions would be subdocuments in an array beneath them. There is little reason to normalize X.509 extensions into a separate table or collection, because an X.509 extension is only relevant to its certificate. On the other hand, a table of cars and their owners should be relational, because cars could change owners over time or have multiple owners or have no owners.
3. My interest in the data is innately relational. Document-oriented data stores are great for simple searches. But my use case is analyzing large amounts of data, not querying individual documents.
4. I believe it is probably easier to go from the more ordered state of relational data to the sloppier anarchic state of document-oriented data than the other way around, but that's just a hunch. That appeals to me, because it means that making a mistake in my choice of DBMS for this project could potentially be less catastrophic if I start off with a relational database.

5. Referential integrity constraints found in relational databases work like a dual-linked list.  Continuing on an earlier example, if you wanted to find the extensions that belong to a certificate, that would be easy in MongoDB: either query the subdocuments in the extensions member, or dereference them, if they are Object IDs. But would you be able to easily find which certificate owns a given extension? That is much harder unless you deliberately add the owning certificate's Object ID to each extension. The relational system of referential integrity not only automatically adds, but enforces this feature.

6. The data normalization that relational databases incentivizes also tends to innately limit the amount of data returned from a query to only what is needed. It "sandboxes" the data returned, in the sense that deliberate action has to be taken to obtain more data than what is already present in a table. For instance, in MongoDB, the default behavior is to query the entire document, regardless of which fields you are actually interested in. You deliberately have to add to your query to limit the fields returned by a document. If you have lazy developers, this means that your application can suffer in performance as their queries pull in largely unnecessary data over the wire. On the other hand, a `SELECT *` statement in a relational database query will only return all fields from a given relational table. You have to explicitly join that table to another table to obtain more information.

   To give an example, let's say you have a lazy developer creating a search results page. Using MongoDB, he queries the entirety of all matching documents, just so he can display the  `name` and `date` fields in the search results grid. On the other hand, if he is querying a relational database, and said relational database contained a table that contained only `name`, `date`, `country` and `id` fields, his lazy `SELECT *` statement would only return all data in that table, and in this case, only the `country` and `id` fields would be wasted. Anything one-to-many, which would become subdocuments in the document-oriented case, would be a separate table in a relational database, and therefore be automatically excluded from his query. I think it wise to count on the laziness of others.

7. Often, when you want to view data, document-oriented databases put out unsightly output. I like to see tables, and I suspect others do, too. 

8. There are hundreds of good general-purpose graphical RDBMS clients. There are only few--if not only one--graphical client per document-oriented database, simply because they are newer. I can even view relational data in Excel.

9. Just like with graphical RDBMS clients, there are a lot more tools available for RDBMS-based BI and analysis, which is precisely my use case.

## Postgres

As with anything I do, I did an immense amount of comparative research in RDBMSs. What initially strongly appealed to me about Postgres is the support for `CHECK` constraints, which meant that I could enforce data quality at the database, rather than trusting my ETL process to do that. Further, data validation at the database level just makes sense, because *data is the database's concern* and because having validation right there in your schema is a more canonical way of performing validation, rather than trusting that your validation is being performed application-side. And in my case, since I *don't have an application-side*, this is especially appealing.

I started creating schema for Postgres and found it a reliable and capable RDBMS. But what killed Postgres for me was not the DBMS itself, but the tools around it. Starting from the [Postgres clients wiki](https://wiki.postgresql.org/wiki/PostgreSQL_Clients), and only accepting cross-platform, free, and open-source clients, I started with pgAdmin (version 3.0 or so). It was terribly buggy to the point of being nearly unusable, so I went to SQLectron. SQLectron was reasonably free of bugs, from what I recall, but the performance was awful on my computer, to the point of almost being unusable. It would take nearly a full second after pressing a keystroke for the next character to appear in the query editor. Scrolling through data was equally painful. As I find out now, SQLectron is now abandoned, but some people have expressed some half-hearted volunteer desires to maintain it.

As fate would have it, my Red Hat Certified System Administrator (RHCSA) certification would bring MariaDB to my attention, which would become my next stab at a DBMS.

## MariaDB

For the next few months to come, I set up a MariaDB server on my local device. This is where I would initially collect and process my data, as well as develop the ETL process. This took months and went very smoothly, at least in relation to how innately difficult the task at hand was.

Finally, showtime was upon me. I exported my schema and data to perform a test load on my local machine. I was devastated to learn that MariaDB has [an outstanding bug](https://jira.mariadb.org/browse/MDEV-17654) where it exports schema with improper syntax, so I had to manually edit my schema across 15 or more tables to make the data load possible. Of course, that is not a big deal one time, but given the experience I already had up until this point, I expected to have to do this quite a few times in the future. There was an outstanding pull request that has already been approved, but it was just hanging there, with nobody actually merging it. God only knew how long it will take for it to be fixed. And if I wanted to put this data up on a cloud service like Amazon Web Services (AWS) Relational Database Services (RDS), I would also have to wait for the fixed version to be adopted and supported.

## MySQL

At this point, I decided to sacrifice my precious `CHECK` constraints and opt for MySQL, figuring that, since it was older and less feature rich, it was probably more free of bugs. MySQL became increasingly attractive as AWS Aurora can behave like a MySQL DBMS, and even more attractive because AWS Serverless RDS only supported MySQL mode.

Setting up an RDS instance on AWS is not easy. In fact, it was so horribly frustrating that I began to question my choice of AWS and signed up for Azure to check out their hosted options. Setting up RDS on AWS required the setup of two different subnets, and applying a Network Address Translation (NAT) gateway on one, and an Internet Gateway on the other, then linking them to create a "subnet group" in which AWS could deploy the storage and DBMS separately. To multiply my suffering, configuring the security permissions for this setup was not straight-forward either.

After finally getting it working, testing connectivity, I began importing some of the smaller tables via CSV dumps, only to find, to my dismay, that the entire process hit a screeching halt once I reached a particular table with a lot more columns and with free-form text fields.

In the process of trying to upload these data, I discovered another bug in MySQL workbench. It was trivial, but foreboding. Upon uploading this particular troublesome CSV dump, I encountered unusual errors that prevented me from proceeding. Figuring that this was an issue with MySQL Workbench, I tried the same thing with HeidiSQL. HeidiSQL did not stop me from importing the CSV, but all but three rows errored out. The three that made it in the table were corrupted.

It would sound crazy to think that there would be a similar bug in two separate graphical clients instead of in the DBMS itself, but I know from experience that HeidiSQL does not dump SQL schema correctly--namely, it does not escape quotes in comments, so single quotes in any `COMMENT` field will destroy your schema. I am normally beneficent enough to write up and report bugs like this, but for a piece of shit program like HeidiSQL, which is written in Pascal and has fewer than 1000 stars on GitHub, I'll pass.

Having said that, I was willing to try a third client: the command-line client. I tried to upload the CSV via the MySQL shell only to get yet another strange error.

Ruling out CSV now, I attempted to upload JSON dumps instead, which are only possible in MySQL workbench, and which also failed with a cryptic error.

Reaching my boiling point, I looked on the MySQL bugs site to find that there was indeed [a bug](https://bugs.mysql.com/bug.php?id=71091) for MySQL's inability to digest certain CSV dumps going all the way back to 2013. It is still open, and probably will be forever and ever, amen.

## Back to the Drawing Board

I went back to the drawing board to consider these options:

| DBMS                 | Pros                                                         | Cons                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| MongoDB              | It's new, popular, and it would be easy to insert the data.  | Document-oriented                                            |
| MariaDB              | Highly compatible, feature rich, and generally well-functioning | Dysfunctional DDL exports                                    |
| ElasticSearch        | It's meant to be a search engine; though not the point, that is handy. The ELK stack is beautiful for viewing data, and obtaining streams of new data. | Document-oriented                                            |
| Neo4j                | Graph searches could be useful; it seems like graph databases be a good middle-ground between RDBMSs and document-oriented databases. | I don't know it, and I suspect the tooling is very limited.  |
| Microsoft SQL Server | The tooling is great--especially SSMS, Azure Data Studio, and SSIS. SSAS and SSRS are crap, but there are plenty of other tools for BI, analysis, and reporting. It is also probably far less buggy, since it has such strong enterprise support behind it, and because it is the top-of-the-line database. | It is barely affordable to buy a few client access licenses; it would destroy me financially to have it hosted. It would also require considerable re-writing of my schema, since T-SQL and MySQL do not match up too well. |
| Postgres             | Highly compatible, feature rich, and well-supported.         | I had a bad experience with clients for Postgres. It also looks like Postgres' bug reporting page is extremely antiquated and even unmanageable, which is a bad sign. |

## Back to Postgres

I went back to Postgres. My rationale for going back was the opposite of my rationale for leaving it. pgAdmin 4.0 had been released, and by my cursory glimpse at the changelog, it sounded like pgAdmin 4.0 should have been quite a revolutionary improvement, but even if pgAdmin proved unsatisfactory, I could still defer to the other DBMS clients I learned about since trying Postgres the last time.

After getting it set up, I had trouble loading hierarchical data that uses an adjacency list model, because the inserts were not in an order in which the foreign key would always correspond to an already inserted primary key. I looked around for a way to temporarily disable foreign key constraints, which you can do with MariaDB and MySQL. It is silly on my part, but eventually, I realized that I could just remove the constraint altogether and re-add it after insertion.

I really struggled inserting the data, to no small degree because of a peculiarity in how MySQL and MariaDB exports CSV files with `NULL` integral types: apparently, MySQL and MariaDB quietly convert all of your `NULL` integers to zero when dumping to CSV. Thanks a lot for corrupting my data, guys. It only took me a year to make that. The only thing I could even find that mentions this behavior is [this known issue](https://bugs.mysql.com/bug.php?id=20501). Because almost no database supports embedded newlines in CSV files, I had to remove those manually as well.

To fix a few issues with foreign keys that were once `NULL` being converted to zeroes, I had to open up my exported CSV in Excel and convert said zeroes to `NULL`. Opening the CSV in Excel inadvertently changed my strings that contained only numbers to numbers themselves, which truncated the leading zeroes. Since said strings were foreign keys, this corrupted my database as well, and the fastest way to do it was simply manually fixing the cells after telling Excel to treat them as strings instead of numbers.

A few days later, I had a network-layer connectivity issue with the database. I was stumped for about an hour, then discovered that my public IP address changed, so the default AWS RDS role, which is configured to permit only traffic from the public IP from which the instance was created, was now blocking my access. This obviously does not implicate any particular DBMS, but it was a silly problem worthy of note for posterity's sake.

Finally, as fate would have it, I searched for "worst bugs with postgres" on the Internet, just to see if I could unearth any unwelcome "surprises" before I get balls-deep in Postgres, and I found [this article](https://eng.uber.com/mysql-migration/) from Uber Engineering that pretty much convinced me once and for all not to use Postgres for anything. I was going to quote the most damning parts from the article, but the entire thing was jaw-dropping, so I'll spare you the block quotes and invite you to read the article yourself.

## Back to MariaDB

I realized that the issue I had with MariaDB is more of an issue with MySQL Workbench, whose export wizard does not allow you to specify an escape character. When I exported the data with DataGrip (after changing CSV exports from the default configuration), all `NULL`s were properly exported as `\N` instead of a zero. (In DataGrip, the default would have been to output them as an empty field.)

This assuaged my major concern with MariaDB, and so I resumed work with MariaDB. However, I decided to start with a test on MySQL.

### Practice MySQL

I set up a test MySQL 8.0 server on AWS RDS to validate the process for inserting data, and to test if my new correct CSV exports would work as expected. Once again, I had immense difficulty inserting hierarchical data that uses an adjacency list model.

At first, the inserts would hang until a timeout message appeared, which read `SQLSTATE[HY000]: General error: 1205 Lock wait timeout exceeded; try restarting transaction`. I toggled every setting I could in DBeaver with no success. I could not find any process with any open locks on that table, so I had to believe that something about the import itself was causing the issue. Retrying the same import on an identical table, except with no foreign key constraint, worked. This seemed to be the case whether or not `FOREIGN_KEY_CHECKS` was set to `FALSE` or not. So I determined that I would perform the inserts without the foreign key constraint first, then add it afterwards.

Then I encountered a separate issue: when I went to apply the foreign key constraint at the end, MySQL complained that a foreign key constraint was not satisfied by one of my rows! I was bamboozled, as my CSV export of this particular table came straight from a MariaDB table that had the foreign key constraint applied! I stressfully stroked my face and hair for an hour, sighing repeatedly as I leaned back in my office chair. Staring at the data, the elusive issue finally caught my attention: for some rows, the foreign keys that were supposed to be `NULL` were, in fact, empty strings! I had a hard time explaining this, because I had DataGrip's `Null value text` option set to `Empty string`, so it would seem that this should not have been an issue. I could not find an outstanding bug report for this, but I wonder if DataGrip meant two consecutive commas with no quotes to mean an "empty string." After replacing the empty strings in my CSV data dump with `\N`, and changing the setting in DataGrip, applying the foreign key constraint worked perfectly.

Then I went to insert the final table in my test, which I had to do in DBeaver to avoid [a known bug](https://youtrack.jetbrains.com/issue/DBE-7199) in DataGrip that causes imports to fail when a line exceeds 5000 characters. I had problems yet again. At first, the issue was that it was not respecting my configured `NULL` string for parsing the CSV. As I later discovered, DBeaver's CSV parser simply escapes the next character after a backslash in a CSV, regardless of whether or not that character is actually a special character in a CSV, such as a quote or comma, so all of my cells with `\N` in the CSV were being seen as `N`. Since I was pretty certain that there were no columns that would simply contain `N` by itself, I changed the configured `NULL` string in DBeaver and got past that issue.

Then, the terrible `SQL Error [1205] [40001]: Lock wait timeout exceeded; try restarting transaction` appeared yet again. I don't even know how I fixed this, but eventually, it figured itself out.

After that, I was getting an error saying that a value was too large to be inserted into its corresponding `VARCHAR(64)` column. As it turns out, my original CSV export, which did not quote all values, had a value that began with a quote, which seems to have confused the CSV parser (and quite understandably so, I will admit). I re-exported the CSV from DataGrip, mandating quotation for all values this time, and this problem was fixed.

Like a Kafkaesque masterpiece, I immediately ran into yet another problem: MySQL was complaining that an integral value was out of range for the column's data type. I used Visual Studio Code's Rainbow CSV RBQL to query just the offending column, and I could see no entries that exceeded the range of the column's data type. Nevertheless, I decided to change the data type of the column to a wider integer; to be fair, I was cutting it pretty close to the maximum with the values I had anyway.

Rather than dropping the whole table, I tried to alter the column. Theory tells you that, since I had no data in the table, this should be trivial, but not for me. Trying to alter a column would hang indefinitely in both DataGrip and DBeaver. I would have to restart either DataGrip or DBeaver. When I logged back in, I would be unable to do anything because I would get a deadlock error. Fixing this was a matter of running `SHOW FULL PROCESSLIST` and deleting the process that was trying to alter the table, or simply rebooting the server.

After giving up on trying to alter the table, I simply dropped the table, and re-inserted it with the wider integral data type for the offending field. After doing so, the insert worked. In DBeaver, I did not receive any pop up or visual indicator of success; I just heard the familiar Microsoft Windows "ding" sound. I then inspected the table, and found, with great elation, that data was inserted.

However, I had yet another problem: not all rows were inserted. In the end, this turned out to have happened simply because of the modifications I made to the CSV to remove the embedded newlines. In the process, I accidentally stripped some of the newlines separating rows, which put two rows on one line. It is a miracle that the inserts worked.

## Hiatus

Finally settling on MariaDB, I went on hiatus from this project, strategically waiting for the aforementioned [DDL exports bug](https://jira.mariadb.org/browse/MDEV-17654) to be fixed. My nightmare of an experience that was trying to use a goddamn database had proven to me that I need to be more prepared for change. Thought I expected MariaDB to be perfectly fine after the DDL exports bug was fixed, I had also to believe that my constant vacillations in choice of DBMS was a trend that was likely to continue.

### PreQL

To be prepared for the next show-stopping catastrophe that would make me change DBMSs entirely, I had to, at the very least, convert my schema into some common format that could be transpiled into targeted SQL dialects on demand. 

Before having a clear idea of how this pre-SQL language would look, I gave it a name, SQLx, whose trailing "x" was inspired by "JSX," a language often used with React-based web applications that transpiles to JavaScript. Later on, I would find out that SQLx was already taken by multiple different applications and even an organization, so, with little effort, I articulated a new name for it: PreQL, which is an obvious pun. The name "PreQL" was taken, but by two libraries on GitHub, both of which had fallen into abandonment. The most popular among the two had only 76 stars and 20 watchers, so I felt quite licensed to commandeer "PreQL" from the namespace. It still astounds me that "PreQL" was otherwise unused.

In the next few months, I ruminated over how such portable schema would look. My original idea was that it would follow MySQL syntax, such that all valid MySQL would also be valid PreQL, though not the other way around.

Originally, the point of this pre-SQL language was only to port table schema from one language to another easily, but as I mulled over the potential features, PreQL became more and more impressive. I thought to add interfaces and possibly classes, so that you could ensure that all implementing tables implemented a modicum of common columns. I added a dearth of new data types that would transpile to some combination of native data types, check constraints, and triggers. I added import statements, so that the schema could be split among different files, but transpiled to a single one.

I also decided to make the syntax declarative, rather than the usual imperative syntax of most--if not all--SQL dialects. This declarative syntax would describe the desired state of your database, rather than what steps to take, and in what order, to obtain that desired state. Behind the scenes, PreQL would resolve dependency relationships between desired objects and transpile to the corresponding imperative SQL in the correct order to create those objects. Rather than the usual pattern of aborting the creation of a table if one already exists, PreQL would separate the table creation from the column addition, so that columns would be added to meet the desired state of a table if it changed in PreQL.

By this time, PreQL had expanded from a glorified DDL to a really advanced and capable language. I got the wise idea to create an `entries` object, that you could use to pre-populate your tables with data. I later got the even wiser idea to allow you to directly import CSV, TSV, JSON, and XML files, whose entries would be converted to `INSERT` statements.

This central language was also not limited to relational databases. I could export to document-oriented databases or LDAP directories easily, though some functionality would be lost. I count at least ten relational database languages I planned to support, which sounds immense, but there is a lot of overlapping code.

Needless to say, all of these features added up to a monumental project. I was fortunate enough to realize that I could just write everything in YAML files rather than create my own new language lexer and parser. This cut down what I thought would be a year-long project to one that could be done in a few weeks.

I started to write [the PreQL transpiler](https://github.com/JonathanWilbur/preql) when I was in Tallahassee, visiting a friend. Only a few days later, after being ignored for months, escalated to critical severity, stalled twice, and reviewed, the fix was merged and released for the dreaded DDL export bug. This escalated my interest in completing the PreQL transpiler, so that I could be done in time for the fixed version to be deployed to AWS and / or Azure.

### The AWS Bill

It had evaded my notice for the previous months, but I had been receiving larger-than-usual AWS bills. Don't panic: it was just around $76 per month, but it still cost me quite a few gumballs. Upon investigating, I discovered that it was because I was being charged for the NAT gateway that I had to have to make AWS RDS work! As it turns out, NAT gateways cost you about $33.00 per month! I had entered into this expecting my RDS instance to cost some $30 a month or less in total after purchasing a reserved instance, but this unexpected charge, coupled with the immense difficulty of setting up AWS RDS, made me reconsider Azure.  What follows is experimentation with MariaDB on Azure, to test both the ease and costs of use, and to determine the _precise_ current version, since Azure only says "10.2," upfront, and omits the third version digit.

### MariaDB on Azure

I spun up a MariaDB instance on Azure. The user interface on Azure was _far better_ than that of AWS. The process of spinning up a new database was flawlessly simple. 

Configuring the firewall, on the other hand, _was_ obnoxious to do, only because accepting traffic from anywhere was not documented. I had assumed that setting a start range and end range to `0.0.0.0` would mean "accept traffic from anywhere," as it usually does in other services, but, instead, I had to use a range of `1.1.1.1` to `255.255.255.255`, which I discovered from a comment on a StackOverflow page. (Now that I am thinking about it, I also wonder if this would fail to support IP addresses with zeroes in them...)

 (I know accepting traffic from anywhere is bad practice, but this was just a test server that I was going to delete anyway.)

Once I got past that, I got a new error, though I do not remember what it was precisely, nor how I figured out how to fix it. Whatever it was, I realized that I needed to enable TLS for the connection using DBeaver.  To do so, I needed the certificate of the certificate authority (the "trust anchor") that signs the certificates (or the intermediary CA certificates) so that DBeaver could verify the server's identity. DBeaver could not just use the trusted store that comes with my operating system; I had to supply it with a specific root certificate.

Finding this certificate was a nuisance. After digging around the documentation for it, and finding nothing, I wrote up [an issue](https://github.com/MicrosoftDocs/azure-docs/issues/30536) in the GitHub comments on the Azure documentation page. Now on my own to figure out what the root certificate was, I thought to use OpenSSL's `s_client` to nab the root certificate from the chain that the server presents. Unfortunately, this too was hampered. Upon connecting `s_client` would err, with a message saying "wrong version number." Eventually, I found the root certificate, not from the documentation pages directly, but instead, by clicking a link in the security settings for my database in the Azure console. Configuring DBeaver to use this certificate finally allowed me to move on to the next error: `Access Denied`.

In the Azure console, I reset the password, thinking I mistyped it, and I later reset the database. I reset DBeaver on my workstation as well. None of these resolved the `Access Denied` error.

Starting to believe that this was an issue with the client, I re-downloaded MySQL Workbench, and found that *it worked perfectly*. That indicated to me that the issue was with DBeaver. As I had become quite accustomed, I checked the bugs listings for DBeaver ultimately to discover that this was [a known bug](https://jira.mariadb.org/browse/CONJ-480) with the MariaDB Connector for Java, which DBeaver was using. No matter how correct your credentials were, the connector would get `Access Denied`. I was using version 1.5.5, but upgraded to version 2.4.1, which fixed the error. In an attempt to fix this the right way, I searched the source code to find a "default version" so I could submit a pull request that would make a newer version the default, but I found no references to "1.5.5," which made me believe that I must have accepted it, somehow. If memory serves me correctly, I only skimmed the DBeaver page asking me to install the driver for MariaDB.

I know you would never believe this, but then I ran into another error: the names on the certificate did not match the endpoint. This re-kindled my need to probe the TLS layer using `s_client`, just to make sure that TLS was actually working. Figuring that LibreSSL (which I was using as a result of using a MacBook Air) was just a crappy second-rate spin-off of OpenSSL, I decided to probe the TLS socket using the bona fide OpenSSL library by running it in a Docker container. As it turns out, Docker Hub was down that day for maintenance; they had a security breach days before. Luckily, I had a Ubuntu image cached on my machine, and I was able to spin up a container and install OpenSSL. It had the same unfortunate result.

I used the `-debug` and `-msg` options in `s_client` to inspect the troublesome packets myself. The first byte of the offending packet did not correspond to any valid content type in the TLS protocol. I submitted [a question on StackOverflow](https://security.stackexchange.com/questions/209512/what-is-this-tls-record/209513), and a responding genius informed me that the MySQL protocol (which MariaDB uses) does not actually use TLS as a wrapper to the whole connection from start to finish; instead, it uses a STARTTLS-like handshake. My attempts to use `s_client` were in vain, because the remote endpoint was still responding with the MySQL protocol!

Unable to probe the endpoint further, I returned to the error message. It was clear, from the (dangerously generous) listing of wildcard Subject Alternative Names, that `*.mariadb.database.azure.com` was missing.  I only discovered this issue via the GitHub comments on Microsoft's documentation. In case you were wondering, here is the list of all of the subject alternative names listed on this certificate:

```
cr1.eastus1-a.control.database.windows.net
*.cr1.eastus1-a.control.database.windows.net
eastus1-a.control.database.windows.net
*.eastus1-a.control.database.windows.net
*.database.windows.net
*.secondary.database.windows.net
*.mysql.database.azure.com
*.postgres.database.azure.com
```

This means that, if the private key is exfiltrated for a single endpoint, all endpoints for MySQL, Postgres, eventually MariaDB, and probably SQL Server on Azure will be susceptible to man-in-the-middle attacks. I [commented on the GitHub issue](https://github.com/MicrosoftDocs/azure-docs/issues/25361) with my concerns, and I emailed [Microsoft's cybersecurity department](mailto:secure@microsoft.com) with my concerns as well.

To conclude a day's work, I discovered that, for some reason, DBeaver was ignoring the setting to skip server certificate verification. Instead, I had to add `disableSslHostnameVerification` with a value of `true` to the driver settings. As I would later discover, the reason MySQL Workbench did not have this issue was that I did not select "Verify Server Identity" in the connection settings; using that setting gave me a verification-related error, as expected.

In the end, it turns out that Azure uses MariaDB version `10.2.22-MariaDB-log (MariaDB Server)`, which is just shy of version 10.2.24, which fixes the DDL export bug. It may sound like all of this was for naught, but my point of setting up this server was simply to test Azure. Considering that half of the issues I experienced above were not related to Azure, I would say I have a slightly favorable opinion of MariaDB on Azure, with my main complaints being the sloppy TLS configuration and lack of important details in documentation. I have a *far greater* opinion of MariaDB on Azure than I do of AWS RDS in general. In fact, my cursory experience with Azure makes me think it is probably superior on many other fronts as well.

In case you're curious, this experiment to simply connect my DBeaver client to a MariaDB database on Azure cost me about 12 hours of my life.

## My Experience with General-Purpose Editors

Below are the details of my experience with various RDBMS GUI clients. My current recommendation for graphical clients is this:

* If you are using Microsoft SQL Server, use SSMS or Azure Data Studio.
* If you are using anything else, use the following:
  * DataGrip on a large workstation (if the $90 price tag is worth it)
  * DBeaver on resource-constrained workstations, such as laptops.

### MySQL Workbench

MySQL Workbench gets the job done, but it is very buggy. There are problems with slowness when viewing even a modest amount of data. There is no _good_ support for theming, and even when you change themes, it leaves certain graphical features unchanged, so it looks hideous. But it can do just about whatever you want it to do.

### SQL Server Management Studio

Microsoft's SQL Server Management Studio is slick. It is unambiguously a top-of-the-line RDBMS GUI client. It does take up a lot of st0rage and a system reboot to install it, but it is lovely once you have it up and running.

### HeidiSQL

HeidiSQL is a joke of a high-school project. It is written in Pascal, which nobody uses any more. The user interface is hideous, the non-data output from queries is hardly better than simply viewing from the command line. I found out the hard way that exporting a table DDL with single quotes in the `COMMENT` fields produces an invalid DDL, because it does not escape the inner quotes. Further, it has almost no recognition on GitHub and no documentation. It is a miracle it was installed by default alongside MariaDB.

### pgAdmin 4.0

It's hideous, it's slow, and it feels like trudging through mud using it. Every click is exhausting. It implements its own in house file browser for when you want to upload files instead of using the sensible built-in one. It is a web application, and I already hate the idea of using web applications for viewing database data.

The only nice thing I have to say about it is that, while CSV imports are tedious as stated already, a lot of it comes from how customizable it is. There were also a few things that I think were buggy with the upload, but I chose not to explore them further, because I already do not care for pgAdmin.

### Valentina

Valentina is pretty nice. Everything loads blazing fast. No dark theme, but still looks good. Pretty authoritarian about requiring registration. Uninstalling because I don't want to have to register it or buy it.

### DBeaver

Don't let DBeaver's website fool you: it is actually a very well-functioning database studio, and it is free and open source. It has a dark theme, but looks fine without it. It is freemium, so there is some money coming in to maintain it. I am very pleased with it so far. The best benefit of it is that its memory usage is very low in comparison to other graphical clients, so it is the best choice for lightweight laptops.

### DataGrip

DataGrip is complex and kind of annoying to set up, but it is loaded with features. It is almost overwhelming. It truly feels like a "studio" rather than just a viewer / editor. Once you get used to it, you can feel the power. It is also unambiguously the most beautiful studio I have seen so far, and it includes a dark theme. It is reasonably cheap, but I don't know that it would be worth it to buy it, since DBeaver is already perfectly fine for my purposes.

CSV imports are exceptionally capable, with the exception of this one issue: [lines over 5000 characters cause the import to fail](https://youtrack.jetbrains.com/issue/DBE-7199). I don't know for sure, but if they are using fixed-length buffers to store variable text, it's a problem, and indicative of further problems I may experience down the line. This single issue converted by confident purchase into hesitation.

## Appendix A: Root Certificates

The root certificate used for MariaDB on Azure is found [here](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem), but for posterity's sake, here is the PEM encoding of the certificate:

```
-----BEGIN CERTIFICATE-----
MIIDdzCCAl+gAwIBAgIEAgAAuTANBgkqhkiG9w0BAQUFADBaMQswCQYDVQQGEwJJ
RTESMBAGA1UEChMJQmFsdGltb3JlMRMwEQYDVQQLEwpDeWJlclRydXN0MSIwIAYD
VQQDExlCYWx0aW1vcmUgQ3liZXJUcnVzdCBSb290MB4XDTAwMDUxMjE4NDYwMFoX
DTI1MDUxMjIzNTkwMFowWjELMAkGA1UEBhMCSUUxEjAQBgNVBAoTCUJhbHRpbW9y
ZTETMBEGA1UECxMKQ3liZXJUcnVzdDEiMCAGA1UEAxMZQmFsdGltb3JlIEN5YmVy
VHJ1c3QgUm9vdDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKMEuyKr
mD1X6CZymrV51Cni4eiVgLGw41uOKymaZN+hXe2wCQVt2yguzmKiYv60iNoS6zjr
IZ3AQSsBUnuId9Mcj8e6uYi1agnnc+gRQKfRzMpijS3ljwumUNKoUMMo6vWrJYeK
mpYcqWe4PwzV9/lSEy/CG9VwcPCPwBLKBsua4dnKM3p31vjsufFoREJIE9LAwqSu
XmD+tqYF/LTdB1kC1FkYmGP1pWPgkAx9XbIGevOF6uvUA65ehD5f/xXtabz5OTZy
dc93Uk3zyZAsuT3lySNTPx8kmCFcB5kpvcY67Oduhjprl3RjM71oGDHweI12v/ye
jl0qhqdNkNwnGjkCAwEAAaNFMEMwHQYDVR0OBBYEFOWdWTCCR1jMrPoIVDaGezq1
BE3wMBIGA1UdEwEB/wQIMAYBAf8CAQMwDgYDVR0PAQH/BAQDAgEGMA0GCSqGSIb3
DQEBBQUAA4IBAQCFDF2O5G9RaEIFoN27TyclhAO992T9Ldcw46QQF+vaKSm2eT92
9hkTI7gQCvlYpNRhcL0EYWoSihfVCr3FvDB81ukMJY2GQE/szKN+OMY3EU/t3Wgx
jkzSswF07r51XgdIGn9w/xZchMB5hbgF/X++ZRGjD8ACtPhSNzkE1akxehi/oCr0
Epn3o0WC4zxe9Z2etciefC7IpJ5OCBRLbf1wbWsaY71k5h+3zvDyny67G7fyUIhz
ksLi4xaNmjICq44Y3ekQEe5+NauQrz4wlHrQMz2nZQ/1/I6eYs9HRCwBXbsdtTLS
R9I4LtD+gdwyah617jzV/OeBHRnDJELqYzmp
-----END CERTIFICATE-----
```

The root certificate used for AWS RDS is found [here](), but for posterity's sake, here is the PEM encoding of the certificate:

```
-----BEGIN CERTIFICATE-----
MIID9DCCAtygAwIBAgIBQjANBgkqhkiG9w0BAQUFADCBijELMAkGA1UEBhMCVVMx
EzARBgNVBAgMCldhc2hpbmd0b24xEDAOBgNVBAcMB1NlYXR0bGUxIjAgBgNVBAoM
GUFtYXpvbiBXZWIgU2VydmljZXMsIEluYy4xEzARBgNVBAsMCkFtYXpvbiBSRFMx
GzAZBgNVBAMMEkFtYXpvbiBSRFMgUm9vdCBDQTAeFw0xNTAyMDUwOTExMzFaFw0y
MDAzMDUwOTExMzFaMIGKMQswCQYDVQQGEwJVUzETMBEGA1UECAwKV2FzaGluZ3Rv
bjEQMA4GA1UEBwwHU2VhdHRsZTEiMCAGA1UECgwZQW1hem9uIFdlYiBTZXJ2aWNl
cywgSW5jLjETMBEGA1UECwwKQW1hem9uIFJEUzEbMBkGA1UEAwwSQW1hem9uIFJE
UyBSb290IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuD8nrZ8V
u+VA8yVlUipCZIKPTDcOILYpUe8Tct0YeQQr0uyl018StdBsa3CjBgvwpDRq1HgF
Ji2N3+39+shCNspQeE6aYU+BHXhKhIIStt3r7gl/4NqYiDDMWKHxHq0nsGDFfArf
AOcjZdJagOMqb3fF46flc8k2E7THTm9Sz4L7RY1WdABMuurpICLFE3oHcGdapOb9
T53pQR+xpHW9atkcf3pf7gbO0rlKVSIoUenBlZipUlp1VZl/OD/E+TtRhDDNdI2J
P/DSMM3aEsq6ZQkfbz/Ilml+Lx3tJYXUDmp+ZjzMPLk/+3beT8EhrwtcG3VPpvwp
BIOqsqVVTvw/CwIDAQABo2MwYTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUTgLurD72FchM7Sz1BcGPnIQISYMwHwYDVR0jBBgwFoAU
TgLurD72FchM7Sz1BcGPnIQISYMwDQYJKoZIhvcNAQEFBQADggEBAHZcgIio8pAm
MjHD5cl6wKjXxScXKtXygWH2BoDMYBJF9yfyKO2jEFxYKbHePpnXB1R04zJSWAw5
2EUuDI1pSBh9BA82/5PkuNlNeSTB3dXDD2PEPdzVWbSKvUB8ZdooV+2vngL0Zm4r
47QPyd18yPHrRIbtBtHR/6CwKevLZ394zgExqhnekYKIqqEX41xsUV0Gm6x4vpjf
2u6O/+YE2U+qyyxHE5Wd5oqde0oo9UUpFETJPVb6Q2cEeQib8PBAyi0i6KnF+kIV
A9dY7IHSubtCK/i8wxMVqfd5GtbA8mmpeJFwnDvm9rBEsHybl08qlax9syEwsUYr
/40NawZfTUU=
-----END CERTIFICATE-----
```





[^1]: https://blogs.msdn.microsoft.com/sqlreleaseservices/announcing-the-modern-servicing-model-for-sql-server/