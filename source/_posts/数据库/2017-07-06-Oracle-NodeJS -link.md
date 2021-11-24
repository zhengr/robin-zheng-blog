---
title: Oracle Node.js 连接Oracle ERP数据库测试
date: 2017-07-06 00:00:00
author: robin zheng
categories: 数据库
tags:
  - Oracle
  - Nodejs
  - Database
---

## Oracle-NodeJS 测试


### 安装 Node.js

Download and extract the Node.js "Linux Binaries" package. For example, if you downloaded version 6.9.4 for 64-bit you could install Node.js into /opt:
```
cd /opt
tar -Jxf node-v6.9.4-linux-x64.tar.xz
```
Set PATH to include Node.js:
```
export PATH=/opt/node-v6.9.4-linux-x64/bin:$PATH
```

### 安装 Oracle 客户端

See the Instant Client Home Page for more information. 

Installation of ZIP files: 

#### 1. Download the desired Instant Client ZIP files. All installations require the Basic or Basic Light package. 

```
drwxr-xr-x 3 root root     4096 Jul  6 10:26 instantclient_12_2/
-rw-r--r-- 1 root root 68965195 Jul  6 10:18 instantclient-basic-linux.x64-12.2.0.1.0.zip
-rw-r--r-- 1 root root   674743 Jul  6 10:17 instantclient-sdk-linux.x64-12.2.0.1.0.zip
-rw-r--r-- 1 root root   904309 Jul  6 10:17 instantclient-sqlplus-linux.x64-12.2.0.1.0.zip
-rw-r--r-- 1 root root  1132671 Jul  6 10:17 instantclient-tools-linux.x64-12.2.0.1.0.zip
```

#### 2. Unzip the packages into a single directory such as /opt/oracle/instantclient_12_2 that is accessible to your application. For example:

```
cd /opt/oracle			
unzip instantclient-basic-linux.x64-12.2.0.1.0.zip
```

#### 3. Create the appropriate libclntsh.so and libocci.so links for the version of Instant Client. For example:
```
cd /opt/oracle/instantclient_12_2
ln -s libclntsh.so.12.1 libclntsh.so
ln -s libocci.so.12.1 libocci.so
```
#### 4. Install the libaio package, for example on Oracle Linux, run this as the root user:
```
yum install libaio
-- apt-get install libaio1
```
On some Linux distributions the package is called libaio1.

#### 5. Set the environment variable LD_LIBRARY_PATH to the directory created in Step 2, for example:

```
export LD_LIBRARY_PATH=/opt/oracle/instantclient_12_2:$LD_LIBRARY_PATH
```
Alternatively, add this path to an ldconfig configuration file if there is no other Oracle software that will be impacted.

#### 6. To use supplied binaries such as SQL*Plus, update your PATH environment variable, for example:
```
export PATH=/opt/oracle/instantclient_12_2:$PATH
```
#### 7. Start your application.


### Install the add-on
Tell the installer where to find Instant Client:
```bash
export OCI_LIB_DIR=/opt/oracle/instantclient_12_2
export OCI_INC_DIR=/opt/oracle/instantclient_12_2/sdk/include

npm config set python /usr/bin/python2.7
npm install oracledb
```

编辑 [select.js](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/files/select.js)

```js
var oracledb = require('oracledb');

oracledb.getConnection(
  {
    user          : "apps",
    password      : "apps",
    connectString : "192.168.1.60:1530/VIS"
  },
  function(err, connection)
  {
    if (err) { console.error(err.message); return; }
    connection.execute(
      "SELECT WN.NOTIFICATION_ID,"
+               " WN.FROM_USER, "
+               " DECODE(WN.MORE_INFO_ROLE, "
+                      " NULL, "
+                      " WN.TO_USER, "
+                      " wf_directory.GetRoleDisplayName(WN.MORE_INFO_ROLE)) AS TO_USER, "
+               " DECODE(WN.MORE_INFO_ROLE, "
+                      " NULL, "
+                      " WN.SUBJECT, "
+                      " FND_MESSAGE.GET_STRING('FND', 'FND_MORE_INFO_REQUESTED') || ' ' || "
+                      " WN.SUBJECT) AS SUBJECT, "
+               " WN.LANGUAGE, "
+               " WN.BEGIN_DATE, "
+               " WN.DUE_DATE, "
+               " WN.STATUS, "
+               " WN.PRIORITY, "
+              " 'P' AS PRIORITY_F, "
+               "WN.RECIPIENT_ROLE, "
+              " WN.END_DATE, "
+              " WIT.DISPLAY_NAME AS TYPE, "
+              " WN.MORE_INFO_ROLE, "
+               " WN.FROM_ROLE, "
+               " WN.MESSAGE_TYPE, "
+               " WN.MESSAGE_NAME, "
+               " WN.MAIL_STATUS, "
+               " WN.ORIGINAL_RECIPIENT "
+          " FROM WF_NOTIFICATIONS WN, WF_ITEM_TYPES_TL WIT, WF_LOOKUPS_TL WL "
+         " WHERE WN.STATUS  = 'OPEN' "
+           " AND WN.message_type = WIT.name "
+           " AND WIT.language = userenv('LANG') "
+           " AND WL.lookup_type = 'WF_NOTIFICATION_STATUS' "
+           " AND WN.status = WL.lookup_code "
+           " AND WL.language = userenv('LANG') "
+           " AND WN.RECIPIENT_ROLE = 'SYSADMIN' ",
      [],  // bind value 
      function(err, result)
      {
        if (err) { console.error(err.message); return; }
        console.log(result.rows);
      });
  });
```

```bash
root@ubuntu:~# node select.js
```

