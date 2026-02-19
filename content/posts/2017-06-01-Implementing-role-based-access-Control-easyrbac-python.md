---
title: "Implementing Role Based Access Control"
description: "Implementing Role Based Access Control"
tags: [python]
comments: true
share: true
cover_image: '/content/images/2017/06/rbac_header.jpg'
---

> You can find the python module for implementing RBAC0 here at [https://github.com/tasdikrahman/easyrbac](https://github.com/tasdikrahman/easyrbac)

## Main Idea behind it

If I have some 100 users in my system and for each user. I need to have some form of ACL using which the system makes choices whether they should be having authorisation for different actions on resources. Meaning, only the actors should be able to perform only those actions for which they are having authorisation.

How do you solve that?

Do you remember your English class ** *teacher* **? I sure do. She used to tell all kinds of interesting facts in and around Indian history. Anyways

Do you see, the word teacher here? What is that?

A Role? So whoever was a ** *teacher* ** had a `role` as a teacher in the school? 

What permissions/privileges did they have on the resources? Were they needed to be as assigned special permissions/authorisations on per teacher basis(yes, for let's say the CS teacher gets access to the CS labs at any time but that would be an exception). More or less they had a lot of responsibilities or so as to speak privileges common among themselves

So it would be common sense to group them (teachers) together and create an entity(role in this case).

How does this help?

Now instead of defining some 100 rules for 100 teachers, I can create a `Role` called `teacher` and create users which would be assigned the role of a teacher. 

This way I can easily manage the permissions for all the 100 teachers without getting repetitive as now I would just need to edit rules at the role level and not the 100 Users which were assigned the role of a teacher.

I also get the freedom to easily delete a role from a user in a cleaner manner. Imagine writing individual ACL policies for every user out there. Horror right?

Analogous to this would be `iptables`, this model does not scale very well when you have a couple more of users in your system. By practicality, your existing rules would be circus managing which would be a huge man-hour consumer. This also, increases the chance of human error while doing so. Editing and removing some user from that? Even harder.

This is what `ufw` solves for you.

## Introduction to RBAC

RBAC is Role Based Access Control, a powerful complement to traditional access control strategies and is the most manageable model (Who’s how to do what (Which)) and is (one of)the most popular access control mechanism which greatly reduces the workload of security administrators.

Here, the use of the role as an authorised intermediary, its basic idea is to access the permissions assigned to a certain role, the user by playing a different role to obtain the role of access rights have access

<center><img src="/content/images/2017/06/rbac_model.jpg"></center>

Professor Sandhu has done a great job in explaining everything. Do check his article out which I have pointed at the end. 

So I would use an RBAC for it’s

# Advantages

- Easy to manage
- Easy to classify according to work needs
- To grant the minimum privilege

## RBAC mainstream model

Layered RBAC basic model

- `RBAC0`: contains RBAC core part
- `RBAC1`: contains RBAC0, another role inheritance (RH)
- `RBAC2`: contains RBAC0, and other constraints (Constraints)
- `RBAC3`: Contains all levels of content and is a complete model

## easyrbac

Easiest way you can grok and retain all I had read, was to make something out of it. `easyrbac` was born out of it. I have tried implementing `RBAC0` for this release. The next release would focus on getting `RBAC1`, which includes role inheritance. 

`easyrbac` has a very simple API to interact around and create `Roles` and `Users`

```python
from easyrbac import Role, User

everyone_role = Role('everyone')
admin_role = Role('admin')

everyone_user = User(roles=[everyone_role])
admin_user = User(roles=[admin_role, everyone_role])
```

For User resource access permissions allocation

```python
acl = AccessControlList()

acl.resource_read_rule(everyone_role, 'GET', '/api/v1/employee/1/info')
acl.resource_delete_rule(admin_role, 'DELETE', '/api/v1/employee/1/')

# checking READ operation on resource for user `everyone_user`
for user_role in [role.get_name() for role in everyone_user.get_roles()]:
    assert acl.is_read_allowed(user_role, 'GET', '/api/v1/employee/1/info') == True

# checking WRITE operation on resource for user `everyone_user`
# Since you have not defined the rule for the particular, it will disallow any such operation by default.
for user_role in [role.get_name() for role in everyone_user.get_roles()]:
    assert acl.is_write_allowed(user_role, 'WRITE', '/api/v1/employee/1/info') == False

# checking WRITE operation on resource for user `admin_user`
for user_role in [role.get_name() for role in everyone_user.get_roles()]:
    if user_role == 'admin': # as a user can have more than one role assigned to them
        assert acl.is_delete_allowed(user_role, 'DELETE', '/api/v1/employee/1/') == True
    else:
        assert acl.is_delete_allowed(user_role, 'DELETE', '/api/v1/employee/1/') == False
```

## Future work

- Adding hierarchical roles, which represent parent<->child relations
- Adding this on top of Bottle/Flask

## Literature material

- [http://profsandhu.com/articles/advcom/adv_comp_rbac.pdf](http://profsandhu.com/articles/advcom/adv_comp_rbac.pdf)
- [http://www.comp.nus.edu.sg/~tankl/cs5322/readings/rbac1.pdf](http://www.comp.nus.edu.sg/~tankl/cs5322/readings/rbac1.pdf)

## Github repo

- [https://github.com/tasdikrahman/easyrbac](https://github.com/tasdikrahman/easyrbac)

Cheerio