&# The DRYML Guide
{.document-title}

Welcome to the DRYML Guide. If you want to learn all the ins and outs of DRYML and become a master of quick and elegant view templates, you're in the right place. If you're very new to Hobo and DRYML you might not be better off with something like the Agility tutorial. This guide is somewhere between a tutorial and a reference, designed to fill in the gaps for people who have already got the hang of the basics. Right, lets get started!


# What is DRYML?

DRYML is a template language for Ruby on Rails that you can use in place of Rails' built in ERB templates. It is part of the larger Hobo project, but will eventually be made available as a separate plugin. DRYML was created in response to the observation that the vast majority of Rails development time seems to be spent in the view-layer. Rails' models are beautifully declarative, the controllers can be made so pretty easily (witness the many and various "result controller" plugins), but the views, ah the views...

Given that so much of the user-interaction we encounter on the web is so similar from one website to another, surely we don't have to code all this stuff up from low-level primitives over and over again? Please, no! Of course what we want is a nice library of ready-to-go user interface components, or widgets, which can be quickly added to our project, and easily tailored to the specifics of our application.

If you've been at this game for a while you're probably frowning about now. Re-use is a very, very thorny problem. It's one of those things that sounds straight-forward and obvious in principle, but turns out to be horribly difficult in practice. When you come to re-use something, you very often find that your new needs differ from the original ones in a way that wasn't foreseen or catered for in the design of the component. The more complex the component, the more likely it is that bending the thing to your needs will be harder than starting again from scratch. 

So the challenge is not in being able to re-use code, it is in being able to re-use code in ways that were not foreseen. The reason we created DRYML was to see if this kind of flexibility could be built into the language itself. DRYML is a tag based language that makes it trivially easy to give the defined tags a great deal of flexibility.

So DRYML is just a means to an end. The real goal is to create a library of reusable user-interface components that actually succeed in making it very quick and easy to create the view-layer of a web application. That library is also part of Hobo -- the *Rapid* tag library, but Rapid is not covered in this guide. Here we will see how DRYML provides the tools and raw materials that make a library like Rapid possible.


# Simple page templates and ERB

In it's most basic usage, DRYML can be indistinguishable from a normal Rails template. That's because DRYML is (almost) an extension of ERB, so you can still use Ruby snippets using the `<% ... %>` notation. For example, a show-page for a blog post might look like this:

    <html>
      <head>
        <title>My Blog</title>
      </head>
      <body>
        <h1>My Famous Blog!</h1>
        <h2><%= @post.title %></h2>
        
        <div class="post-body">
          <%= @post.body %>
        </div>
      </body>
    </html>
{.dryml}

## No ERB inside tags

DRYML's support for ERB is not *quite* the same as true ERB templates. The one thing you can't do is use ERB snippets inside a tag. To have the value of an attribute generated dynamically in ERB, you could do:

    <a href="<%= my_url %>">
    
In DRYML you would do:

    <a href="#{my_url}">
    
In rare cases, you might use an ERB snippet to output one or more entire attributes:

    <form <%= my_attributes %>>
    
To do the equivalent in DRYML, you would need your attributes to be in a hash (rather than a string), and do:

    <form merge-attrs="&my_attributes">
    
Finally, in a rare case you could even use an ERB snippet to generate the tag name itself:

    <<%= my_tag_name>> ... </<%= my_tag_name %>>
    
To achieve that in DRYML, you would use the special tag `call-tag`

    <call-tag tag="&my_tag_name"> ... </call-tag>
    
## Wot no layouts?

Going back to the `<page>` tag at the start of this section, from a "normal Rails" perspective, you might be wondering why the boilerplate stuff like `<html>`, `<head>` and `<body>` are there. What happened to layouts? You don't tend to use layouts with DRYML, instead you would define your own tag, typically `<page>`, and call that. Using tags for layouts is much more flexible, and it moves the choice of layout out of the controller and into the view-layer, where it should be.
    
We'll see how to define a `<page>` tag in the next section.


# Defining simple tags

One of the strengths of DRYML is that defining tags is done right in the template (or in an imported tag-library) using the same XML-like syntax. This means that if you've got mark-up you want to re-use, you can simply cut-and-paste it into a tag definition.

