---
layout: post
title: Installing Bootstrap 4 on Rails 6
excerpt: This is a meta description
permalink: installing-bootstrap-4-on-rails-6
tags: [ruby on rails, bootstrap]
image: /test2.png
---

<figure>
<img src="{{ page.image }}" alt="Installing Bootstrap 4 on Rails 6">
</figure>

[Bootstrap](https://getbootstrap.com/){:target="_blank"} is a front-end web framework that makes creating responsive web layouts and interactive front-end components far easier than it was years ago.  While it’s not the only front-end, responsive, mobile-first framework these days, it is arguably the most popular.  It also pairs well with [Ruby on Rails](https://rubyonrails.org/){:target="_blank"}.  In this post, we’ll explore how to install Bootstrap 4 on Rails 6.

From Rails 3.1 to Rails 5, the Asset Pipeline was where Rails stored JavaScript, stylesheets and image assets.  Rails 6 now uses [webpack](https://webpack.js.org/){:target="_blank"} rather than the Asset Pipeline for JavaScript.  In Rails 6, all JavaScript now resides in its own directory under _app/javascript_ rather than under _app/assets/javascript_ as it did in Rails 5.

Because Bootstrap 4 depends on jQuery for its various interactive elements (tooltips, popups, modals), we need a way to install jQuery and have it compile through webpack.  In Rails 5, this was usually done using Ruby gems for Bootstrap and jQuery.

In Rails 6, we use [yarn](https://classic.yarnpkg.com/en/){:target="_blank"} to install and import the node modules and then use webpack to run the modules.  A new Rails 6 app will have webpack installed by default through the [webpacker gem](https://github.com/rails/webpacker){:target="_blank"} when a new Rails app is created.

In Rails 6, not only does the JavaScript go through Webpack.  All CSS does as well.  Technically, all CSS still belongs in the _app/assets/stylesheets_ directory but for this post we will keep things simple and put the CSS in the _app/javascript_.

First, let’s create a new Rails app called bootstrapper:

{% highlight terminal %}
$ rails new bootstrapper
{% endhighlight %}

Now let’s **cd** into the new bootstrapper directory and install Bootstrap and its dependencies jQuery and popper.js which is used for tooltips, popovers, dropdowns and modals:

{% highlight terminal %}
$ cd bootstrapper
$ yarn add bootstrap jquery popper.js
{% endhighlight %}

Next, to setup Bootstrap the first thing we want to do is to go into the webpack directory’s environment.js file located at _config/webpack/environment.js_ and add the new **Provide** plugin for webpack.  Note that the top and bottom lines are already in the file.  We just need to add the code block in the middle:

{% highlight js %}
const { environment } = require('@rails/webpacker')

const webpack = require('webpack')
environment.plugins.append('Provide',
  new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    Popper: ['popper.js', 'default']
  })
)

module.exports = environment
{% endhighlight %}

Next we go into _app/javascript/packs/application.js_ and under the **require** statements we import Bootstrap and also **addEventListener** for tooltips and popovers in Bootstrap:

{% highlight js %}
import "bootstrap";

document.addEventListener("turbolinks:load", () => {
  $('[data-toggle="tooltip"]').tooltip()
  $('[data-toggle="popover"]').popover()
})
{% endhighlight %}

As a next step, we need to create a directory for our stylesheets.  Unlike in Rails 5 where the stylesheets were normally stored in the _app/assets/stylesheets_, this file actually goes in a directory we create under _app/javascript/stylesheets_ directory.  We can do this manually but here we’ll do it it at the command line.  In our Mac OS X (Unix) or Linux terminal, let’s **cd** into the _app/javascript_ directory, make a directory called stylesheets and in that directory create a Sass file called _application.scss_:

{% highlight terminal %}
$ cd app/javascript/
$ mkdir stylesheets
$ cd stylesheets
$ touch application.scss
{% endhighlight %}

Then, inside the newly created _application.scss_ file, add the following import statement:

{% highlight sass %}
@import "~bootstrap/scss/bootstrap";
{% endhighlight %}

Back in our _app/javascript/packs/application.js_ file, we need to import the stylesheet:

{% highlight js %}
require("@rails/ujs").start()
require("turbolinks").start()
require("@rails/activestorage").start()
require("channels")

import "bootstrap";
import "../stylesheets/application"  // <- Add this line

document.addEventListener("turbolinks:load", () => {
  $('[data-toggle="tooltip"]').tooltip()
  $('[data-toggle="popover"]').popover()
})
{% endhighlight %}

Next, in our layout file _app/views/layouts/application.html.erb_ we need to add an additional line for the stylesheet link to include the **stylesheet_pack_tag**.  This is going to export the JavaScript and CSS in a single JS file (which is a bit different than the standard way) but it’s for the Webpack dev server:

{% highlight erb %}
<!DOCTYPE html>
<html>
  <head>
    <title>Bootstrapper</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
{% endhighlight %}

## Creating Our Test Controller, Views & Routes

Let’s test our Bootstrap installation by creating a controller called post with an index action which will also create our post index view:

{% highlight terminal %}
$ rails g controller post index
{% endhighlight %}

The Rails controller generate statement above also created a route for us in the _config/routes.rb_ file so let’s edit that file to make the post index view our root:

{% highlight ruby %}
Rails.application.routes.draw do
  root to: 'post#index'
end
{% endhighlight %}

Now we are ready to start adding Bootstrap components.  Let’s add the standard Bootstrap navbar by grabbing the example [navbar component code](https://getbootstrap.com/docs/4.4/components/navbar/){:target="_blank"} from the Bootstrap website and pasting it into our layout file _app/views/layouts/application.html.erb_.

Underneath the navbar code we can also add the example [tooltip component code](https://getbootstrap.com/docs/4.4/components/tooltips/){:target="_blank"} to test that functionality as well:

{% highlight erb %}
<!DOCTYPE html>
<html>
  <head>
    <title>Bootstrapper</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>

    <!-- Begin Bootstrap Navbar -->

    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">Navbar</a>
      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto">
          <li class="nav-item active">
            <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="#">Link</a>
          </li>
          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          Dropdown
            </a>
            <div class="dropdown-menu" aria-labelledby="navbarDropdown">
              <a class="dropdown-item" href="#">Action</a>
              <a class="dropdown-item" href="#">Another action</a>
              <div class="dropdown-divider"></div>
              <a class="dropdown-item" href="#">Something else here</a>
            </div>
          </li>
          <li class="nav-item">
            <a class="nav-link disabled" href="#" tabindex="-1" aria-disabled="true">Disabled</a>
          </li>
        </ul>
        <form class="form-inline my-2 my-lg-0">
              <input class="form-control mr-sm-2" type="search" placeholder="Search" aria-label="Search">
          <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
        </form>
      </div>
    </nav>

    <!-- End Bootstrap Navbar -->

    <!-- Begin Bootstrap Tooltips -->

    <button type="button" class="btn btn-secondary" data-toggle="tooltip" data-placement="top" title="Tooltip on top">
      Tooltip on top
    </button>
    <button type="button" class="btn btn-secondary" data-toggle="tooltip" data-placement="right" title="Tooltip on right">
      Tooltip on right
    </button>
    <button type="button" class="btn btn-secondary" data-toggle="tooltip" data-placement="bottom" title="Tooltip on bottom">
      Tooltip on bottom
    </button>
    <button type="button" class="btn btn-secondary" data-toggle="tooltip" data-placement="left" title="Tooltip on left">
      Tooltip on left
    </button>

     <!-- End Bootstrap Tooltips -->

    <%= yield %>
  </body>
</html>
{% endhighlight %}

Now, all we need to do is start our local dev server by typing in **rails s** at the command line from inside our application directory, open our browser and navigate to <http://localhost:3000/>.

As we can see in the screenshot below, when our page loads both our navbar and tooltips are rendering in the main Bootstrap style which means we have successfully installed Bootstrap 4 in Rails 6:

![bootstrap installed on rails](/rails_bootstrap_installed.png)
