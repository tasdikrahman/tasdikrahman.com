---
layout: post
title: "Testing your ansible roles using travis-CI"
description: "Testing your ansible roles using travis CI"
tags: [python, devops, ansible]
comments: true
share: true
cover_image: '/content/images/2017/04/ansible_plus_travis.png'
---

**NOTE**: __The ansible playbook written here can be found at [tasdikrahman/ansible-bootstrap-server](https://github.com/tasdikrahman/ansible-bootstrap-server)__

## Continous Integration

Simply put with each commit that you are making to shared repository, which is then verified by an automated build. This helps in detection of errors early on.

If you are new to this development style. There are [plenty](https://martinfowler.com/articles/continuousIntegration.html
) [of](http://softwareengineering.stackexchange.com/questions/198471/simple-explanation-of-continuous-integration) [places](https://www.thoughtworks.com/continuous-integration) which explain will help you understand. This practice in itself is quite old. CI/CD anyone?

But I am not writing this to explain what is CI right?

## So you made an ansible playbook?

I have talked about Infra as code in some of my earlier blog posts. Automatically provisioning your complete server(s) in minutes is something which every org is trying/has achieved.

If you follow TDD principles, you would be knowing right where I am taking this conversation to.

Here is the directory structure for the ansible role I am testing this out

```bash
ansible-bootstrap-server
├── .travis.yml
├── ansible.cfg
├── play.yml
├── roles
│   ├── basic_server_hardening
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── create_new_user
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── install_minimal_packages
│   │   └── tasks
│   │       └── main.yml
│   ├── update
│   │   └── tasks
│   │       └── main.yml
│   └── vimserver
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   └── vimrc_server
│       └── tasks
│           └── main.yml
└── tests
    ├── inventory
    └── test.yml
```

**If you want to understand how the files are organised. I have written about it in ["Organising tasks in roles using Ansible"](http://www.tasdikrahman.com/2017/03/19/Organising-tasks-in-roles-using-Ansible/)**

## Writings tests

I would be running the tests inside the travis build environment.

Why travis?

I like them more! But there are many other good CI/CD providers namely circleCI, bambooCI and some more. Choose whatever fits best to your organisation or personal appeal.

So unlike when I am running the `ansible-playbook` from the controller node, I would be running the playbook on the `localhost`. This part would be obvious by now.

Let's have a look at `tests/test.yml`

```bash
---
- hosts: localhost
  connection: local
  become: true
  roles:
    - {role: ../roles/update}
    - {role: ../roles/install_minimal_packages}
    - {role: ../roles/create_new_user}
    - {role: ../roles/basic_server_hardening}
    - {role: ../roles/vimserver}
```

Let's break it down,

- `hosts: localhost` this is simply telling the host/group of hosts which this playbook would be targeting.
- `connection: local` would tell ansible to run the tasks on the system itself and not `ssh` onto to some remote machine for executing the playbook.
- `become: true` makes the tasks run as the `root` user

and the `roles` part is where we would be organising our roles to be executed sequentially.

## Testing against different versions of Ansible

For travis to build your code, it would be requiring a `.travis.yml` file in the root directory of your project.

In this particualr example, the contents of it.

```bash
---
sudo: required
dist: trusty

language: python
python: "2.7"

# Doc: https://docs.travis-ci.com/user/customizing-the-build#Build-Matrix
env:
  - ANSIBLE_VERSION=latest
  - ANSIBLE_VERSION=2.2.2.0
  - ANSIBLE_VERSION=2.2.1.0
  - ANSIBLE_VERSION=2.2.0.0
  - ANSIBLE_VERSION=2.1.5
  - ANSIBLE_VERSION=2.1.4
  - ANSIBLE_VERSION=2.1.3
  - ANSIBLE_VERSION=2.1.2
  - ANSIBLE_VERSION=2.1.1.0
  - ANSIBLE_VERSION=2.1.0.0
  - ANSIBLE_VERSION=2.0.2.0
  - ANSIBLE_VERSION=2.0.1.0
  - ANSIBLE_VERSION=2.0.0.2
  - ANSIBLE_VERSION=2.0.0.1
  - ANSIBLE_VERSION=2.0.0.0
  - ANSIBLE_VERSION=1.9.6

branches:
  only:
    - master

before_install:
  - sudo apt-get update -qq

install:
  # Install Ansible.
  - if [ "$ANSIBLE_VERSION" = "latest" ]; then pip install ansible; else pip install ansible==$ANSIBLE_VERSION; fi
  - if [ "$ANSIBLE_VERSION" = "latest" ]; then pip install ansible-lint; fi

script:
  # Check the role/playbook's syntax.
  - ansible-playbook -i tests/inventory tests/test.yml --syntax-check

  # Run the role/playbook with ansible-playbook.
  - ansible-playbook -i tests/inventory tests/test.yml -vvvv --skip-tags update,copy_host_ssh_id

  # check is the user is created or not
  - id -u tasdik | grep -q "no" && (echo "user not found" && exit 1) || (echo "user found" && exit 0)
```

The interesting thing to note here is the list arguments for `env` here. Travis expands on these environment variables to create multiple build environments one after another.

For this case we have 16 `env` variables, so there would be 16 seperate builds for the specified ansible versions which this playbook will be tested against.

So for the line

```bash
.
env:
  - ANSIBLE_VERSION=latest
.
```

The Build config will be something like, where you can see the `"env": "ANSIBLE_VERSION=latest"`

```bash
{
  "sudo": "required",
  "dist": "trusty",
  "language": "python",
  "python": "2.7",
  "env": "ANSIBLE_VERSION=latest",
  "before_install": [
    "sudo apt-get update -qq"
  ],
  "install": [
    "if [ \"$ANSIBLE_VERSION\" = \"latest\" ]; then pip install ansible; else pip install ansible==$ANSIBLE_VERSION; fi",
    "if [ \"$ANSIBLE_VERSION\" = \"latest\" ]; then pip install ansible-lint; fi"
  ],
  "script": [
    "ansible-playbook -i tests/inventory tests/test.yml --syntax-check",
    "ansible-playbook -i tests/inventory tests/test.yml -vvvv --skip-tags update,copy_host_ssh_id",
    "id -u tasdik | grep -q \"no\" && (echo \"user not found\" && exit 1) || (echo \"user found\" && exit 0)"
  ],
  "group": "stable",
  "os": "linux"
}
```

- `ansible-playbook -i tests/inventory tests/test.yml --syntax-check`:

would check for any syntax errors as obvious from the command itself, helps in checking any errors early on before the build.

- `ansible-playbook -i tests/inventory tests/test.yml -vvvv --skip-tags update,copy_host_ssh_id`:

is the line which actually runs the playbook

- `id -u tasdik | grep -q "no" && (echo "user not found" && exit 1) || (echo "user found" && exit 0)`:

checks whether a user named `tasdik` exists or not after the playbook has completed its execution.

I have skipped explaining some of the parts in my `.travis.yml`. You can learn more about [build configuration inside travis from the docs](https://docs.travis-ci.com/user/customizing-the-build)

## Skipping tasks inside build

Travis builds will fail, if you try to exceed a particular limit while your build job is running. You can find more about the exact specs and timings in the [travis docs talking about build timeouts](https://docs.travis-ci.com/user/customizing-the-build#Build-Timeouts)

Some roles in particular had tasks which would make some network calls which is unnecessary to test in a travis build. I needed to skip them in the builds

Enter ansible `tags`.

It provides an elegant way to skip or only run tasks with some specified tags.


```bash
.
"ansible-playbook -i tests/inventory tests/test.yml -vvvv --skip-tags update,copy_host_ssh_id"
.
```

The above playbook run will run the playbook `tests/test.yml` on the hosts described on `tests/inventory` skipping the tags `update,copy_host_ssh_id` which I have specified inside my tasks.

Would be trying out testing ansible roles on docker next. Stay tuned.

Happy ansibling!

**Further reading**

- [http://docs.ansible.com/ansible/playbooks_tags.html](http://docs.ansible.com/ansible/playbooks_tags.html)
- [https://docs.travis-ci.com/user/customizing-the-build#Build-Timeouts](https://docs.travis-ci.com/user/customizing-the-build#Build-Timeouts)
- [https://docs.travis-ci.com/user/customizing-the-build](https://docs.travis-ci.com/user/customizing-the-build)
- [https://www.jeffgeerling.com/blog/testing-ansible-roles-travis-ci-github](https://www.jeffgeerling.com/blog/testing-ansible-roles-travis-ci-github)
