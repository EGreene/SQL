---
title: "Connect-SQL-Developer-Windows-to-Oracle-Linux"
date: 2023-09-22
---
Here is how to view your Virtual Machine Oracle on Linux (Centos 7) database through SQL Developer on a Windows host machine.
First, open a terminal and connect to your Linux OS via the virtual machine. Here I am using Centos 7 with VMWare.

```
[oracle@localhost ~]$ sqlplus / as sysdba
SQL> startup;

ORACLE instance started.

Total System Global Area 1593832624 bytes
Fixed Size		    9135280 bytes
Variable Size		 1224736768 bytes
Database Buffers	  352321536 bytes
Redo Buffers		    7639040 bytes
Database mounted.
Database opened.
```

The next step, if you are using pluggable databases like I am, is to list the pluggable databases and their status with the "show pdbs" command.

```
SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB			  MOUNTED
	 5 PDBTS			  MOUNTED
```

Now, in order to use a listener we must alter the pluggable databases to change from mounted status to open.
```
SQL> alter pluggable database all open;

Pluggable database altered.
SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB			  READ WRITE NO
	 5 PDBTS			  READ WRITE NO
```

Next, open a new terminal and start the listener control application:

I will show the results I got for my machine

```
[oracle@localhost ~]$ lsnrctl start

LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 22-SEP-2023 15:34:37

Copyright (c) 1991, 2019, Oracle.  All rights reserved.

Starting /u01/app/oracle/product/19.0.0/db_home/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 19.0.0.0.0 - Production
System parameter file is /u01/app/oracle/product/19.0.0/db_home/network/admin/listener.ora
Log messages written to /u01/app/oracle/diag/tnslsnr/localhost/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1522)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=localhost)(PORT=2484)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=db19c)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost.localdomain)(PORT=1522)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
Start Date                22-SEP-2023 15:34:37
Uptime                    0 days 0 hr. 0 min. 0 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/19.0.0/db_home/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/localhost/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1522)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=localhost)(PORT=2484)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=db19c)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521)))
The listener supports no services
The command completed successfully
```

Once you start the listener, you may have to wait about 1 minute before the listener is shows supporting services. Remember, the listener.ora file will help you configure your listener, although we may go into that later.

Now, let us check the status of the listener.
```
[oracle@localhost ~]$ lsnrctl status
```

I will skip to the services summary:

```
LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 22-SEP-2023 15:37:51
...
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1522)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=localhost)(PORT=2484)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=db19c)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521)))
Services Summary...
Service "86b637b62fdf7a65e053f706e80a27ca.com" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "e9d31348ee0d2867e055000000000001.com" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "eb66f70fd45a1f34e055000000000001.com" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "orcl.com" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "orclXDB.com" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "orclpdb.com" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "pdbts.com" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
The command completed successfully
```

Your instance name and service name will vary by how you set your oracle databases.

The next step is to switch back to Windows and open up SQL Developer. First You will want to download SQL Developer for Windows (32-bit or 64-bit depending on the machine).

![image](https://github.com/EGreene/skills-github-pages/assets/145801984/3c8e0f27-7df4-4d25-922d-843ecd717d44)

From SQL Developer in Windows, start a new connection with File->New->General->Connections
Here I will make a connection for the hr database in the orclpdb pluggable database. My database password is also hr.

![image](https://github.com/EGreene/skills-github-pages/assets/145801984/7dd00d4a-5490-4f10-8fc7-f5bec3bd8c6b)

I picked the parameters based on my database and listener control settings. Here I use port 1522 to connect. Now, if all is well, you get a message that reads "Status:Success".
However, I previouly had trouble with this step. There are several ways to work around this.
One way, which I will not focus on now, is to use SSH. The way I want to highlight, which is a bit simpler, is the Basic option.
You will have to find some parameters with the Linux machine. 
To find the hostname, log in to Linux and use the `hostname` command.

```
[oracle@localhost ~]$ hostname
localhost.localdomain
```

Now, you may find your connection not going through. One preliminary step is to adjust your firewall settings on Linux.

Use the command:
`sudo firewall-cmd --zone=public  --add-port=1522/tcp --permanent`

You will have to reload the firewall afterwards:
```
sudo firewall-cmd --reload
```

I found changing the firewall settings to be the final step to make things right.

