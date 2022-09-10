---
layout: post
title: "My Ramblings with Oracle-11g"
description: "Oracle 11g - My Experience"
tags: [sql, oracle11g]
comments: true
share: true
---

## Intro : 

I have recently started using `Oracle 11g` and boy, doesn't it come with enough problems already!! However, it may be due to my undying allegiance to `mysql` and `sqlite`. But whatever be the case. 

Here are some of the things which I found would be useful to a first time user of `Oracle 11g`. I will try to update it when I find something useful.

>**If you haven't already installed `Oracle 11g`. [I wrote a small article on how to do so sometime back](http://www.tasdikrahman.com/2015/09/26/Install-and-Configure-Oracle-11g-on-Ubuntu-14.10/).**

## Creating a new user : 

Now we can use the default users `SYSTEM` or `SYS`, but I prefer creating my own here.

To do that

{% highlight sql %}
tasdik@Acer:~$ sqlplus

SQL*Plus: Release 11.2.0.2.0 Production on Sat Sep 26 14:25:09 2015

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

Enter user-name: SYSTEM
Enter password: 

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> create user lab identified by lab;

User created.

## check the new user 

SQL> select username from dba_users ;

USERNAME
------------------------------
LAB
SYS
SYSTEM
ANONYMOUS
APEX_PUBLIC_USER
APEX_040000
OUTLN
XS$NULL
FLOWS_FILES
MDSYS

USERNAME
------------------------------
CTXSYS
XDB
HR

14 rows selected.

SQL> grant create session to lab ;

Grant succeeded.

SQL> grant connect to lab ;

Grant succeeded.

SQL> grant all privileges to lab ; 

Grant succeeded.

SQL> exit
Disconnected from Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
{% endhighlight %}

Now connect to the new user `lab` 

{% highlight sql %}
tasdik@Acer:~$ sqlplus

SQL*Plus: Release 11.2.0.2.0 Production on Sat Sep 26 14:34:10 2015

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

Enter user-name: lab
Enter password: 

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production


## clear the screen
SQL> cl scr

SQL> select table_name from user_tables ;

no rows selected
## as no tables have been created by the user `lab` till now
## so lets create some, shall we

SQL> CREATE TABLE department (
  2   dept_name varchar(20), 
  3   building varchar(15), 
  4   budget numeric(12,2) check (budget > 0), 
  5   primary key (dept_name)
  6  ) ; 

Table created.

SQL> select table_name from user_tables ; 

TABLE_NAME
------------------------------
DEPARTMENT

## you can do whatever you can normally do with this user
SQL> drop table department ; 

Table dropped.

SQL> select table_name from user_tables ;

no rows selected

SQL> 

{% endhighlight %}

## To Delete a User : 

After dropping the user, you need to, for each related tablespace, take it offline and drop it. For example if you had a user named 'SAMPLE' and two tablespaces called 'SAMPLE' and 'SAMPLE_INDEX', then you'd need to do the following:

{% highlight sql linenos %}
DROP USER SAMPLE CASCADE;
ALTER TABLESPACE SAMPLE OFFLINE;
DROP TABLESPACE SAMPLE INCLUDING CONTENTS;
ALTER TABLESPACE SAMPLE_INDEX OFFLINE;
DROP TABLESPACE SAMPLE_INDEX INCLUDING CONTENTS;
{% endhighlight %}

## There is no such thing as   `IF EXISTS`  :

I mean what! I find this really irritating in the part of `Oracle` to not give support for this feature as **any other major RDMS system** implements this. 


Lets try it out

{% highlight sql %}
SQL> select table_name from user_tables ; 

no rows selected

SQL>
{% endhighlight %}

So we don't have any `relations` right now. Lets try to drop `tasdik` which does not exist the way we do in `mysql`.

{% highlight sql %}
SQL> DROP TABLE IF EXISTS tasdik ; 
DROP TABLE IF EXISTS tasdik
              *
ERROR at line 1:
ORA-00933: SQL command not properly ended


SQL>
{% endhighlight %}

If you try to `DROP` a relation which does not exist, the query will not stop executing in the middle, but will just give you an error like the above

Well, `sqlite` has a limitation where we cannot `alter` the `attribute` in a `relation`, like `add` or `delete` an attribute to a relation. So nobody is perfect here.

## No support for cursor keys : 

{% highlight bash %}
tasdik@Acer:~$ sqlplus

SQL*Plus: Release 11.2.0.2.0 Production on Sun Sep 27 19:12:41 2015

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

Enter user-name: lab
Enter password: 

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> ^[[A^[[A^[[A
{% endhighlight %}

As you can see, whenever you press the cursor keys, there is garbage on the screen. Now I was not sure why this was happening. Turns out it doesn't support the arrow keys in `*nix` based systems. (Good news for those on `windows`).

But well there is a workaround.

You can use a third party utility called `rlwrap`.

>rlwrap is a readline wrapper, a small utility that uses the GNU readline library to allow the editing of keyboard input for any other command. It maintains a separate input history for each command, and can TAB-expand words using all previously seen words and/or a user-specified file.So you will be able to use arrows and also get a command history as a bonus.

After you have installed the utility run sqlplus the following way:

{% highlight bash %}
tasdik@Acer:~$ rlwrap sqlplus

SQL*Plus: Release 11.2.0.2.0 Production on Sun Sep 27 19:04:38 2015

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

Enter user-name: lab
Enter password: 

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL>
{% endhighlight %}

And the arrow keys would work just fine

## References: 

* [http://stackoverflow.com/questions/22386976/create-a-user-with-all-privileges-in-oracle](http://stackoverflow.com/questions/22386976/create-a-user-with-all-privileges-in-oracle)
* [http://stackoverflow.com/questions/205736/get-list-of-all-tables-in-oracle](http://stackoverflow.com/questions/205736/get-list-of-all-tables-in-oracle)
* [http://stackoverflow.com/questions/969245/how-to-delete-a-user-in-oracle-10-including-all-its-tablespace-and-datafiles](http://stackoverflow.com/questions/969245/how-to-delete-a-user-in-oracle-10-including-all-its-tablespace-and-datafiles)
* [http://stackoverflow.com/questions/9890636/arrow-keys-are-not-functional-in-sqlplus](http://stackoverflow.com/questions/9890636/arrow-keys-are-not-functional-in-sqlplus)