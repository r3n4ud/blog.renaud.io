---
layout: post
title: "Rails 3: Migrate your data from SQLite to PostgreSQL"
date: 2011-10-05 21:29:00
comments: true
categories: [Ruby, RoR, SQLite, PostgreSQL, Old Enki Blog]
---
Using my brand new enki blog instance with a SQLite database, I wrote my ﬁrst post… then I told myself that it could be nice to use a PostgreSQL database, just for fun…

Google and [a stackoverflow entry](http://stackoverflow.com/questions/1670154/convert-a-ruby-on-rails-app-from-sqlite-to-mysql) gives me the answer, here the step-by-step result for a `RAILS_ENV="production"`:

* Install PostgreSQL (see my previous post [Postgresql installation on debian Wheezy](http://blog.renaud.io/2011/10/05/postgresql-installation-on-debian-wheezy))
* `$ cp config/database.yml config.database.yml.sqlite3`
* Install [yaml_db](https://github.com/ludicast/yaml_db#readme): `$ sudo gem install yaml_db` and add that gem to your Gemﬁle
* Add the PostgreSQL support: `$ sudo gem install pg` and add that gem to your Gemﬁle
* Add a new `config/database.yml.pg` ﬁle:

{% codeblock lang:ruby %}
# PostgreSQL version
#   gem install pg
development:
  adapter: postgresql
  encoding: unicode
  database: blog_development
  pool: 5
  username: blog
  password: blurp
  host: localhost

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  adapter: postgresql
  encoding: unicode
  database: blog_test
  pool: 5
  username: blog
  password: blurp
  host: localhost

production:
  adapter: postgresql
  encoding: unicode
  database: blog_production
  pool: 5
  username: blog
  password: blurp
  host: localhost
{% endcodeblock %}

* Backup your SQLite data with yaml_db: `$ rake db:dump RAILS_ENV="production"`
* Configure Rails to use your new PostgreSQL configuration: `$ cp config/database.yml.pg config/database.yml`
* Create the database and the associated tables: `$ rake db:create RAILS_ENV="production"`
* Load your schema: `$ rake db:schema:load  RAILS_ENV="production"`
* Load your data with yaml_db:`$ rake db:load RAILS_ENV="production"`

Et voilà !
