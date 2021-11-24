---
title: 重建 Oracle EM For EBS R12
date: 2017-07-04 00:00:00
author: robin zheng
categories: 数据库
tags:
  - Oracle
  - EM
  - EBS
---


## 重建Oracle EM For EBS R12

Oracle EBS R12.1.1使用的是Oracle Database 11R1数据库,但是Oracle Database 11R1无法用像10g以前的版本那样的通过文件就可以打开alter<SID>.log,只能通过EM查看alter，而在安装EBS R12时，默认是
不安装EM.导致无法查看Alter的信息.

下面就介绍如何手工启动EM:
### 一、检查参数remote_login_passwordfile是否为EXCLUSIVE或SHARED

##### 1).以sys用户登录

```
$sqlplus /nolog
SQL>conn /as sysdba
SQL>show parameter remote_login_passwordfile;
NAME TYPE VALUE
--------------------------- ----------- -----------
remote_login_passwordfile string EXCLUSIVE
```

##### 2).如果VALUE的值不是EXCLUSIVE，可以用如下的命令修 emote_login_passwordfile的值
```
SQL>alter system set remote_login_passwordfile='exclusive' scope=spfile;
```
注:然后重启Database;

### 二、检查是否存在用户被授予了sysdba或者sysoper权限

```
SQL> select * from v$pwfile_users;
USERNAME SYSDB SYSOP SYSAS
---------------- ----- ----- -----
SYS TRUE TRUE FALSE
```

如果全为空，请用如下命令创建密码文件:
```
orapwd file=$ORACLE_HOME/dbs/orapw$ORACLE_SID password=sys entries=5;
```
### 三、删除默认安装测试版的EM和重建EM

##### 1).Delete DB Control Repository
```
$emca -deconfig dbcontrol db -repos drop
STARTED EMCA at Mar 19, 2010 10:51:30 AM
EM Configuration Assistant, Version 11.1.0.7.0 Production
Copyright (c) 2003, 2005, Oracle. All rights reserved.
Enter the following information:
Database SID: <ORACLE_SID>
Listener port number: <ORACLE_PORT>
Password for SYSMAN user:<sysman passwd>
Do you wish to continue? [yes(Y)/no(N)]: y
……
```

##### 2).Delete DB Control Repository Objects Manually
```
SQL> drop user sysman cascade;
SQL> drop role MGMT_USER;
SQL> drop user MGMT_VIEW cascade;
SQL> drop public synonym MGMT_TARGET_BLACKOUTS;
SQL> drop public synonym SETEMVIEWUSERCONTEXT;
3).create the DB Control Configuration Files and Repository
$emca -config dbcontrol db -repos create
STARTED EMCA at Mar 19, 2010 11:51:30 AM
EM Configuration Assistant, Version 11.1.0.7.0 Production
Copyright (c) 2003, 2005, Oracle. All rights reserved.
Enter the following information:
Database SID: <ORACLE_SID>
Listener port number: <ORACLE_PORT>
Password for SYS user:<sys passwd>
Password for DBSNMP user:<sysman passwd>
Password for SYSMAN user:<sysman passwd>
Password for SYSMAN user: Email address for notifications (optional):
Outgoing Mail (SMTP) server for notifications (optional):
-----------------------------------------------------------------
You have specified the following settings
Database ORACLE_HOME ................ <ORACLE HOME>
Local hostname ................ <host name>
Listener port number ................ <ORACLE PORT>
Database SID ..............
.. <ORACLE SID>
Email address for notifications ...............
Outgoing Mail (SMTP) server for notifications ...............
-----------------------------------------------------------------
Do you wish to continue? [yes(Y)/no(N)]: Y
```

### 四、启动/关闭EM
```
$emctl start/stop dbconsole
$ emctl start dbconsole
Oracle Enterprise Manager 11g Database Control Release 11.1.0.7.0
Copyright (c) 1996, 2008 Oracle Corporation. All rights reserved.
https://ebsdev.glccb.com:1158/em/console/aboutApplication
- An instance of Oracle Enterprise Manager 11g Database Control is already running.
$ emctl stop dbconsole
Oracle Enterprise Manager 11g Database Control Release 11.1.0.7.0
Copyright (c) 1996, 2008 Oracle Corporation. All rights reserved.
https://ebsdev.glccb.com:1158/em/console/aboutApplication
Stopping Oracle Enterprise Manager 11g Database Control ...
... Stopped.
You have mail in /usr/spool/mail/root
```
#### 五、oracle 控制台启动端口修改
```
$ emca -reconfig ports -dbcontrol_http_port 1159
$ emca -reconfig ports -agent_port 3939
```
克隆Oracle数据库(已经启用EM)，在异机上复活数据库系统是需要将EM删除，并重建新环境下EM，在操作前需要做如下操作：

