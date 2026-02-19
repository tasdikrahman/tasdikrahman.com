---
title: "Say Hi to peewee"
description: "Peewee: A simple ORM software"
tags: [python, orm, peewee, sql]
comments: true
share: true
cover_image: '/content/images/2016/1/peewee.png'
---

Once upon a time, when we had to interact with the databases. We had to write bare bones `SQL`(seequel if you may) or Structured Query language. A language which many common databases like

- `SQLite`
- `MySQL`
- `MariaDB` to name a few.

`SQL` is amazing but for day to day tasks it is pretty daunting and could be one of the ways in which you can shoot yourself squarely in the foot.

Now I am sure that you don't want that to happen!

## Enter ORM 

ORM is an acronym for `Object Relational Mapping`. Okay, but what does an ORM do?

> It turns object in your code to rows in your database and vice versa. 

They do this by defining a model. A model represents a `table` in the database. The models are nothing but `classes` who have their attributes represent the `columns` 

I stumbled upon [Peewee](https://en.wikipedia.org/wiki/Peewee_ORM), a lightweight ORM tool for python. 

[Github/peewee](https://github.com/coleifer/peewee)   |   [Documentation](http://docs.peewee-orm.com/)

Now you might ask, why `peewee` and not any other ORM like `SQLAlchemy` or `Storm` for the matter? 

Well my reason for that would be the closeness to the `Django` style declaration models. This would be immensely helpful for anybody who is gonna be learning/working with `Django`. Plus it's lightweight!

Here is a quick demo for how to use it.

## Peewee, A gentle intro

Let's try to model a `Student` database

{% highlight python %}
from peewee import *  

db = SqliteDatabase('student.db')

class Student(Model):
    username = CharField(max_length=255, unique=True)
    points = IntegerField(default=0)

    class Meta:     
        database = db
{% endhighlight %}

The model has been created.

Let's enter some students to the database

{% highlight python %}
students = [
    {'username': 'tasdik',
     'points': 200
     },
    {'username': 'kellogs',
     'points': 400
     },
    {'username': 'john',
     'points': 500
     },
    {'username': 'doe',
     'points': 600
     },
    {'username': 'foo',
     'points': 1000
     },
]
{% endhighlight %}

How about we make the process of creating a seperate function for adding the students to the database?

{% highlight python %}
def add_student():
    for student in students:
        try:    
            Student.create(
                username=student['username'],
                points=student['points']
            )
        except IntegrityError: 
            student_record = Student.get(username=student['username'])
            if student['points'] != student_record.points:
                student_record.points = Student.get(points=student['points'])
            student_record.save()
{% endhighlight %}

## Checking whether there is anything in there

Firing up the interpreter and running `sqlite3`

{% highlight sql %}
$ sqlite3 students.db
-- Loading resources from /home/tasdik/.sqliterc

SQLite version 3.8.6 2014-08-15 11:46:33
Enter ".help" for usage hints.
sqlite> .tables
student
sqlite> SELECT * FROM student;
id          username    points    
----------  ----------  ----------
1           tasdik      200       
2           kellogs     400       
3           john        500       
4           doe         600       
5           foo         1000      
sqlite> .exit
{% endhighlight %}


Querying the database becomes as simple as a breeze too!

## The whole thing put together

So there you go 

{% gist 99adec66eda0754d853c %}

## Some notes - If you may:

- `model` - A code object that represents a database table
- `SqliteDatabase` - The class from Peewee that lets us connect to an SQLite database
- `Model` - The Peewee class that we extend to make a model
- `CharField` - A Peewee field that holds onto characters. It's a varchar in SQL terms
- `max_length` - The maximum number of characters in a CharField
- `IntegerField` - A Peewee field that holds an integer
- `default` - A default value for the field if one isn't provided
- `unique` - Whether the value in the field can be repeated in the table
- `.connect()` - A database method that connects to the database
- `.create_tables()` - A database method to create the tables for the specified models.
- `safe` - Whether or not to throw errors if the table(s) you're attempting to create already exist
- `.create()` - creates a new instance all at once
- `.select()` - finds records in a table
- `.save()` - updates an existing row in the database
- `.get()` - finds a single record in a table
- `.delete_instance()` - deletes a single record from the table
- `.order_by()` - specify how to sort the records
- `.update()` - also something we didn't use. Offers a way to update a record without `.get()` and `.save()`
- `.where()` - method that lets us filter our .select() results
- `.contains()` - method that specifies the input should be inside the specified field

Example:

    Student.update(points=student['points']).where(Student.username == student['username']).execute()

## References:

- [Peewee query methods](http://peewee.readthedocs.org/en/latest/peewee/querying.html)
- [Stackoverflow: List of Python ORM tools](http://stackoverflow.com/questions/53428/what-are-some-good-python-orm-solutions)
- [Wiki: ORM software lists](https://en.wikipedia.org/wiki/List_of_object-relational_mapping_software)
- [strftime options](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)