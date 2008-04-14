# Generators

## Hobo

#### Command

    script/generate hobo [--add-routes]
  
Installs a blank application.dryml in app/views/hobolib and a guest model in 
app/models.

---

## Hobo Rapid

#### Command 

    script/generate hobo_rapid [ --import-tags ] 

Install the Hobo Rapid JavaScript library and Lowpro in public/javascripts and the default theme in 
app/views/hobolib/themes and public/hobothemes.

Option          | Description                                                             |
----------------|-------------------------------------------------------------------------|
`--import-tags` | Modify application.dryml to import the Hobo Rapid and<br />default theme tags|
{: .command-options}

---

## Hobo Model

#### Command 

    script/generate hobo_model <model-name> 

Create a regular Rails model in app/models, with the extra declaration `hobo_model` (which 
is just a shorthand for include `Hobo::Model`) and stubs for the four Hobo permission 
methods. 

---

## Hobo Model Controller

#### Command 

    script/generate hobo_model_controller <model-name> 
    
Pluralises the model name and generates a controller of that name. The controller has the 
declaration `hobo_model_controller` added (a shorthand for include 
Hobo::ModelController) 

---

## Hobo Model Resource

#### Command 

    script/generate hobo_model_resource <model-name> 

Generates a `hobo_model` and a `hobo_model_controller` for the given model name. 
The same as running the `hobo_model` generator followed by the `hobo_model_controller` generator.

---

## Hobo User Model

#### Command

    script/generate hobo_user_model <model-name> 
    
Generates a regular Rails model in app/models with the following declarations: 


#### Model class declarations 

    hobo_user_model

    fields do
      username :string, :login => true, :name => true
      administrator :boolean, :default => false
      timestamps
    end

    set_admin_on_first_user

Also creates stubs for the permissions methods.

---

## Hobo Front Controller

#### Command 

    script/generate hobo_front_controller <controller-name> 
                        [ --add-routes ] [ --delete-index ]
                                          
Generate a controller and views to handle the applications front page, a search page, a login 
page and a sign-up page. 

Option           | Description                                                             |
-----------------|-------------------------------------------------------------------------|
`--add-routes`   | Modify config/routes.rb to include routes for these pages               |
`--delete-index` | Delete public/index.html so that the front page works                   |
{: .command-options}