- 修改emca文件(/d01/oracle/EBSTST01/db/tech_st/11.1.0/bin)
在emca文件中，将ORACLE_HOME修改实际路径，如下：
ORACLE_HOME="/d01/oracle/EBSTST01/db/tech_st/11.1.0"

- 修改emctl文件(/d01/oracle/EBSTST01/db/tech_st/11.1.0/bin)
在emctl文件中，将ORACLE_HOME及EM_OC4J_HOME修改为实际路径，如下：
ORACLE_HOME=/d01/oracle/EBSTST01/db/tech_st/11.1.0及EM_OC4J_HOME=/d01/oracle/EBSTST01/db/tech_st/11.1.0/oc4j/j2ee/OC4J_DBConsole

##### 问题1：

```
Sep 11, 2010 4:56:54 PM oracle.sysman.emcp.util.PlatformInterface executeCommand
CONFIG: /d01/oracle/EBSTST01/db/tech_st/11.1.0/sysman/admin/scripts/emca/emcaDbUtil[40]: /fin/db/db/tech_st/11.1.0/perl/bin/perl: n
ot found.
Sep 11, 2010 4:56:54 PM oracle.sysman.emcp.util.PlatformInterface executeCommand
CONFIG: Error executing /d01/oracle/EBSTST01/db/tech_st/11.1.0/sysman/admin/scripts/emca/emcaDbUtil /d01/oracle/EBSTST01/db/tech_st/
11.1.0/perl/lib:/d01/oracle/EBSTST01/db/tech_st/11.1.0/perl/lib/site_perl:/d01/oracle/EBSTST01/db/tech_st/11.1.0/sysman/admin/script
s:/d01/oracle/EBSTST01/db/tech_st/11.1.0/bin: /fin/db/db/tech_st/11.1.0/perl/bin/perl /d01/oracle/EBSTST01/db/tech_st/11.1.0/sysman/
admin/scripts/emca/emcaDbUtil.pl /d01/oracle/EBSTST01/db/tech_st/11.1.0 EBSTST01 SYS SYSDBA declare repos_pwd varchar2(30); view_use
r varchar2(30); view_user_pwd varchar2(30); view_user_acc_status varchar2(30); begin repos_pwd := ?; sysman.mgmt_view_priv.get_view_
user(view_user); select account_status into view_user_acc_status from sys.dba_users where upper(username) = upper ( view_user ); IF
view_user_acc_status like '%LOCKED%' THEN execute immediate 'alter user ' || view_user || ' account unlock'; END I
F; IF view_user_ac
c_status like '%EXPIRED%' THEN sysman.mgmt_view_priv.set_view_user_creds ( repos_pwd ); sysman.mgmt_view_priv.GET_VIEW_USER_CREDS (
view_user, view_user_pwd ); execute immediate 'alter user ' || view_user || ' identified by ' || view_user_pwd || ''; END IF; end; 1
SYSMAN_PWD
Sep 11, 2010 4:56:54 PM oracle.sysman.emcp.EMReposConfig unlockMGMTAccount
CONFIG: Failed to unlock mgmt_view account
Sep 11, 2010 4:56:54 PM oracle.sysman.emcp.EMReposConfig invoke
SEVERE: Failed to unlock all EM-related accounts
Sep 11, 2010 4:56:54 PM oracle.sysman.emcp.EMConfig perform
SEVERE: Failed to unlock all EM-related accounts
Refer to the log file at /d01/oracle/EBSTST01/db/tech_st/11.1.0/cfgtoollogs/emca/EBSTST01/emca_2010_09_11_16_56_25.log for more deta
ils.
Sep 11, 2010 4:56:54 PM oracle.sysman.emcp.EMConfig perform
CONFIG: Stack Trace:
oracle.sysman.emcp.exception.EMConfigException: Failed to unlock all EM-related accounts
at oracle.sysman.emcp.EMReposConfig.invoke(EMReposConfig.java:335)
at oracle.sysman.emcp.EMReposConfig.invoke(EMReposConfig.java:147)
at oracle.sysman.emcp.EMConfig.perform(EMConfig.java:222)
at oracle.sysman.emcp.EMConfigAssistant.invokeEMCA(EMConfigAssistant.java:535)
at oracle.sysman.emcp.EMConfigAssistant.performConfiguration(EMConfigAssistant.java:1215)
at oracle.sysman.emcp.EMConfigAssistant.statusMain(EMConfigAssistant.java:519)
at oracle.sysman.emcp.EMConfigAssistant.main(EMConfigAssistant.java:468)
$
```

