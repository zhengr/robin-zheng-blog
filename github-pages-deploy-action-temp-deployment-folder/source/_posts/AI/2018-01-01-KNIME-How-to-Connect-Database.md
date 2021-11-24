---
title: KNIME 连接主流数据库方法
date: 2017-10-29 00:00:00
author: robin zheng
img: https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/db_prefs.png
categories: AI
tags:
  - Database
  - Knime
---
## KNIME 连接主流数据库方法


### Database Documentation
#### Installation and Setup
KNIME allows connecting to almost all JDBC-compliant databases. Within the Database category in the Node Repository view, KNIME offers a number of database access, manipulation and writing nodes.

In the KNIME preference page that can be accessed via File > Preferences > KNIME > Database Driver, the vendor-specific database driver need to be registered before using the database nodes. The driver file can be either a Jar or Zip file:

![Homepage_Illustration_2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/db_prefs.png)

After the driver is registered, all database nodes can be used to connect to your favorit database. The Database Reader and Database Connection Reader, or Database Writer, can be configured in the following way:

![Homepage_Illustration_2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/db_reader_with_credentials.png)


Database Driver: the combo box shows all available database driver, select the one that is approviated for your database.
Database URL: the database-specific protocol used to connect to the database.
Workflow Credentials: as specified on the workflow using the item 'Workflow Credentials', the user can simply select from on of the credentials holding user name and password.

![Homepage_Illustration_2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/db_credentials_dialog.png)

User Name / Password: the other option is to use user name / password directly to connect to the database.
Database Browser: enter all settings before connecting to the database, the browser view shows all tables and columns contained in the database. Click on either option to insert it into the SQL statement.
SQL Statement: enter the SELECT statement to fetch the data from the spcified table.
Flow Variable List: list of flow variables at this particular node; those can be used within the SQL statement to parametrize the Reader node and/or to use it within a loop.


#### Vendor-specific Database Connectors
All vendors bring along their specific database driver which are not part of the KNIME release. Those need to be downloaded from the vendor's website or can be retrieved from your database installation/adminstrator:

- SQLite: http://bitbucket.org/xerial/sqlite-jdbc/downloads
- Postgres: http://jdbc.postgresql.org/download.html
- Microsoft SQL Server: http://msdn.microsoft.com/en-us/library/ms378526.aspx
- Oracle: http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html
- MySQL: http://www.mysql.com/downloads/connector/j
- IBM DB2 / Informix: http://www14.software.ibm.com/webapp/download/search.jsp?go=y&rs=ifxjdbc
- Apache Derby: http://db.apache.org/derby/derby_downloads.html
- SAP HANA: https://support.sap.com/en/my-support/software-downloads.html 