Here's the page from the previous section, defined as a `<page>` tag simply by wrapping the mark-up in a `<def>` tag:

    <def tag="page">
      <html>
        <head>
          <title>My Blog</title>
        </head>
        <body>
          <h1>My Famous Blog!</h1>
          <h2><%= @post.title %></h2>
      
          <div class="post-body">
            <%= @post.body %>
          </div>
        </body>
      </html>
    </def>
{.dryml}


Now we can call that tag just as we would call any other:

    <page/>
{.dryml}

If you'd like an analogy to "normal" programming, you can think of the `<def>...</def>` as defining a method called `page`, and `<page/>` as a call to that method. In fact, DRYML is implemented by compiling to Ruby, and that is exactly what is happening.

## Parameters

We've illustrated the most basic usage of `<def>`, but out `<page>` tag is not very useful. Let's take it a step further to make it into the equivalent of a layout. First of all, we clearly need the body of the page to be different each time we call it. In DRYML we achieve this by adding *parameters* to the definition, which is accomplished with the `param` attribute. Here's the new definition:

    <def tag="page">
      <html>
        <head>
          <title>My Blog</title>
        </head>
        <body param/>
      </html>
    </def>
{.dryml}


Now we can call that tag providing our own body:

    <page>
      <body:>
        <h1>My Famous Blog!</h1>
        <h2><%= @post.title %></h2>
    
        <div class="post-body">
          <%= @post.body %>
        </div>
      </body:>
    </page>
{.dryml}

See how easy that was? We just added `param` to the `<body>` tag, which means our page tag now has a parameter called `body`. In the call we provide some content for that parameter. It's very important to read that call to `<page>` properly. In particular, the `<body:>` (note the trailing ':') is *not* a call to a tag, it is providing a named parameter to the call to `<page>`. We call `<body:>` a *parameter tag*. In Ruby terms you could think of the call like this:
        
    page(:body => "...my body content...")
    
Note that is not actually what the compiled Ruby looks like in this case, but it illustrates the important point that `<page>` is a call to a defined tag, whereas `<body:>` is providing a parameter to that call.
    
## Changing Parameter Names
    
To give the parameter a different name, we can provide a value to the `param` attribute:

    <def tag="page">
      <html>
        <head>
          <title>My Blog</title>
        </head>
        <body param="content"/>
      </html>
    </def>
{.dryml}

We would now call the tag like this:

    <page><content:> ...body content goes here... </content:></page>
    
## Multiple Parameters
    
As you would expect, we can define many parameters in a single tag. For example, here's a page with a side-bar:

    <def tag="page">
      <html>
        <head>
          <title>My Blog</title>
        </head>
        <body>
          <div param="content"/>
          <div param="aside" />
        </body>
      </html>
    </def>
{.dryml}

Which we could call like this:

    <page>
      <content:> ... main content here ... </content:>
      <aside:>  ... aside content here ... </aside:>
    </page>
{.dryml}

Note that when you name a parameter, DRYML automatically adds css classes of the same name to the output, so the two `<div>` tags above will be output as `<div class="content">` and `<div class="aside">` respectively.
    
## Default Parameter Content

In the examples we've seen so far, we've only put the `param` attribute on empty tags. That's not required though. If you declare a non-empty tag as a parameter, the content of that tag becomes the default when the call does not provide that parameter. This means you can easily add a parameter to any part of the template that you think the caller might want to be able to change

    <def tag="page">
      <html>
        <head>
          <title param>My Blog</title>
        </head>
        <body param>
      </html>
    </def>
{.dryml}

We've made the page title parameterised. All existing calls to `<page/>` will continue to work unchanged, but we've now got the ability to change the title on a per-page basis:

    <page>
      <title:>My VERY EXCITING Blog</title:>
      <body:>
        ... body content
      </body:>
    </page>
{.dryml}

This is a very nice feature of DRYML - whenever you're writing a tag, and you see a part that might be useful to change in some situations, just throw the `param` attribute at it and you're done.

## Nested `param` Declarations

