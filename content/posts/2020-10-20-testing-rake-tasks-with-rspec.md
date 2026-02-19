---
title: "Testing rake tasks with rspec"
description: "Testing rake tasks with rspec"
tags: [ruby, rubyonrails, testing]
comments: true
share: true
cover_image: '/content/images/2020/10/rspec.png'
---

This blog post is a continuation of this thread.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">On trying to write a spec for one of the rake tasks, when trying to invoke the same rake tasks within the same <a href="https://twitter.com/rspec?ref_src=twsrc%5Etfw">@rspec</a> contexts, for different flows, weirdly the tests failed if I ran the whole suite, but would pass if I ran them separately.</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1293581788455952384?ref_src=twsrc%5Etfw">August 12, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

So for example

```ruby
# ./lib/tasks/foo_task.rake
desc 'Foo task'

namespace :task do
  task :my_task, [:foo, :bar] => [:baz] do |task, args|
    ...
    # does my_task
    ...
  end
end
```

Now if we try writing a spec a for it

```ruby
# ./spec/tasks/foo_task_spec.rb
require 'rails_helper'

Rails.application.load_tasks

describe "task_my_task" do
  context "foo case" do
    let(:arg1) {"foo"}
    let(:arg2) {"baz"}

    it "it does foo behaviour" do
      Rake::Task["task:my_task"].invoke(arg1, arg2)

      # assert the expected behaviour here related for foo case
    end
  end

  context "baz case" do
    let(:arg1) {"bazbee"}
    let(:arg2) {"foobee"}

    it "it does baz behaviour" do
      Rake::Task["task:my_task"].invoke(arg1, arg2)

      # assert the expected behaviour here related for baz case
    end
  end
end
```

Now if we try to run the specs for specific contexts, where the rake task is being invoked, all will work well, but when we try to run the specs for all the contexts in the test file, the first rake task will run, but the rest of them will start failing.

Which is confusing, turns out that the tasks can be invoked only once in a given context. Not sure of the history behind this or the reasoning on why this is the case, couldn't find it. (let me know if you were able to get this bit, would be happy to learn it's history)

### How to make it work

To work around this, the task needs to be explicitly re-enabled in before the next spec is run.

Addin an `after_each` block would work for starters, if inside that block you would re-enable your task which you are trying to test out in your spec, which would mean that after each spec, inside which you are exercising your method, this routine is called. So something like

```ruby
# ./spec/tasks/foo_task_spec.rb
require 'rails_helper'

Rails.application.load_tasks

describe "task_my_task" do
  after(:each) do
    Rake::Task["task:my_task"].reenable
  end

  context "foo case" do
    let(:arg1) {"foo"}
    let(:arg2) {"baz"}

    it "it does foo behaviour" do
      Rake::Task["task:my_task"].invoke(arg1, arg2)

      # assert the expected behaviour here related for foo case
    end
  end

  context "baz case" do
    let(:arg1) {"bazbee"}
    let(:arg2) {"foobee"}

    it "it does baz behaviour" do
      Rake::Task["task:my_task"].invoke(arg1, arg2)

      # assert the expected behaviour here related for baz case
    end
  end
end
```

`reenable` first resets the task's `already_invoked` state, allowing the task to then be executed again with all it's dependencies.

There are `execute` and `invoke` methods too which are mentioned in this [SO answer](https://stackoverflow.com/a/32382929). No preference as such to why I went with `reenable`.

Running all the specs would work now.

### References

- [https://relishapp.com/rspec/rspec-core/v/2-2/docs/hooks/before-and-after-hooks](https://relishapp.com/rspec/rspec-core/v/2-2/docs/hooks/before-and-after-hooks)
- [https://ruby-doc.org/stdlib-2.0.0/libdoc/rake/rdoc/Rake/Task.html#method-i-reenable](https://ruby-doc.org/stdlib-2.0.0/libdoc/rake/rdoc/Rake/Task.html#method-i-reenable)
- [https://stackoverflow.com/questions/32381017/running-rake-tasks-in-rspec-multiple-times-returns-nil](https://stackoverflow.com/questions/32381017/running-rake-tasks-in-rspec-multiple-times-returns-nil)

### Credits

- Picture credits to [https://rspec.info/](https://rspec.info/)
