# Getting Started

*Hobo is beta software - version 0.7.3 Although Hobo is in active use in production applications, please be aware that if you choose to deploy a Hobo/Rails application on the Internet you do so at your own risk.*

*Also note that at this stage we reserve the right to make breaking changes to the API.*

## Getting Hobo

Hobo is distributed under the terms of the [MIT license](http://www.opensource.org/licenses/mit-license.php).

*PLEASE NOTE* Hobo currently requires Rails 2.0

Hobo is distributed in two forms, a gem and a plugin (svn repo).

### Hobo Gem

The gem is ideal for trying out Hobo with a new app. It gives you a single command:

    $ hobo <app-name>

It works just like the `rails` command, creating a blank Rails application pre-configured for Hobo. 

To install, simply

    $ gem install hobo

### Hobo plugin

If you want to add Hobo to an existing application, first do:

    $ ./script/plugin install svn://hobocentral.net/hobo/trunk
    $ ./script/generate hobo --add-routes

(the flag tells tells the generator to modify your config/routes.rb)

Then there are a few optional steps, depending on which Hobo features you're after. In the screencast you've seen:

#### Hobo Rapid and the default theme:

	$ ./script/generate hobo_rapid --import-tags

(the flag tells the generator to add some necessary tags to your application.dryml)
	
#### Hobo's user model:

	$ ./script/generate hobo_user_model user
	
#### The automatic front page, signup/login and search:

    $ ./script/generate hobo_front_controller front --add-routes --delete-index
	
(the flags tell the generator to add some new routes to config/routes.rb and to delete public/index.html so the front page will work)