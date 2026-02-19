---
title: "A few things about database migrations"
description: "A few things about database migrations"
tags: [rubyonrails, golang, databases]
comments: true
share: true
cover_image: '/content/images/2020/10/migration.jpg'
---

This blog post is a continuation of these two threads.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">A few things about database schema changes. (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1317490663613624320?ref_src=twsrc%5Etfw">October 17, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This is where <a href="https://twitter.com/rails?ref_src=twsrc%5Etfw">@rails</a> active record migrations really shine. I find it&#39;s UX super clean. (1/n)<a href="https://t.co/vA6Jb345yc">https://t.co/vA6Jb345yc</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1317695102924451840?ref_src=twsrc%5Etfw">October 18, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The schema of your relational database, will change over time for your application. Trying to introduce these changes from dev setup -> integration/UAT -> production env, in a clear, consistent and repeatable manner, would definitely add value in trying to maintain repeatability across different environments.

## Ways to introduce changes to your database

If someone is introducing the schema changes manually in these environments, tracking such changes and reproducing them becomes a task.

What about auditing what ran, or what was changed? Who introduced it? What happens when a schema change was made and you were not aware of it in the integration env (fill anything else here for that matter), which is also making your current build fail, but you have already wasted some time trying to debug it?

The time lost in debugging such issues, could be utilized somewhere else. The other thing to note here being, that using a single database for the integration environment, where multiple developers will deploy their codebases and introduce changes to the schema, is one way or the other, gonna bite the team down the line.

The ability to be able to recreate the structure of your database, consistently across local and dev environments will allow to iterate faster for sure. Having all the DDL and DML scripts, with a sequence, checked into your VCS allows one to at least track what has been run.

## Tracking how your schema is evolving

But now how do you track which DDL/DML scripts ran for a particular environment and it's database?

