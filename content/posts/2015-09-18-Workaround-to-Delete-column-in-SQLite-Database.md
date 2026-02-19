---
title: "Workaround for deleting Columns in SQLite"
description: "inserting and deleting Columns in SQLite"
tags: [sql, sqlite]
comments: true
share: true
---

## Intro : 

SQLite has limited `ALTER TABLE` support that you can use to add a column to the end of a table or to change the name of a table. If you want to make more complex changes in the structure of a table, you will have to recreate the table. You can save existing data to a temporary table, drop the old table, create the new table, then copy the data back in from the temporary table.

From the docs.

> It is not possible to rename a column, remove a column, or add or remove constraints from a table.

source : [http://www.sqlite.org/lang_altertable.html][1]

While you can always create a new table and then drop the older one.

I will try to explain this [this workaround ][2] with an example.

{% highlight sql %}
sqlite> .schema
CREATE TABLE person(
 id INTEGER PRIMARY KEY, 
 first_name TEXT,
 last_name TEXT, 
 age INTEGER, 
 height INTEGER
);
sqlite> select * from person ; 
id          first_name  last_name   age         height    
----------  ----------  ----------  ----------  ----------
0           john        doe         20          170       
1           foo         bar         25          171  
{% endhighlight %}


Now you want to remove the column `height` from this table.


Create another table called `new_person` 

{% highlight sql %}
sqlite> CREATE TABLE new_person(
   ...>  id INTEGER PRIMARY KEY, 
   ...>  first_name TEXT, 
   ...>  last_name TEXT, 
   ...>  age INTEGER 
   ...> ) ; 
sqlite> 
{% endhighlight %}


Now copy the data from the old table 


{% highlight sql %}
sqlite> INSERT INTO new_person
   ...> SELECT id, first_name, last_name, age FROM person ;
sqlite> select * from new_person ;
id          first_name  last_name   age       
----------  ----------  ----------  ----------
0           john        doe         20        
1           foo         bar         25        
sqlite>
{% endhighlight %}


Now Drop the `person` table and rename `new_person` to `person`

{% highlight sql %}
sqlite> DROP TABLE IF EXISTS person ; 
sqlite> ALTER TABLE new_person RENAME TO person ;
sqlite>
{% endhighlight %}

So now if you do a `.schema`, you will see

{% highlight sql %}
sqlite>.schema
CREATE TABLE "person"(
 id INTEGER PRIMARY KEY, 
 first_name TEXT, 
 last_name TEXT, 
 age INTEGER 
);
{% endhighlight %}


So that's how you insert and delete columns from an SQLite Database

[1]: http://www.sqlite.org/lang_altertable.html
[2]: http://www.sqlite.org/faq.html#q11

  