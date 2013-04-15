---
layout: post
title: "Add Google Analytics to Enki blog (i.e. a rails 3 application)"
date: 2011-11-20 14:23:00
comments: true
categories: [Ruby, RoR, Old Enki Blog]
---
<span itemprop="description">This blog is powered by [Enki](http://www.enkiblog.com/) and I'd like to monitor the (absence of?) frequentation. As a rails 3 application, integrating [Google Analytics](http://www.google.com/analytics/index.html) is dead simple thanks to the [rack-google_analytics ruby gem](http://rubygems.org/gems/rack-google_analytics) and [the associated documentation](https://github.com/ambethia/rack-google_analytics#readme)</span>

Just add the following to your `Gemfile`:

{% codeblock lang:ruby %}
group :production do
  gem 'rack-google_analytics', :require => "rack/google_analytics"
end
{% endcodeblock %}

and this conﬁguration entry to your `config/environments/production.rb`:

{% codeblock lang:ruby %}
# Configure Rack::GoogleAnalytics
config.middleware.use("Rack::GoogleAnalytics", :web_property_id => "UA-12345678-1")
{% endcodeblock %}

with the `web_property_id` provided by Analytics.

I told you: dead simple!
