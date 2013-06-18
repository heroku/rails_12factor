# Rails on Heroku

Makes running your Rails app on Heroku easier.

## What

This gem enables serving assets in production, and setting your logger to standard out. Both of which are required for Rails4

## Install

In your `Gemfile` add:

```
gem 'rails_12factor'
```

Then run

```
$ bundle install
```

Now you're good to go. See the Heroku Quick start guide for information on deploying


## How

This gem adds two other gems `rails_serve_static_assets` and `rails_stdout_logging`. Which are required to run your Rails app on Heroku if you want to use logplex and serve your assets in production. Here is how they work:

## Rails 4 Logging

By default Rails writes its logs to a file which is convenient but only you only have one log file to tail. When you start scaling your app to multiple machines or dynos then finding a single request or failure across multiple files becomes much harder. Storing logs on disk can also take down a server if the hard drive fills up. Because of these limitations: every Rails core member we talked to uses a custom logger to replace Rail's default functionality. By using the `rails_stdout_logging` gem with Heroku, we set the logger for you.

The gem `rails_stdout_logging` ensures that your logs will be sent to standard out, from there Heroku sends them to [logplex](https://github.com/heroku/logplex) so you can access them from the command line, `$ heroku logs --tail`, or from enabled addons like [papertrail](https://addons.heroku.com/papertrail). By using Heroku's logplex, you can [treat logs as event streams](http://www.12factor.net/logs).


## Rails 4 Serve Static Assets

In the default Rails development environment assets are served through a middleware called [sprockets](https://github.com/sstephenson/sprockets). In production however most non-heroku Rails deployments will put their ruby server behind reverse HTTP proxy server such as Nginx which can load balance their sites and can serve static files directly. When Nginx sees a request for an asset such as `/assets/rails.png` it will grab it from disk at `/public/assets/rails.png` and serve it. The Rails server will never even sees the request.

On Heroku, Nginx is not needed to run your application. Our [routing layer](https://devcenter.heroku.com/articles/http-routing) handles load balancing while you scale out horizontally. The caching behavior of Nginx is not needed if your application is serving static assets through an [edge caching CDN](https://en.wikipedia.org/wiki/Content_delivery_network).

By default Rails4 will return a 404 if an asset is not handled via an external proxy such as Nginx. While this default behavior will help you debug your Nginx configuration, it makes a default Rails app with assets unusable on Heroku. To fix this we've released a gem `rails_serve_static_assets`.

This gem, `rails_serve_static_assets`, enables your Rails server to deliver your assets instead of returning a 404. You can use this to populate an edge cache CDN, or serve files directly from your web app. This gives your app total control and allows you to do things like redirects, or setting headers in your Ruby code. To enable this behavior in your app we only need to set this one configuration option through this gem:

```
config.serve_static_assets = true
```

Note: this gem will set this value for you, you don't need to change any configuration manually.

All you need to do to get this functionality of both gems is add the `rails_12factor` gem to your project.

## Why?

Why do you need to include this gem in Rails 4 and not Rails 3? Rails4 is getting rid of the concept of plugins. Before libraries were easily distributed as Gems and in the form of Engines, Rails had a folder `vendor/plugins`. Any code you put there would be initialized much like a Gem is today. This was a very simple and easy way to share and use libraries, but it wasn't very maintainable. You could use a library, and make a change locally and then deploy which makes your version incompatible from future versions. Even worse there was no concept of versioning aside from source control, so semantic versioning was out of the question. For these reasons and more Rails3 deprecated plugins. With Rails4 plugins have been removed completely. Why does this affect your app on Heroku?

In the past Heroku has used plugins as a safe way to configure your application where code was needed. While we advocate [separating config from code](http://12factor.net), this was the only option if we wanted your apps to work with no changes from you. With Rails3 Heroku will add the asset serving and standardout logging plugins to your app automatically. With Rails4, Heroku needs you to add these libraries to your Gemfile.

It is important to note that unlike Gems, plugins do not have a dependency resolution phase like what happens when you run `bundle install`. Heroku does not and will not add anything to your Gemfile on compilation.


## The Future

We will be working with Rails and the Rails core team to make future versions of Rails work on Heroku out of the box. Until then you'll need to add this gem to your project.