You can nest `param` declarations inside other tags that have `param` on them. For example, there's no need to chose between a `<page>` tag that provides a single content section and one that provides an aside section as well -- a single definition can serve both purposes:

    <def tag="page">
      <html>
        <head>
          <title>My Blog</title>
        </head>
        <body param>
          <div param="content"/>
          <div param="aside" />
        </body>
      </html>
    </def>
{.dryml}

Here the `<body>` tag is a `param`, and so are the two `<div>` tags inside it. It can be called either like this:
    
    <page>
      <body:> ... page content goes here ... </body:>
    </page>
{.dryml}    

Or like this:

    <page>
      <content:> ... main content here ... </content:>
      <aside:>  ... aside content here ... </aside:>
    </page>
{.dryml}

An interesting question is, what happens if you give both a `<body:>` parameter and say, `<content:>`. By providing the `<body:>` parameter, you have replaced everything inside the body section, including those two parameterised `<div>` tags, so the `<body:>` you have provided will appear as normal, but the `<content:>` parameter will be silently ignored.

    
## The Default Parameter

In the situation where you only want a single parameter, you can give your tag a more compact XML-like syntax by using the special parameter name `default`:

    <def tag="page">
      <html>
        <head>
          <title>My Blog</title>
        </head>
        <body param="default"/>
      </html>
    </def>
{.dryml}

Now there is no need to give a parameter tag in the call at all - the content directly inside the `<page>` tag becomes the `default` parameter:
    
    <page> ... body content goes here -- no need for a parameter tag ... </page>
    
