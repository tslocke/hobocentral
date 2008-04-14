# HOWTO
## Create an admin sub-site in Hobo

A sub-site is part of a web-app that has the appearance and behavior of a separate site. The most
common example of a sub-site is an "admin" section. This guide shows how to create a typical
admin sub-site in Hobo.

At the time of writing, Hobo's generators are unable to generate sub-site controllers so it is 
necessary to create the files manually.

Create a new directory `app/controllers/admin/`

Create a base controller for the sub-site in `app/controllers/admin/admin_controller.rb`:

    class Admin::AdminController < ApplicationController
  
      hobo_controller

      # a taglib to be included by every controller in the sub-site
      include_taglib 'admin'

      # require administrator to access any controller in the sub-site
      before_filter :admin_required
      def admin_required
        logged_in? && current_user.administrator?
      end

      # the default page when visiting the sub-site
      def index
        redirect_to '/admin/users'
      end
    end
{: .ruby}

Now create a model controller in the admin sub-site. As an example we will create a controller to manage users. The following goes in `app/controllers/admin/users_controller.rb`:

    class Admin::UsersController < Admin::AdminController

      hobo_model_controller User
  
      auto_actions :index, :edit, :destroy, :update
  
    end
{: .ruby}
  
Notice two things that are slightly different to normal:

* The controller inherits from `Admin::AdminController` instead of `ApplicationController`
* You have to explicitly specify the name of the model as an argument to `hobo_model_controller`. In a non-sub-site controller Hobo can usually work this out automatically so it isn't needed.

Add a route for the home page of the sub-site in `config/routes.rb`:

    map.admin  'admin', :controller => 'admin/admin', :action => 'index'
{: .ruby}

In the base controller we wrote `include_taglib 'admin'`. This means that the tags
in `app/views/taglibs/admin.dryml` will be loaded for every action in the sub-site.
This is a convenient place to re-define `<page>`, set a theme and define any tags that
are specific to the sub-site. As an example, create `app/views/taglibs/admin.dryml`
and add the following:

    <def tag="page" extend-with="admin" attrs="layout">
      <% layout ||= 'aside' %>
      <page-without-admin layout="#{layout}" merge>
        <scripts: param>
          <param-content/>
          <javascript name="admin"/>
        </scripts:>
        <stylesheets: param>
          <param-content/>
          <stylesheet name="admin"/>
        </stylesheets:>
        <main-nav: replace>
          <navigation class="main-nav">
            <nav-item with="&User">Users</nav-item>
          </navigation>
        </main-nav:>    
      </page-without-admin>
    </def>
{: .dryml}

Here we've defined an admin specific version of `<page>` that uses an aside layout by
default. We've also added admin specific javascript and stylesheet files which you will
need to create in `public/javascripts/admin.js` and `public/stylesheets/admin.css`. Finally
we have defined our admin specific nav bar containing a link to our users model controller
because Hobo excludes users from the nav bar by default.

We'll want to tweak Rapid's generic pages to be a bit more admin oriented. By adding the following
to `admin.dryml` we can replace the normal list on an index page with an admin style table:

    <def tag="index-page" extend-with="admin">
      <index-page-without-admin merge>
        <collection: replace>
          <table-plus fields="username, administrator" param/>
        </collection:>
      </index-page-without-admin>
    </def>
{: .dryml}

If we add additional model controllers to the admin sub-site they will all use this modified
version of `<index-page>`. Similar admin specific tweaks can be made to the other generic pages
and tags.
