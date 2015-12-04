Does your team manually populate the database (e.g. test data, config values)?  Do they ever encounter problems where deployed code fails due to a missing column or table, or runs slowly due to a missing index? Do team members email DDL statements to each other in an attempt to setup their development environment? Is one person solely responsible for maintaining and applying DDL changes (even in the non production environments)? If any of these scenarios is familiar, then you probably aren't doing a very good job managing your database. Effective management of your database brings many benefits including reduced development time ($$$), reduced production support ($$$), higher quality, and happier team members and end users. 
 
Your codebase and your database should be in sync in every environment. Modern web development frameworks include built-in tools for this type of work; Ruby on Rails includes Active Migration, Pythons' Django framework includes Migrations , and Java... well, Java doesn't really have *a* web framework (hello Spring, Struts, Wicket, Tapestry, Server Faces....) but it does include some very good migration managers. We have been using Liquibase for several years now, and recently converted the Client Data Management project (aka PETR) to use Liquibase for the management of DDL and data population. 
 
Liquibase
 
Liquibase is an established and actively maintained open-source project. The liquibase code is written in Java (but can be used for any type of project) and is typically included via a Maven plugin or Ant task. You store your database migrations along with your project source code. Each migration is stored as a discrete changeset which includes metadata such as an identifier, the author(s), comments, etc. For supported databases, you can use the Liquibase DSL which allows you write migrations in an SQL agnostic way (one benefit being that you can port the changes to other databases). Here is a sample:
 
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.7"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.7
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.7.xsd">
    <changeSet id="1" author="bob">
        <comment>A sample change log</comment>
        <createTable .../>
    </changeSet>
    <changeSet id="2" author="bob" runAlways="true">
        <alterTable .../>
    </changeSet>
    <changeSet id="3" author="alice" failOnError="false" dbms="oracle">
        <alterTable .../>
    </changeSet>
</databaseChangeLog>
 
If you don't use a supported database or you prefer writing the SQL yourself, you can supply the raw SQL instead:
 
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.7"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.7
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.7.xsd">
    <changeSet author="bob" id="101">
        <sql>
            insert into person (name) values ('Bob')
            <comment>Adding Bob test value</comment>
        </sql>
    </changeSet>
    <changeSet author="bob" id="101">
        <sqlFile dbms="h2, oracle"
            encoding="utf8"
            endDelimiter="\nGO"
            path="my/path/file.sql"
            relativeToChangelogFile="true"
            splitStatements="true"
            stripComments="true"/>
    </changeSet>
</databaseChangeLog>
 
Liquibase makes it easy to automate the execution of changesets on your database. You can run liquibase whenever you deploy your code or startup your server (because your changesets are included as part of your code artifact). By default, Liquibase uses a special table in your database to store metadata for all of the changesets it has applied. It typically looks like this:
 
ID 	AUTHOR 	FILENAME 	DATEEXECUTED 	ORDER	MD5SUM	COMMENTS
1	Bob	changeSet1.sql	2015-11-23 09:16:47.280	1	7:4d156240f3f79936d14b20500468477b 	Adding Bob..
2	Alice	changeSet2.sql	2015-11-23 09:16:47.373	2	7:6012929267a8588da966859a0e537f00	Renamed Column from...
 
This changelog table allows Liquibase to determine which changeSets have already been run, and which remain outstanding. It also provides traceability; anyone can inspect what changes have been made to the database, who created them, and what their purpose was. All of the changesets are check summed, so if someone tries to retroactivley modify a changeSet, Liquibase will fail and notify you when attempting to update your environment. 
 
But wait you say! This isn't the wild west! Someone needs to vet the SQL before it can be applied on the production database. No problem. For those environments that you don't have control over, you can tell Liquibase to run in offline mode and it will output the necessary SQL to a file.  You can then hand this to your gatekeeper and ask them to apply it to the database. 
 
Finally, you can apply context tags to your changesets to ensure that some changes are only run in specific environments. For example, you may have a changeset that populates the EMPLOYEES table with fake data for the development and QA environments. You don't want this changeset run in the PAT or PROD environment. This can easily be achieved by assigning a context tag to the changeset.
 
These are just a few of the features available in Liquibase. There are many others (Rollbacks, Diffs, Preconditions, etc) so I encourage you to investigate the tool for yourself!
