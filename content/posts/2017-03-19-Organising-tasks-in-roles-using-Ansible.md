---
title: "Organising tasks in roles using Ansible"
description: "Organising tasks in roles using Ansible"
tags: [python, devops, ansible]
comments: true
share: true
cover_image: '/content/images/2017/03/front.png'
---

**NOTE**: __The ansible playbook written here can be found at [tasdikrahman/ansible-playbook](https://github.com/tasdikrahman/ansible-playbooks/tree/master/digitalocean)__

Roles are nothing but a further abstraction of making your playbook more modular. If you have played around with the `ansible-playbook` command. You might have noticed the common pattern of repeating tasks which you did some or the other time back.

Ansible roles provide you a way to reuse tasks(or roles for that matter). Imagine this to be a very similar concept writing Object oriented code.

## Need for roles?

I realised that I was doing the same thing over and over again whenever I had to spin up a new droplet(instance for the EC2 people). Things like

- Updating and upgrading the existing packages
- Installing some bare essentials on it (eg: git, vim, ncdu etc)
- creating a non-root user with admin privileges
- enabling a basic firewall
- some common chores.

Hence I found myself writing [tasdikrahman/ansible-playbook](https://github.com/tasdikrahman/ansible-playbooks/tree/master/digitalocean)

Take this structure for example.

```bash
$ tree digitalocean
digitalocean
├── README.md
├── play.yml
└── roles
    ├── bootstrap_server
    │   └── tasks
    │       └── main.yml
    ├── create_new_user
    │   └── tasks
    │       └── main.yml
    ├── update
    │   └── tasks
    │       └── main.yml
    └── vimserver
        ├── files
        │   └── vimrc_server
        └── tasks
            └── main.yml
```

Let's break it down further.

- `digitalocean` : The root dir which will contain the `roles` dir which further contains roles for tasks
- `play.yml` : A normal playbook in `.yml` format which stiches the roles that we have created inside `roles` dir
- `bootstrap_server` : contains the task file(s) needed for the necessary tasks declared inside the `tasks/main.yml`. The roles `create_new_user` et el suggest the same thing.
- `tasks` : This directory contains all of the tasks that would normally be in a playbook. These can reference files and templates contained in their respective directories without using a path.

Similar to `tasks` dir inside roles, we have many more files specifically used for other things

Those being,

- `files`: This directory contains regular files that need to be transferred to the hosts you are configuring for this role. This may also include script files to run.
- `handlers`: All handlers that were in your playbook previously can now be added into this directory.
- `meta` : This directory can contain files that establish role dependencies. You can list roles that must be applied before the current role can work correctly.
- `templates`: You can place all files that use variables to substitute information during creation in this directory.
- `vars`: Variables for the roles can be specified in this directory and used in your configuration files.

## `play.yml`

The contents of my `play.yml` are sequenced in such a manner so that the roles get executed in a sequence. This is a very handy feature which allows you to configure software which depends on some other software.

```
---
- hosts: testdroplets
  roles:
    - update
    - bootstrap_server
    - role: create_new_user
      username: tasdik
    - role: vimserver
      username: tasdik
```

The roles are placed as key-value's inside the dict roles here. the `username` variable is being passed to the roles `create_new_server` and `vimserver` as a value using which the new user should be created.

You can also put the `username` variable inside individual roles dir, which would look something like

```
└── roles
    └── create_new_user
        ├── vars
        │   └── main.yml
        └── tasks
            └── main.yml
```

The `vars/main.yml` would contain

```
---
username: tasdik
```

But I feel this would become a cumbersome task for the ansible-playbook at question. I would again have to put the same `vars/main.yml` inside the role `vimserver` which would be a duplicate of this file.

For that reason, I am passing the variable at the play level for each of the ansible roles.

## How does Jinja2 come into play here?

Have a look at the file

```bash
$ cat digitalocean/roles/vimserver/tasks/main.yml
---
- name: Place vimrc_server on ~/.vimrc
  copy:
    src: vimrc_server
    dest: /home/{ username }/.vimrc
    mode: 0644
    owner: "{ username }"
```

The `"{ username }"` is substitued with the value that you provided to the role at runtime. The reason we have put quotes around it is because the yaml parser for ansible requires it so when you are just starting the jinja variable as the starting value.

You needn't have put the quotes if for example it were something like this.

```bash
- name: Copy .ssh/id_rsa from host box to the remote box for user username
  become: true
  copy:
    src: ~/.ssh/id_rsa.pub
    dest: /home/{ username }/.ssh/authorized_keys
```

**NOTE: Please note that there are double curly braces in the above example which surround `username`. The templating engine wasn't allowing me to put it there as it would take it as a variable for substitution.**

## Closing thoughts

Ansible uses Jinja2 as the templating engine of choice(also the choice of the biggies Flask and Django). So you would be in familair territory if you have dabbled in those.

You can finally run this playbook using

```bash
$ ansible-playbook play.yml -vvv
```

Happy ansibling!