You might notice that the `<page>` tag is now indistinguishable from a normal HTML tag. Some find this aspect of DRYML disconcerting at first -- how can you tell what is an HTML tag and what it a defined DRYML tag? The answer is -- you can't, and that's quite deliberate. This allows you to do nice tricks like define your own smart `<form>` tag or `<a>` tag (the Rapid library does exactly that). Other tag-based template languages (e.g. Java's JSP) like to put everything in XML namespaces. The result is very cluttered views that are boring to type and hard to read. From the start we put a very high priority on making DRYML templates compact and elegant. When you're new to DRYML you might have to do a lot of looking things up, as you would with any new language or API, but things gradually become familiar and then view templates can be read and understood very easily.
{.aside}    


## The Implicit Context

In addition to the most important goal behind DRYML - creating a template language that would encourage re-use in the view-layer, a secondary goal is for templates to be concise, elegant and readable. One aspect of DRYML that helps a lot in this regard is something called the *implicit context*.

This feature was borne of a simple observation that pretty much every page in a web-app renders some kind of hierarchy of application objects. Think about a simple page in a blog - say, the permalink page for an individual post. The page as a whole can be considered a rendering of a BlogPost object. Then we have sections of the page that display different "pieces" of the the post -- the title, the date, the author's name, the body. Then we have the comments. The list of comments as a whole is also a "piece" of the BlogPost. Within that we have each of the individual comments, and the whole thing starts again: the comment title, date, author... This can carry on even further, for example some blogs are set-up so that you can comment on comments.

This structure is incredibly common, perhaps even universal, as it seems to be intrinsically tied to the way we visually parse information. DRYML's implicit context takes advantage of this fact to make templates extremely concise while remaining readable and clear. The object that you are rendering in any part of the page is known as the *context*, and every tag has access to this object through the method `this`. The controller sets up the initial context, and the templates then only have to mention where the context needs to *change*.

We'll dive straight into some examples, but first a quick general point about this guide. If you like to use the full Hobo framework, you will probably always use DRYML and the Rapid tag library together. DRYML and Rapid have grown up together, and the design of each is heavily influenced by the other. Having said that, this is the DRYML Guide, not the Rapid Guide. We won't be using any Rapid tags in this guide, because we want to document DRYML the language properly. That will possibly be a source of confusion if you're very used to working with Rapid. Just keep in mind that we're not allowed to use any Rapid tags in this guide and you'll be fine.

In order to see the implicit context in its best light, we'll start by defining a `<view>` tag, that simply renders the current context with HTML escaping. Remember the context is always available as `this`:
    
    <def tag="view"><%= h this.to_s %></def>
{.dryml}

And here's a tag for making a link to the current context. We'll assume the object will be recognised by Rails' polymorphic routing. Let's call it `<l>` (for link).

    <def tag="l"><a href="#{url_for this}" param="default"/></def>
    
Note that by defining that `<a>` tag, normal HTML `<a>` tags won't work anymore. We'll see how to fix that in a later section. Now let's use these tags in a page template. We'll stick with the comfortingly boring blog post example. In order to set the initial context, our controller action would need to do something like this:

    def show
      @this = @blog_post = BlogPost.find(params[:id])
    end
{.ruby}

The DRYML template handler looks for the `@this` instance variable for the initial context. It's quite nice to also set the more conventionally named instance variable as we've done here. Now we'll create the page, let's assume we're using a `<page>` tag along the lines of those defined above. We'll also assume that the blog post object has these fields: `title`, `published_at`, `body` and `belongs_to :author`, and that the author has a `name` field.
    
    <page>
      <content:>
        <h2><view:title/></h2>
        <div class="details">
          Published by <l:author><view:name/></l> on <view:published-at/>.
        </div>
        <div class="post-body">
          <view:body/>
        </div>
      </content:>
    </page>
{.dryml}

When you see a tag like `<view:title/>`, you don't get any prizes for guessing what will be displayed. In terms of what actually happens, you can read this as "change the context to be the `title` attribute of the current context, then call the `<view`> tag". You might like to think of that change to the context as `this = this.title` (although in fact `this` is not assignable). But really you just think of it as "view the title". Of what? Of whatever is in context, in this case the blog post.

Be careful with the two different uses of colon in DRYML. A trailing colon as in `<foo:>` indicates a parameter tag, whereas a colon joining two names as in `<view:title/>` indicates a change of context.

When the tag ends, the context is set back to what it was. In the case of `<view/>` which is a self-closing tag familiar from XML, that happens immediately. The `<l>` tag is more interesting. We set the context to be the author, so that the link goes to the right place. Inside the `<l>` that context remains in place so we just need `<view:name/>` in order to display the author's name.

## `with` and `field` attributes

The `with` attribute is a special DRYML attribute that sets the context to be the result of any Ruby expression before the tag is called. In DRYML any attribute value that starts with '&' is interpreted as a Ruby expression. Here's the same example as above using only the with attribute:

    <page>
      <content:>
        <h2><view with="@blog_post.title"/></h2>
        <div class="details">
          Published by <l with="&@blog_post.author"><view with="&this.name"/></l>
          on <view with="&@blog_post.published-at"/>.
        </div>
        <div class="post-body">
          <view with="&@blog_post.body"/>
        </div>
      </content:>
    </page>
{.dryml}

Note that we could have used `&this.title` instead of `&@blog_post.title`.

The `field` attribute makes things more compact by taking advantage of a common pattern. When changing the context, we very often want to change to some attribute of the current context. `field="x"` is a shorthand for `with="&this.x"` (actually it's not quite the same, using the `field` version also sets `this_parent` and `this_field`, whereas `with` does not. This is discussed later in more detail).

The same template again, this time using `field`:

    <page>
      <content:>
        <h2><view field="title"/></h2>
        <div class="details">
          Published by <l field="author"><view field="name"/></l> on <view field="published-at"/>.
        </div>
        <div class="post-body">
          <view field="body"/>
        </div>
      </content:>
    </page>
{.dryml}

If you compare that example to the first one, you should notice that the `:` syntax is just a shorthand for the `field` attribute. i.e. `<view field="name">` and `<view:name>` are equivalent.

 - merging attributes

 - Repeated and optional content
 
 - Attributes
 
 - Parameters
 
 - before/after/append/prepend and without
  
 - nested parameters
 
 - extending tags and merging params/attributes
 
 - aliasing tags
 
 - polymorphic tags
 
 - wrapping - restore and param-content
 
 - special local variables: `attributes`, `all_attributes`, `parameters` and `all_parameters`
 
 - Variables - set and set-scoped
 
 - Taglibs
 
 - Special Methods: `this_field`, `this_parent` (OTHERS?)
 
 - Current limitations and other gotchas
 
 