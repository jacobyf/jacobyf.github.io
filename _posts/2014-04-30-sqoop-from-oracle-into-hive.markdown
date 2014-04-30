---
layout: post
title: "Sqoop from oracle into hive"
date: 2014-04-30 14:32:00
categories: [hadoop, hive]
tags: [Oracle, Hive, Sqoop]
---

> Recently, I have a good chance to use a data import tool what is called 'Sqoop' to export massive data from a Oracle database server.
At the same time, these data have to be imported into a Hive database inhabit in Hadoop. Before transferring data, I have to make some
preparation to use my tools. Let me introduce in details below.

# Step 1: Download a Oracle JDBC connector driver from Oracle official website.
Before downloading, you should check carefully the database version that you have used. While the version of JDBC driver must match 
you database version, you probably type some SQL statements to get your oracle version.

	SELECT * FROM V$version;
	# You got this below
	BANNER
	Oracle Database 11g Enterprise Edition Release 11.2.0.3.0 - 64bit Production
	PL/SQL Release 11.2.0.3.0 - Production
	CORE	11.2.0.3.0	Production
	TNS for IBM/AIX RISC System/6000: Version 11.2.0.3.0 - Production
	NLSRTL Version 11.2.0.3.0 - Production

Ok, you may get it! The oracle version you currently use is `11.2.0.3.0`! Try to google the `Oracle JDBC driver for 11g`, you may download a jar file named `ojdbc6.jar`.

# Step 2: Copy the `ojdbc6.jar` file into your `Sqoop` library path.
Copy the `ojdbc6.jar` into your library path at `SQOOP_HOME\lib`. Then, you can type such command in the shell.

	sqoop import \
		--connect jdbc:oracle:thin:@//ip:1521/schema -username user_name -password passwd \
		--table table_name
		--split-by columns


