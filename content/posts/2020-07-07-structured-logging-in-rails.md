---
title: "Structured logging in Rails"
description: "Structured logging in Rails"
tags: [ruby, rubyonrails, logging]
comments: true
share: true
cover_image: '/content/images/2020/07/rails_logging.jpeg'
---

> This post was originally published in [Gojeks engineering blog, here](https://blog.gojekengineering.com/structured-logging-in-rails-75e9a8c5370b), this post is a cross post of the same

If you are on rails, you would have noticed that the rails logs which you get by default are quite verbose and spread across multiple lines, even if the context is of processing just one simple controller action.

What I will discuss in this post is how can one sanitize the logs, without losing out on information along with how you can add additional information for your log lines to make full use of the querying features of your logging platform.

What is not in scope of this blog is setting up the mechanism to push the logs from your rails app to your respective logging platform.

Let's take the example of a simple health check controller which you would add to just make the health check pass for your rails app deployed on kubernetes

```ruby
# app/controllers/health_check_controller.rb
class HealthCheckController < ActionController::Base
  def ping
    render json: { success: true, errors: nil, data: 'pong' }, status: :ok
  end
end
```

and for the config (shown for the development evironment for this example)

```ruby
# config/environments/development.rb
Rails.application.configure do
    config.log_tags = [:request_id]
    config.log_level = :debug
end
```

A simple route for the `GET` verb, to call the `#ping` action in the `HealthCheckController`

```ruby
# config/routes.rb
Rails.application.routes.draw do
    get '/ping', to: 'health_check#ping'
end
```

This is what the logs would look like for the route `ping`

```
[my-app-fbf8d7bfc-wk5cd] I, [2020-07-01T07:19:05.007174 #1]  INFO -- : [ec0ad0ba-dfb2-419b-bd81-5feb7dacb308] Processing by HealthCheckController#ping as HTML
[my-app-fbf8d7bfc-wk5cd] I, [2020-07-01T07:19:05.007874 #1]  INFO -- : [ec0ad0ba-dfb2-419b-bd81-5feb7dacb308] Completed 200 OK in 0ms (Views: 0.2ms)
[my-app-fbf8d7bfc-wk5cd] I, [2020-07-01T07:19:05.290929 #1]  INFO -- : [86332306-62e4-412e-a690-eee8253ab1c8] Started GET "/ping" for 10.177.3.1 at 2020-07-01 07:19:05 +0000
[my-app-fbf8d7bfc-wk5cd] I, [2020-07-01T07:19:05.292363 #1]  INFO -- : [86332306-62e4-412e-a690-eee8253ab1c8] Processing by HealthCheckController#ping as HTML
```

You can notice a few things here,

- the logs are spread over multiple lines, adding to the difficulty in debugging the whole request/response flow.
    - given you would be pushing to your logging platform, let's say EFK, which would allow you to do full text search, the configuration to have the request-id for each log would come handy. A little better than not having anything at all. (Have a look at [baritolog](https://github.com/BaritoLog) if you haven't, our in house EFK platform)
    - if you are not pushing to any logging platform, then you would be debugging this by tailing the logs of the rails app somewhere, if deployed to kubernetes, doing a [kubetail](https://github.com/johanhaleby/kubetail) or if on VMs, sitting inside the VM and then tailing the logs of each and every application server instance. But then the first step here would be to obviously have centralized logging.
- extremely verbose, hindering debugging, if you don't have that already.
- no clear way to parse the logs as the logs format are non-standard (if you hadn't added `config.log_tags = [:request_id]` which is not present by default)

## Ways to improve this

Taking things one at a time, you can compact out the log so that it stays meaningful and not verbose the way it is right now, by using something like [lograge](https://github.com/roidrage/lograge). There are a bunch of other alternatives like [https://logger.rocketjob.io/](https://logger.rocketjob.io/) and [https://github.com/shadabahmed/logstasher](https://github.com/shadabahmed/logstasher), but I went ahead with lograge since it has been around for sometime now and has a larger adoption, my use case of what I required to do worked, and the use case for this as such was just to have certain things injected in our logs along with json formatted logging, and this worked very well for our use case.

Add lograge in your `Gemfile`

```
...
gem 'lograge', '~> 0.11.2'
...
```

and do a `$ bundle`, after which you need to add setup the configuration as followed

```ruby
# config/environments/developement.rb
Rails.application.configure do
  config.lograge.formatter = Lograge::Formatters::Json.new
  config.lograge.enabled = true
  config.lograge.base_controller_class = ['ActionController::Base']
  config.lograge.custom_options = lambda do |event|
    {
      request_time: Time.now,
      application: Rails.application.class.parent_name,
      process_id: Process.pid,
      host: event.payload[:host],
      remote_ip: event.payload[:remote_ip],
      ip: event.payload[:ip],
      x_forwarded_for: event.payload[:x_forwarded_for],
      params: event.payload[:params].except(*exceptions).to_json,
      rails_env: Rails.env,
      exception: event.payload[:exception]&.first,
      request_id: event.payload[:headers]['action_dispatch.request_id'],
    }.compact
  end
end
```

`config.lograge.base_controller_class = ['ActionController::Base']`, this part assumes that each controller will be inheriting from `ActionController::Base`, you can include any other controller which doesn't inherit from the Base controller, in order for lograge to pick it up.

along with

```ruby
class ApplicationController < ActionController::Base
    protect_from_forgery with: exception

    def append_info_to_payload(payload)
        super
        payload[:host] = request.host
        payload[:remote_ip] = request.remote_ip
        payload[:ip] = request.ip
    end
end
```

now if you do a

```
$ curl -I localhost:3000/ping | grep -i "request"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
X-Request-Id: 4967a677-ab86-4a10-8a01-ea520951cf46
```

and check the logs of your rails app

```json
{"level":"INFO","progname":null,"message":"{\"method\":\"GET\",\"path\":\"/ping\",\"format\":\"*/*\",\"controller\":\"HealthCheckController\",\"action\":\"ping\",\"status\":200,\"duration\":8.36,\"view\":0.22,\"db\":0.0,\"request_time\":\"2020-07-07 16:10:17 +0530\",\"application\":\"MyApplication\",\"process_id\":7869,\"params\":\"{}\",\"rails_env\":\"development\",\"request_id\":\"4967a677-ab86-4a10-8a01-ea520951cf46\"}"}
```

Now the whole log line captures a lot of metadata so that you can debug for each log line, you can also notice that one of the keys present in the log line is the `request_id` present, in this way, what is happening is, you have a unique trace id for your particular request, which you can do a full text search on your logging platform, if it supports it.

To extend this further, what one can do is capture the `request_id`, and pass it along the each controller's flow, to capture it, you can simply do a `request.request_id`. Not that you have the `request_id`, if you are doing a `Rails.logger.{info|debug}` you can use it to log the `request_id`, this way, the request_id's generated would now also be propagated to the custom logs which you would be adding to your application. The biggest advantage is, now you can add this `request_id` to each and `Rails.logger.{info|debug}` every core flow, which would be logged now, what this gives you is, you can just put the `request_id` in your search param in your logging platform, and it will give you all the logs which has this key in it.

You would also be able to capture additional information like host, remote_ip, ip of the rails app servicing it. by simply doing `request.host`, `request.remote_ip`, `request.ip`, `request.request_id`

### Why not use lograge for even the custom logger?

Lograge doesn't support [this](https://github.com/roidrage/lograge/issues/136).

### What should I do now to add structured logging for my custom logs

You can add a custom logger for your application and initialize it in your application config for the environments wherever you want it.

only thing you need to add is

```ruby
# app/logger/log_formatter.rb
class LogFormatter < ::Logger::Formatter
  def call(severity, time, program_name, message)
    message = '' if message.blank?
    severity = '' if message.blank?

    {
      level: severity,
      progname: program_name,
      message: message,
    }.to_json + "\r\n"
  end
end
```

initialize it in the application config (development here in this case for this example)

```ruby
# config/environments/developement.rb
Rails.application.configure do
    config.log_formatter = LogFormatter.new
end
```

and just to test the above out

```ruby
# app/controllers/health_check_controller.rb
class HealthCheckController < ActionController::Base
  def ping
    Rails.logger.info("bazbar, request-id: #{request.request_id}")
    render json: { success: true, errors: nil, data: 'pong' }, status: :ok
  end
end
```

## How will my logs look like after this?

```
{"level":"INFO","progname":null,"message":"bazbar, request_id: 4967a677-ab86-4a10-8a01-ea520951cqw3"}
{"level":"INFO","progname":null,"message":"{\"method\":\"GET\",\"path\":\"/ping\",\"format\":\"*/*\",\"controller\":\"HealthCheckController\",\"action\":\"ping\",\"status\":200,\"duration\":8.36,\"view\":0.22,\"db\":0.0,\"request_time\":\"2020-07-07 16:10:17 +0530\",\"application\":\"MyApplication\",\"process_id\":7869,\"params\":\"{}\",\"rails_env\":\"development\",\"request_id\":\"4967a677-ab86-4a10-8a01-ea520951cqw3\"}"}
```

As you can see, the logs from both
- the controller logs which lograge is showing
- the `Rails.logger.info` is spitting

are having

- logs in json format
- log line has the `request_id` appended to it in the message key

That's all folks

## References

- [https://github.com/roidrage/lograge](https://github.com/roidrage/lograge)
- [https://www.paperplanes.de/2012/3/14/on-notifications-logsubscribers-and-bringing-sanity-to-rails-logging.html](https://www.paperplanes.de/2012/3/14/on-notifications-logsubscribers-and-bringing-sanity-to-rails-logging.html)
- [https://github.com/BaritoLog](https://github.com/BaritoLog)
- [https://stripe.com/blog/canonical-log-lines](https://stripe.com/blog/canonical-log-lines)
- [https://github.com/roidrage/lograge/issues/136](https://github.com/roidrage/lograge/issues/136)
- [https://medium.com/better-programming/ruby-on-rails-single-line-logging-5a76852de1d2/](https://medium.com/better-programming/ruby-on-rails-single-line-logging-5a76852de1d2/)

## Credits

- Image credits [https://pixabay.com/photos/logs-timber-wood-logging-lumber-690888/](https://pixabay.com/photos/logs-timber-wood-logging-lumber-690888/) and [https://en.wikipedia.org/wiki/Ruby_on_Rails](https://en.wikipedia.org/wiki/Ruby_on_Rails)
