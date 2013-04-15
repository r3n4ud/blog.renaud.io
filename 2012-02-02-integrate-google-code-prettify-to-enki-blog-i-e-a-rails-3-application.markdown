---
layout: post
title: "Integrate google-code-prettify to Enki blog (i.e. a rails 3 application)"
date: 2012-02-02 22:20:00
comments: true
categories: [Ruby, RoR, Old Enki Blog]
---
<span itemprop="description">This blog is powered by [Enki](http://www.enkiblog.com/) and as technical blog, I'd like to insert some code snippets to the entries. As a rails 3 application, integrating [google-code-prettify](http://code.google.com/p/google-code-prettify/) is relatively simple.</span>

First download the installation bundle (I experienced some difficulties, with perl, to make a distribution package from the svn):

    $ curl --progress-bar -o prettify-1-Jun-2011.tar.bz2 \
        http://google-code-prettify.googlecode.com/files/prettify-1-Jun-2011.tar.bz2

Now, copy the `.js` and the `pretiffy.css` to your rails application public folder:

    $ cd google-code-prettify/distrib/google-code-prettify
    $ cp *.js /path/to/my/railsapp/public/javascripts/
    $ cp prettify.css public/stylesheets/

You can use [one of the other styles](http://google-code-prettify.googlecode.com/svn/trunk/styles/index.html) by using the corresponding `.css` file.

Insert `<%= stylesheet_link_tag 'prettify' %>` and `<%= javascript_include_tag 'prettify' %>` within
the `head` element of your `/path/to/my/railsapp/app/views/layouts/application.html.erb` file. Add
the `onLoad="prettyPrint()"` attribute to the `body` element.

Test and enjoy the efficient syntax highlighting!