解决方法： 

修改emca.properties，具体路径如下：

```
/d01/oracle/EBSTST01/db/tech_st/11.1.0/sysman/config/emca.properties
将emca.properties文件中PERLBIN修改为如下内容：
PERLBIN=/d01/oracle/EBSTST01/db/tech_st/11.1.0/perl/bin
```
##### 问题2：

```
CONFIG: Creating targets.xml in state directories...
Source location: /fin/db/db/tech_st/11.1.0
Destination : /d01/oracle/EBSTST/db/tech_st/11.1.0/ebsdev.glccb.com_EBSTST
Creating directories...
Creating targets.xml...
Sep 14, 2010 11:27:06 AM oracle.sysman.emcp.util.PlatformInterface executeCommand
CONFIG: Unable to open /fin/db/db/tech_st/11.1.0/sysman/emd/targets/xml
Sep 14, 2010 11:27:06 AM oracle.sysman.emcp.util.PlatformInterface executeCommand
WARNING: Error executing /d01/oracle/EBSTST/db/tech_st/11.1.0/perl/bin/perl /d01/oracle/EBSTST/db/tech_st/11.1.0/bin/targetdeploy.pl /d01/oracle/EBSTST/db/tech_st/11.1.0/ebsdev.glccb.com_EBSTST ebsdev.glccb.com EBSTST
Sep 14, 2010 11:27:06 AM oracle.sysman.emcp.EMAgentConfig instantiateEMConfigFiles
CONFIG: Failed to deploy state dirs
Sep 14, 2010 11:27:06 AM oracle.sysman.emcp.EMConfig perform
SEVERE: Error instantiating EM configuration files
Refer to the log file at /d01/oracle/EBSTST/db/tech_st/11.1.0/cfgtoollogs/emca/EBSTST/emca_2010_09_14_11_11_20.log for more details.
Sep 14, 2010 11:27:06 AM oracle.sysman.emcp.EMConfig perform
CONFIG: Stack Trace:
oracle.sysman.emcp.exception.EMConfigExcepti
on: Error instantiating EM configuration files
at oracle.sysman.emcp.EMAgentConfig.updateAgentConfigFiles(EMAgentConfig.java:3093)
at oracle.sysman.emcp.EMAgentConfig.performConfiguration(EMAgentConfig.java:1287)
at oracle.sysman.emcp.EMAgentConfig.invoke(EMAgentConfig.java:252)
at oracle.sysman.emcp.EMAgentConfig.invoke(EMAgentConfig.java:230)
at oracle.sysman.emcp.EMConfig.perform(EMConfig.java:226)
at oracle.sysman.emcp.EMConfigAssistant.invokeEMCA(EMConfigAssistant.java:535)
at oracle.sysman.emcp.EMConfigAssistant.performConfiguration(EMConfigAssistant.java:1215)
at oracle.sysman.emcp.EMConfigAssistant.statusMain(EMConfigAssistant.java:519)
at oracle.sysman.emcp.EMConfigAssistant.main(EMConfigAssistant.java:468)
```
解决方法：

找到targetdeploy.pl，具体路径如下：
/d01/oracle/EBSTST/db/tech_st/11.1.0/bin/targetdeploy.pl
将targetdeploy.pl文件中$ORACLE_HOME修改为如下内容：
$ORACLE_HOME="/d01/oracle/EBSTST01/db/tech_st/11.1.0";

