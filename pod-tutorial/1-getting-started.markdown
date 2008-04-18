# Part 1 -- Getting Started

In this tutorial we'll run through the creation of the "POD" classified adverts application which was featured in the early Hobo screencasts. At the time of writing those screencasts are very out-dated, but this guide brings things up to date with Hobo 0.7.

For this guide we're going to assume that you're working with a text-editor and command-line terminal.

1. [Setup and Installing Hobo](#install-setup)
2. [Creating A Blank Application](#blank-app)
3. [Adding Models and Controllers](#models-controllers)

## <a name="install-setup">1. Setup and Installing Hobo</a>

### Pre-requisites

The main dependency for Hobo is, of course, Ruby on Rails. Hobo needs at least Rails 2.0, but it's best to get the latest version (2.0.2 at the time of writing). Rails has a few dependencies itself of course, but rather than duplicating the instructions here, just follow the instructions on the main Rails site:

 * [rubyonrails.org: installing Rails](http://www.rubyonrails.org/down)
 
You will also need subversion in order to obtain the needed `will_paginate` plugin:

 * [tigris.org: installing subversion](http://subversion.tigris.org/project_packages.html)
 
(Tip: Try a web search for a better guide to installing subversion on your particular platform)

Hobo also requires a database. Before Rails 2.0.2 the default was MySQL. It just changed to SQLite3. Either of those are a fine for working through this tutorial. If you're on a Mac, SQLite3 is pre-installed since 10.4

 * [mysql.com: download MySQL 5](http://dev.mysql.com/downloads/mysql/5.0.html#downloads)
 * [sqlite.org: download SQLite](http://www.sqlite.org/download.html)


### Installing Hobo

In the process of getting Rails up and running you will install RubyGems, the package-manager for Ruby. That means installing Hobo is as easy as:

    $ gem install hobo
    
That will give you the `hobo` command, which can be used to create a new Hobo application.

---

## <a name="blank-app">2. Creating A Blank Application</a>

### The `hobo` command

Get yourself in a directory where you want your Hobo app to be kept, and:

    $ hobo pod
    
You should see the following: (lots of details removed for brevity)

    Generating Rails app...
          ...
          
    Installing classic_pagination
          ...
          
    Installing Hobo plugin...

    Initialising Hobo...
          ...

    Installing Hobo Rapid and default theme...
          ...

    Creating user model and controller...
          ...

    Creating standard pages...
          ...
          
          
A directory is created called pod, have a look at the files that have been created. This is mostly a standard Rails app, but if you are familiar with the file structure that Rails creates you will notice there are some extra things here and there. In particular you will see that Hobo has been installed in `vendor/plugins`

From here on, all the commands have to be executed in the main directory of your application.

    $ cd pod

### Database setup

If you're using Sqlite, `config/database.yml` is already set up for you and you don't need to create a dababase. If you're using MySQL you now need to modify `config/database.yml` and create the database.

### Migration generator
    
By default, the `hobo` command creates a user model. The database table for this model needs to be created. The migration generator can do this for us:

    $ ruby script/generate hobo_migration
    
You should see:

    ---------- Up Migration ----------
    create_table :users do |t|
      t.string   :crypted_password, :limit => 40
      t.string   :salt, :limit => 40
      t.string   :remember_token
      t.datetime :remember_token_expires_at
      t.string   :username
      t.boolean  :administrator, :default => false
      t.datetime :created_at
      t.datetime :updated_at
    end
    ----------------------------------

    ---------- Down Migration --------
    drop_table :users
    ----------------------------------
    What now: [g]enerate migration, generate and [m]igrate now or [c]ancel?
{: .ruby}
    
Respond with `m` and then give something like "add users" as the filename. The migration file will be created in db/migrate, and the `users` table will be created.

  
## Start the app

Launch Rails' built in development server with:

    $ ruby script/server
    
And point your browser at `http://localhost:3000`. You should see:

![Front Page](/images/tutorial/front-page.png)

## Sign-up and Log-in

You should find that you are already able to sign up as a new user, and log in and out.

We now have a basic Hobo app up and running with the default theme. Of course the app doesn't do much at this stage. To add functionality, the first step is to create some models.

---

## <a name="models-controllers">3. Adding Models and Controllers</a>

### Create the models

The POD demo has a simple data-model. Let's start with a look at three models and the relationships between them. There are many users, (Hobo has already created the user model for us), a user has many adverts, and conversely each advert belongs to a user. There are also categories, such as Computers, Musical Instruments, etc. Each advert belongs in one category, and conversely, each category contains many adverts.

In Rails terms, we would write:

    class User < ActiveRecord::Base
      has_many :adverts
    end

    class Advert < ActiveRecord::Base
      belongs_to :user
      belongs_to :category
    end

    class Category < ActiveRecord::Base
      has_many :adverts
    end
{: .ruby}

The models need some data in them. Users won't need any extra fields beyond what hobo gives us automatically, but adverts will need a title and a body, and categories will need a name. With Hobo we declare the fields inside the models. We would code this as follows (don't do anything yet -- this is just to give you an overview of the data-model):

    class Advert < ActiveRecord::Base
      fields do
        title :string
        body  :text
      end
      belongs_to :user
      belongs_to :category
    end

    class Category < ActiveRecord::Base
      fields do
        name :string
      end
      has_many :adverts
    end
{: .ruby}

OK let's go ahead and create these models. As mentioned, `User` is already done. Let's generate `Advert` and `Category`

(Tip: keep the development server running in its terminal, and open up a new terminal in which to run these commands. That way you can just refresh the browser when you want to see the changes to the web-app)

    $ ruby script/generate hobo_model advert
    $ ruby script/generate hobo_model category

We need to edit the files that have been created in `app/models`. Let's start with `advert.rb`. Open it up in your editor, it should look like:

    class Advert < ActiveRecord::Base

      hobo_model

      fields do
        timestamps
      end


      # --- Hobo Permissions --- #
      # Ignore everything below here for now
    end
{: .ruby}
    
There are a few differences from the skeleton file we described above. Firstly, there are four stub methods for Hobo's permission system -- we'll ignore those for now. There's the `hobo_model` declaration, which all Hobo-enhanced models need, and there's the word `timestamps` in the `fields` block. `timestamps` tells Hobo to add the standard `created_at` and `updated_at` fields to this model (Active Record will maintain these fields automatically).

We need to add the two associations:

    belongs_to :user
    belongs_to :category
{: .ruby}

And the two fields:
    
    fields do
      title :string
      body  :text
      timestamps
    end
{: .ruby}
    
The file should end up looking like this:
    
    class Advert < ActiveRecord::Base

      hobo_model

      fields do
        title :string
        body  :text
        timestamps
      end

      belongs_to :user
      belongs_to :category

      # --- Hobo Permissions --- #
      # Don't change anything below here yet
    end
{: .ruby}

Similarly, edit the `app/models/category.rb` to look like this:

    class Category < ActiveRecord::Base

      hobo_model

      fields do
        name :string
        timestamps
      end

      has_many :adverts


      # --- Hobo Permissions --- #
      # Don't change anything below here yet
    end
{: .ruby}
    
Finally, we need to add just one thing to `app/models/user.rb` -- the `has_many :adverts` declaration. The file should end up looking like this:

    class User < ActiveRecord::Base

      hobo_user_model

      fields do
        username :string, :login => true, :name => true
        administrator :boolean
        timestamps
      end
  
      has_many :adverts

      set_admin_on_first_user

      # --- Hobo Permissions --- #
      # Don't change anything below here yet
    end
{: .ruby}
    
With those changes in place, we're ready to create the database tables:

    $ ruby script/generate hobo_migration
    
That should output:

    --------- Up Migration ----------
    create_table :adverts do |t|
      t.string   :title
      t.text     :body
      t.datetime :created_at
      t.datetime :updated_at
      t.integer  :user_id
      t.integer  :category_id
    end

    create_table :categories do |t|
      t.string   :name
      t.datetime :created_at
      t.datetime :updated_at
    end
    ----------------------------------

    ---------- Down Migration --------
    drop_table :adverts
    drop_table :categories
    ----------------------------------
    What now: [g]enerate migration, generate and [m]igrate now or [c]ancel?
{: .ruby}
    
Notice how the generator knows to create the two foreign keys on the adverts table according to Active Record conventions. Respond with `m` and give something like "add adverts and categories" as the migration name.

That's the basics of the model layer in place. You won't see any changes in the app yet though, because there are no controllers for our new models.

### Create the controllers

We'll look at controllers in a bit more detail later. For now we just need some controllers in place so we can play with the app. We can generate them like this:

    $ ruby script/generate hobo_model_controller advert
    $ ruby script/generate hobo_model_controller category
    
Note that you provide the *singular* model name to the generator.

Refresh the browser and you should see:

<img src="/images/tutorial/front-with-models.png">

Next: [Part 2. Customizing Models and Controllers](/pod-tutorial/2-customizing-models-and-controllers)