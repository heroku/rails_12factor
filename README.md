# Rails 12factor [![Build Status](https://travis-ci.org/heroku/rails_12factor.png)](https://travis-ci.org/heroku/rails_12factor)

Makes running your Rails 4 (or 3) app easier. Based on the ideas behind [12factor.net](http://12factor.net)

If you are starting a new application with Rails 5, you do not need this gem. You can [remove the need for this gem](#migrating-to-rails-5) if you upgrade from Rails 4 to 5.

## What

Rails gets a lot right when it comes to twelve-factor apps, but it could still be better. The two biggest areas right now are that in production [logs should be directed to stdout](http://www.12factor.net/logs) and [dev/prod parity](http://www.12factor.net/dev-prod-parity) while delivering assets.

This gem enables serving assets in production and setting your logger to standard out, both of which are required to run a Rails 4 application on a twelve-factor provider. The gem also makes the appropriate changes for Rails 3 apps.

## Install

In your `Gemfile` add:

```ruby
gem 'rails_12factor', group: :production
```

Then run

```sh
$ bundle install
```

Now you're good to go.

## How

This gem adds two other gems `rails_serve_static_assets` and `rails_stdout_logging`. These gems are required to run your Rails app with both logging aggregation and static assets serving in production. All you need to do to get the functionality of both gems is to add the `rails_12factor` gem to your project. Here is how they work:

## Rails 4 Logging

By default Rails writes its logs to a specific file, which is convenient if you only have one log file to tail. When you start scaling to multiple instances running your app, finding a single request or failure across multiple files becomes much harder. Storing logs on disk can also take down a server if the hard drive fills up. Because of these limitations, every Rails core member we’ve  talked to uses a custom logger to replace Rails' default functionality. By using the `rails_stdout_logging` gem, the logger is set for you.

The gem `rails_stdout_logging` ensures that your logs will be sent to standard out, and from there the twelve-factor platform can send them to a log aggregation service ( like  [logplex](https://github.com/heroku/logplex) on Heroku, or [Papertrail](https://papertrailapp.com)) so you can access them from one place. By using stdout instead of files, you can [treat logs as event streams](http://www.12factor.net/logs).


## Rails 4 Serve Static Assets

In the default Rails development environment assets are served through a middleware called [sprockets](https://github.com/sstephenson/sprockets). In production however most one-off Rails deployments will put their ruby server behind a reverse HTTP proxy server such as Nginx, which can then load balance the app and serve static files directly. When Nginx sees a request for an asset such as `/assets/rails.png` it will grab it from disk at `/public/assets/rails.png` and serve it. The Rails server will never see these requests.

On a twelve-factor platform, Nginx is typically not required to run your application. Your app should be capable of handling requests directly, or through a [routing layer](https://devcenter.heroku.com/articles/http-routing) that may handle load balancing while you scale out horizontally. Note that the caching behavior of Nginx is not needed if your application is serving static assets through an [edge caching CDN](https://en.wikipedia.org/wiki/Content_delivery_network), which is generally recommended.

By default Rails 4 will return a 404 if an asset is not handled via an external proxy such as Nginx. While this default behavior may help you debug your Nginx configuration, it makes a default Rails app with assets unusable on a twelve-factor platform. To fix this we've released a gem: `rails_serve_static_assets`.

The `rails_serve_static_assets` gem enables your Rails server to deliver your assets directly, instead of returning a 404. You can use this to populate an edge cache CDN, or serve files directly from your web app. This gives your app total control, allowing you to do things like redirects or setting headers in your Ruby code. The gem achieves this behavior in your app by simply setting a single configuration option, `config.serve_static_assets = true`. By using the `rails_serve_static_assets` gem, you do not need to set this configuration manually.


## Rails 5

We worked with the Rails core team to make Rails 5 work on twelve-factor platforms out of the box.

### New Rails 5 Apps

If you are starting a new application with Rails 5, **you do not need this gem.**

### Migrating to Rails 5

You can remove this gem after making sure the following sections are added in
your `production.rb` file:

**`config/environments/production.rb`**
```ruby
# Disable serving static files from the `/public` folder by default since
# Apache or NGINX already handles this.
config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?

if ENV["RAILS_LOG_TO_STDOUT"].present?
  logger           = ActiveSupport::Logger.new(STDOUT)
  logger.formatter = config.log_formatter
  config.logger = ActiveSupport::TaggedLogging.new(logger)
end
```

Make sure to add both the `RAILS_SERVE_STATIC_FILES` and `RAILS_LOG_TO_STDOUT` ENV vars and set them to `true`. (This is done for you on Heroku)
