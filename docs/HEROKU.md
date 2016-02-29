This fork of [Discourse](http://www.discourse.org/) is intended to be fully compatible with deploying to Heroku. The default instructions provision add-ons and services that do incur a cost. However, there are tips towards the end that allow you to reduce the cost for non-production instances.

Please submit any [issues](https://github.com/rwdaigle/discourse/issues) you find in the deployment or management of the app on Heroku.

## Installing locally

Assuming you already have the [Heroku Toolbelt](https://toolbelt.heroku.com/) installed:

1. `$ git clone git@github.com:rwdaigle/discourse.git && cd discourse`
1. `$ bundle install`
1. `$ cp .env.sample .env`
1. `$ bundle exec rake db:create db:migrate`
1. `$ foreman start web`
1. Open [http://localhost:5000](http://localhost:5000) to see Discourse running locally

## Deploying to Heroku

Once the app is running locally, and while still in the app directory, execute the following to deploy to Heroku:

1. Create the app on Heroku: `$ heroku create discourse-myname`
1. Provision recommended add-ons:
  1. `$ heroku addons:create heroku-postgresql`
  1. Promote your database to be the main database for your app. `color` would be `gold` if you saw `HEROKU_POSTGRESQL_GOLD_URL` after running `heroku config`
  
    `$ heroku pg:promote [color]`

  1. `$ heroku addons:create openredis`
  1. `$ heroku addons:create sendgrid`
  1. `$ heroku addons:create papertrail`
  1. `$ heroku addons:create librato --logs`
  1. `$ heroku addons:create honeybadger`
  1. `$ heroku addons:create newrelic:stark`
1. Enable recommended lab features
  1. `$ heroku labs:enable user-env-compile`
  1. `$ heroku labs:enable log-runtime-metrics`
1. Set config:
```
$ heroku config:set SECRET_TOKEN=`openssl rand -base64 32` RACK_ENV=production RUBY_GC_MALLOC_LIMIT=90000000 WEB_CONCURRENCY=2 NEW_RELIC_APP_NAME=Discourse
```
1. Deploy and scale
  1. `$ git push heroku master` (this initial deploy can take some time due to the extensive dependency list and asset compilation for an app of Discourse's size)
  1. `$ heroku run bundle exec rake db:migrate`
  1. `$ heroku ps:scale web=1 worker=1`
  1. `$ heroku open`
  1. Verify dyno startup w/ `$ heroku ps` and `$ heroku logs -t`

#### Reducing cost

The default instructions provision add-ons/services we recommend for running a production application. Developers that are just playing around, or don't need a production caliber service, can reduce their cost with a few adjustments.

1. We recommend the Honeybadger add-on to notify you of any app exceptions that occur. If you are willing to forego this service, remove it with: `$ heroku addons:remove honeybadger`
1. Openredis provides a small Redis instance, which is required for Discourse. You can provision a smaller, free, alternative if you wish: `$ heroku addons:remove openredis && heroku addons:add redistogo`

If you follow each step here, your total cost will be reduced to that of two dynos (one of which is free) - about $35/month.

## Configure Discourse

Once the app is deployed you still need to establish the admin user and set some basic settings.

1. Log into the app, using your preferred auth provider.
1. Connect to the Heroku console to make the first user an Admin: `$ heroku run console`
1. Enter the following commands.

```ruby
u = User.first
u.admin = true
u.approved = true
u.save
```

4. Set `force_hostname` to your applications Heroku domain.

    This step is required for Discourse to properly form links sent with account confirmation emails and password resets. The auto detected application url would point to an Amazon AWS instance.

    Since you can't log in yet, you can set `force_hostname` in the console.

```ruby
SiteSetting.create(:name => 'force_hostname', :data_type =>1, :value=>'yourappnamehere.herokuapp.com')
```