A very simple implementation to solve this is described [here](http://blog.cherouvim.com/a-table-that-should-exist-in-all-projects-with-a-database/) where you have a table specifically to track this,which records the sequence of the DDL/DML script, which last ran, the time when it was run & a small description mentioning what is it doing.

And whenever someone is running any DDL/DML command, they would also need to insert into this table, helping in keeping track what's the last status of the database for that environment.

There are much more mature & well tested ways to do this, like active record migrations in [rails](https://rubyonrails.org), [Flyway](https://flywaydb.org/) in java world, [golang-migrate](https://github.com/golang-migrate/migrate/) in golang et al. The root of it being, having a way to know what ran where & having a repeatable way to setup the schema of your database across different environments.

This is where rails active record migrations really shine. I find it's UX super clean.

All migration files, along with the schema of your database is also checked into the VCS along with your business logic, prefixed with a UTC timestamp, the timestamp being in the `schema_migrations` table, which active record would maintain inside your database.

The idea is the same. Track the status of which DDL/DML scripts have run for your application.

Want to see, what's the status of your database and what has run on it? `rails db:migrate:status` is gonna give you all the status of all the migration scripts and the status of whether it has already been applied or not.

Want to roll back the migration which was run on your database? `rails db:rollback STEP=n`, to rollback n versions of the migration id which have already been applied. The n being any integer value, of the number of migration files you want to roll back by.

Want to redo a migration which was rolled back/run it again? `rails db:migrate:redo VERSION=<UTC-timestamp-prefix-of-migration-file>`.

A similar approach can be found in [golang-migrate](https://github.com/golang-migrate/migrate), where each schema change is introduced with an SQL script numbered sequentially starting from 1 for example (the 1 can be any 64 bit unsigned integer), and ending with a suffix of up.sql and down.sql

Each new schema change will be added in a new SQL file, and numbered accordingly. Users would then add the helper methods provided by golang-migrate, for running migrations into the cli interface for their app, for both applying the migrations and to rollback them.

Both the examples, we can infer that the central theme is the same, which is to keep track of what ran and what did has not run along with a way, either via timestamps or via simple count to keep track of the order of the migration scripts

Another thing to notice here is that, both of them encourage you/give you mechanisms to write the schema change such that it is possible to reverse it to a previous state.

## Constraints which can be put on the database

Rails allows one to introduce validations on models before persisting the object to the database. But it's also important to have the same validation wherever you can on your schema. Allowing both the ORM and the database to enforce validations.

The model.[update](https://apidock.com/rails/ActiveRecord/Persistence/update) method, does this for you. It will do the validations and the callbacks required for example, along with updated_at/updated_on for you, whenever you are trying to update the attributes for the object.

This will immediately help, in having the 1st-order check to prevent inserting something you shouldn't have. The 2nd-order check being in your database schema itself. Being present as your final guard for the entries to not be dirty. While I don't remember anyone telling that it is a rule of thumb to have both, but it's not either uncommon to stumble upon this either. You may argue that it goes against [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), but I feel having the final validation on the schema of the database, definitely acts as the final source of truth where you can always fallback on and there's no downside to it.

A very simple example of this can be, when you are trying to put a check on a column in your database to ever not be a null value, even if your model has a validation to protect against inserting a null value, it can be backed by a database constraint at the same time for the database for enforce this at the end too along with the ORM.

There are also methods like [update_column](https://apidock.com/rails/ActiveRecord/Persistence/update_column), which will straight up update the attribute which you want to insert to the database, skipping all the validations, callbacks etc. Don't use it in your DML script, as part of a migration file, unless you have a good reason to.

## Rolling out migrations for your applications

As to how to roll out migration changes to the end application? I feel since the state changes over schema changes, it is similar to doing a new deployment for your application

For example, if you are dropping a column from a table, the code changes should go first, where you remove reference to that table everywhere and then do a deploy. Do your e2e tests, check if everything is working and then in the next deployment, dropping that column.

As with all changes, I feel iterative and small changes are always easier to make sense of or debug if an issue arises, given the changelog will be easier to grok through and pin-point easily what could have introduced the issue.

As I learn more on this, I would love to hear on resources to read more on this topic/processes/techniques which you folks follow, which have worked well for you.

### References

- [https://web.archive.org/web/20111115150123/http://blog.stelligent.com:80/integrate-button/2010/07/database-integration-in-your-build-scripts.html](https://web.archive.org/web/20111115150123/http://blog.stelligent.com:80/integrate-button/2010/07/database-integration-in-your-build-scripts.html)
- [https://odetocode.com/blogs/scott/archive/2008/01/30/three-rules-for-database-work.aspx](https://odetocode.com/blogs/scott/archive/2008/01/30/three-rules-for-database-work.aspx)
- [https://odetocode.com/blogs/scott/archive/2008/01/31/versioning-databases-the-baseline.aspx](https://odetocode.com/blogs/scott/archive/2008/01/31/versioning-databases-the-baseline.aspx)
- [http://blog.cherouvim.com/a-table-that-should-exist-in-all-projects-with-a-database/](http://blog.cherouvim.com/a-table-that-should-exist-in-all-projects-with-a-database/)
- [https://dba.stackexchange.com/questions/2/how-can-a-group-track-database-schema-changes](https://dba.stackexchange.com/questions/2/how-can-a-group-track-database-schema-changes)
- [https://github.com/golang-migrate/migrate/blob/master/MIGRATIONS.md](https://github.com/golang-migrate/migrate/blob/master/MIGRATIONS.md)
- [https://edgeguides.rubyonrails.org/active_record_migrations.html](https://edgeguides.rubyonrails.org/active_record_migrations.html)
- [https://www.martinfowler.com/articles/evodb.html](https://www.martinfowler.com/articles/evodb.html)
- [https://thoughtbot.com/blog/validation-database-constraint-or-both](https://thoughtbot.com/blog/validation-database-constraint-or-both)

### Credits

- Cover image credits to Shripal Dapthary. [Source](https://unsplash.com/photos/3BbEYiIV7bo)
