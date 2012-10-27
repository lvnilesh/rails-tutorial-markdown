# Chapter 2 A demo app

In this chapter, we'll develop a simple demonstration application to show off
some of the power of Rails. The purpose is to get a high-level overview of
Ruby on Rails programming (and web development in general) by rapidly
generating an application using _scaffold generators_. As discussed in
Box1.1, the rest of
the book will take the opposite approach, developing a full application
incrementally and explaining each new concept as it arises, but for a quick
overview (and some instant gratification) there is no substitute for
scaffolding. The resulting demo app will allow us to interact with it through
its URIs, giving us insight into the structure of a Rails application,
including a first example of the _REST architecture_ favored by Rails.

As with the forthcoming sample application, the demo app will consist of
_users_ and their associated _microposts_ (thus constituting a minimalist
Twitter-style app). The functionality will be utterly under-developed, and
many of the steps will seem like magic, but worry not: the full sample app
will develop a similar application from the ground up starting in
Chapter3, and I will provide
plentiful forward-references to later material. In the mean time, have
patience and a little faith--the whole point of this tutorial is to take you
_beyond_ this superficial, scaffold-driven approach to achieve a deeper
understanding of Rails.

## [2.1 Planning the application](a-demo-app.html#sec-
planning_the_application)

In this section, we'll outline our plans for the demo application. As in
Section1.2.3,
we'll start by generating the application skeleton using the `rails` command:

    
    $ cd ~/rails_projects
    $ rails new demo_app
    $ cd demo_app
    

Next, we'll use a text editor to update the `Gemfile` needed by Bundler with
the contents of [Listing2.1](a-demo-app.html#code-
demo_gemfile_sqlite_version_redux).

Listing 2.1. A `Gemfile` for the demo app.

    
    source 'https://rubygems.org'
    
    gem 'rails', '3.2.8'
    
    group :development do
      gem 'sqlite3', '1.3.5'
    end
    
    
    # Gems used only for assets and not required
    # in production environments by default.
    group :assets do
      gem 'sass-rails',   '3.2.5'
      gem 'coffee-rails', '3.2.2'
    
      gem 'uglifier', '1.2.3'
    end
    
    gem 'jquery-rails', '2.0.2'
    
    group :production do
      gem 'pg', '0.12.2'
    end
    

Note that [Listing2.1](a-demo-app.html#code-
demo_gemfile_sqlite_version_redux) is identical to
Listing1.9.

As in Section1.4.1,
we'll install the local gems while suppressing the installation of production
gems using the `--without production` option:

    
    $ bundle install --without production
    

Finally, we'll put the demo app under version control. Recall that the `rails`
command generates a default `.gitignore` file, but depending on your system
you may find the augmented file from
Listing1.7 to be more
convenient. Then initialize a Git repository and make the first commit:

    
    $ git init
    $ git add .
    $ git commit -m "Initial commit"
    

!create_demo_repo_new

Figure 2.1: Creating a demo app repository at GitHub.[(full
size)](http://railstutorial.org/images/figures/create_demo_repo_new-full.png)

You can also optionally create a new repository
(Figure2.1 and
push it up to GitHub:

    
    $ git remote add origin git@github.com:<username>/demo_app.git
    $ git push -u origin master
    

(As with the first app, take care _not_ to initialize the GitHub repository
with a `README` file.)

Now we're ready to start making the app itself. The typical first step when
making a web application is to create a _data model_, which is a
representation of the structures needed by our application. In our case, the
demo app will be a microblog, with only users and short (micro)posts. Thus,
we'll begin with a model for _users_ of the app
(Section2.1.1,
and then we'll add a model for _microposts_
([Section2.1.2](a-demo-app.html#sec-
modeling_demo_microposts)).

### 2.1.1 Modeling demo users

There are as many choices for a user data model as there are different
registration forms on the web; we'll go with a distinctly minimalist approach.
Users of our demo app will have a unique `integer` identifier called `id`, a
publicly viewable `name` (of type `string`), and an `email` address (also a
`string`) that will double as a username. A summary of the data model for
users appears in [Figure2.2](a-demo-app.html#fig-
demo_user_model).

!demo_user_model

Figure 2.2: The data model for users.

As we'll see starting in [Section6.1.1](modeling-users.html
#sec-database_migrations), the label `users` in
Figure2.2
corresponds to a _table_ in a database, and the `id`, `name`, and `email`
attributes are _columns_ in that table.

### [2.1.2 Modeling demo microposts](a-demo-app.html#sec-
modeling_demo_microposts)

The core of the micropost data model is even simpler than the one for users: a
micropost has only an `id` and a `content` field for the micropost's text (of
type `string`).1 There's an additional complication, though: we want to
_associate_ each micropost with a particular user; we'll accomplish this by
recording the `user_id` of the owner of the post. The results are shown in
Figure2.3.

!demo_micropost_model

Figure 2.3: The data model for microposts.

We'll see in [Section2.3.3](a-demo-app.html#sec-
demo_user_has_many_microposts) (and more fully in
Chapter10 how this `user_id`
attribute allows us to succinctly express the notion that a user potentially
has many associated microposts.

## 2.2 The Users resource

In this section, we'll implement the users data model in
Section2.1.1,
along with a web interface to that model. The combination will constitute a
_Users resource_, which will allow us to think of users as objects that can be
created, read, updated, and deleted through the web via the [HTTP
protocol](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol). As
promised in the introduction, our Users resource will be created by a scaffold
generator program, which comes standard with each Rails project. I urge you
not to look too closely at the generated code; at this stage, it will only
serve to confuse you.

Rails scaffolding is generated by passing the `scaffold` command to the `rails
generate` script. The argument of the `scaffold` command is the singular
version of the resource name (in this case, `User`), together with optional
parameters for the data model's attributes:2

    
    $ rails generate scaffold User name:string email:string
          invoke  active_record
          create    db/migrate/20111123225336_create_users.rb
          create    app/models/user.rb
          invoke    test_unit
          create      test/unit/user_test.rb
          create      test/fixtures/users.yml
           route  resources :users
          invoke  scaffold_controller
          create    app/controllers/users_controller.rb
          invoke    erb
          create      app/views/users
          create      app/views/users/index.html.erb
          create      app/views/users/edit.html.erb
          create      app/views/users/show.html.erb
          create      app/views/users/new.html.erb
          create      app/views/users/_form.html.erb
          invoke    test_unit
          create      test/functional/users_controller_test.rb
          invoke    helper
          create      app/helpers/users_helper.rb
          invoke      test_unit
          create        test/unit/helpers/users_helper_test.rb
          invoke  assets
          invoke    coffee
          create      app/assets/javascripts/users.js.coffee
          invoke    scss
          create      app/assets/stylesheets/users.css.scss
          invoke  scss
          create    app/assets/stylesheets/scaffolds.css.scss
    

By including `name:string` and `email:string`, we have arranged for the User
model to have the form shown in [Figure2.2](a-demo-app.html
#fig-demo_user_model). (Note that there is no need to include a parameter
for`id`; it is created automatically by Rails for use as
the _primary key_ in the database.)

To proceed with the demo application, we first need to _migrate_ the database
using _Rake_ (Box2.1:

    
    $ bundle exec rake db:migrate
    ==  CreateUsers: migrating ====================================================
    -- create_table(:users)
       -> 0.0017s
    ==  CreateUsers: migrated (0.0018s) ===========================================
    

This simply updates the database with our new `users` data model. (We'll learn
more about database migrations starting in [Section6.1.1
](modeling-users.html#sec-database_migrations).) Note that, in order to ensure
that the command uses the version of Rake corresponding to our `Gemfile`, we
need to run `rake` using `bundle exec`.

With that, we can run the local web server using `rails s`, which is a
shortcut for `rails server`:

    
    $ rails s
    

Now the demo application should be ready to go at
http://localhost:3000/.

Box 2.1.Rake

In the Unix tradition, the
_make_ utility has played an
important role in building executable programs from source code; many a
computer hacker has committed to muscle memory the line

    
      $ ./configure && make && sudo make install

commonly used to compile code on Unix systems (including Linux and Mac
OSX).

Rake is _Ruby make_, a make-like language written in Ruby. Rails uses Rake
extensively, especially for the innumerable little administrative tasks
necessary when developing database-backed web applications. The `rake
db:migrate` command is probably the most common, but there are many others;
you can see a list of database tasks using `-T db`:

    
    $ bundle exec rake -T db

To see all the Rake tasks available, run

    
    $ bundle exec rake -T

The list is likely to be overwhelming, but don't worry, you don't have to know
all (or even most) of these commands. By the end of the _Rails Tutorial_,
you'll know all the most important ones.

### 2.2.1 A user tour

Visiting the root
urlhttp://localhost:3000/ shows
the same default Rails page shown in
Figure1.3, but in
generating the Users resource scaffolding we have also created a large number
of pages for manipulating users. For example, the page for listing all users
is at /users, and the page for making a new
user is at /users/new. The rest of this
section is dedicated to taking a whirlwind tour through these user pages. As
we proceed, it may help to refer to [Table2.1](a-demo-
app.html#table-user_urls), which shows the correspondence between pages and
URIs.

**URI****Action****Purpose**

/users

`index`

page to list all users

/users/1

`show`

page to show user with id `1`

/users/new

`new`

page to make a new user

/users/1/edit

`edit`

page to edit user with id `1`

Table 2.1: The correspondence between pages and URIs for the Users resource.

We start with the page to show all the users in our application, called
`index`; as you might expect, initially there
are no users at all ([Figure2.4](a-demo-app.html#fig-
demo_blank_user_index_rails_3)).

![demo_blank_user_index_rails_3](images/figures/demo_blank_user_index_rails_3.
png)

Figure 2.4: The initial index page for the Users resource
(/users](http:
//railstutorial.org/images/figures/demo_blank_user_index_rails_3-full.png)

To make a new user, we visit the `new`
page, as shown in [Figure2.5](a-demo-app.html#fig-
demo_new_user_rails_3). (Since the http://localhost:3000 part of the address
is implicit whenever we are developing locally, I'll usually omit it from now
on.) In Chapter7, this will become the
user signup page.

!demo_new_user_rails_3

Figure 2.5: The new user page
(/users/new.[(full
size)](http://railstutorial.org/images/figures/demo_new_user_rails_3-full.png)

We can create a user by entering name and email values in the text fields and
then clicking the Create User button. The result is the user
`show` page, as seen in
Figure2.6.
(The green welcome message is accomplished using the _flash_, which we'll
learn about in Section7.4.2
Note that the URI is /users/1; as you might
suspect, the number`1` is simply the
user's`id` attribute from [Figure2.2](a
-demo-app.html#fig-demo_user_model). In [Section7.1](sign-
up.html#sec-showing_users), this page will become the user's profile.

!demo_show_user_rails_3

Figure 2.6: The page to show a user
(/users/1](h
ttp://railstutorial.org/images/figures/demo_show_user_rails_3-full.png)

To change a user's information, we visit the
`edit` page
(Figure2.7.
By modifying the user information and clicking the Update User button, we
arrange to change the information for the user in the demo application
([Figure2.8](a-demo-app.html#fig-
demo_update_user_rails_3)). (As we'll see in detail starting in
Chapter6, this user data is
stored in a database back-end.) We'll add user edit/update functionality to
the sample application in [Section9.1](updating-showing-
and-deleting-users.html#sec-updating_users).

!demo_edit_user_rails_3

Figure 2.7: The user edit page ([/users/1/edit](http://localhost:3000/users/1/
edit)).[(full size)](http://railstutorial.org/images/figure
s/demo_edit_user_rails_3-full.png)

!demo_update_user_rails_3

Figure 2.8: A user with updated information.[(full size)](h
ttp://railstutorial.org/images/figures/demo_update_user_rails_3-full.png)

Now we'll create a second user by revisiting the
`new` page and submitting a second set of
user information; the resulting user `index` is
shown in [Figure2.9](a-demo-app.html#fig-
demo_user_index_two_rails_3). [Section7.1](sign-up.html
#sec-showing_users) will develop the user index into a more polished page for
showing all users.

!demo_user_index_two_rails_3

Figure 2.9: The user index page (/users with a
second user.[(full size)](http://railstutorial.org/images/f
igures/demo_user_index_two_rails_3-full.png)

Having shown how to create, show, and edit users, we come finally to
destroying them ([Figure2.10](a-demo-app.html#fig-
demo_destroy_user_rails_3)). You should verify that clicking on the link in
Figure2.10
destroys the second user, yielding an index page with only one user. (If it
doesn't work, be sure that JavaScript is enabled in your browser; Rails uses
JavaScript to issue the request needed to destroy a user.)
[Section9.4](updating-showing-and-deleting-users.html#sec-
destroying_users) adds user deletion to the sample app, taking care to
restrict its use to a special class of administrative users.

!demo_destroy_user_rails_3

Figure 2.10: Destroying a user.[(full size)](http://railstu
torial.org/images/figures/demo_destroy_user_rails_3-full.png)

### 2.2.2 MVC in action

Now that we've completed a quick overview of the Users resource, let's examine
one particular part of it in the context of the Model-View-Controller (MVC)
pattern introduced in [Section1.2.6](beginning.html#sec-
mvc). Our strategy will be to describe the results of a typical browser hit--a
visit to the user index page at /users--in
terms of MVC ([Figure2.11](a-demo-app.html#fig-
mvc_detailed)).

!mvc_detailed

Figure 2.11: A detailed diagram of MVC in Rails.[(full
size)](http://railstutorial.org/images/figures/mvc_detailed-full.png)

  1. The browser issues a request for the /users URI.
  2. Rails routes /users to the `index` action in the Users controller.
  3. The `index` action asks the User model to retrieve all users (`User.all`).
  4. The User model pulls all the users from the database.
  5. The User model returns the list of users to the controller.
  6. The controller captures the users in the `@users` variable, which is passed to the `index` view.
  7. The view uses embedded Ruby to render the page as HTML.
  8. The controller passes the HTML back to the browser.3

We start with a request issued from the browser--i.e., the result of typing a
URI in the address bar or clicking on a link (Step1 in
Figure2.11. This
request hits the _Rails router_ (Step2), which dispatches
to the proper _controller action_ based on the URI (and, as we'll see in
Box3.2, the type of
request). The code to create the mapping of user URIs to controller actions
for the Users resource appears in [Listing2.2](a-demo-
app.html#code-rails_routes); this code effectively sets up the table of
URI/action pairs seen in [Table2.1](a-demo-app.html#table-
user_urls). (The strange notation `:users` is a _symbol_, which we'll learn
about in [Section4.3.3](rails-flavored-ruby.html#sec-
hashes_and_symbols).)

Listing 2.2. The Rails routes, with a rule for the Users resource.

`config/routes.rb`

    
    DemoApp::Application.routes.draw do
      resources :users
      .
      .
      .
    end
    

The pages from the tour in [Section2.2.1](a-demo-app.html
#sec-a_user_tour) correspond to _actions_ in the Users _controller_, which is
a collection of related actions; the controller generated by the scaffolding
is shown schematically in [Listing2.3](a-demo-app.html
#code-demo_users_controller). Note the notation `class UsersController <
ApplicationController`; this is an example of a Ruby _class_ with
_inheritance_. (We'll discuss inheritance briefly in
Section2.3.4
and cover both subjects in more detail in [Section4.4
](rails-flavored-ruby.html#sec-ruby_classes).)

Listing 2.3. The Users controller in schematic form.

`app/controllers/users_controller.rb`

    
    class UsersController < ApplicationController
    
      def index
        .
        .
        .
      end
    
      def show
        .
        .
        .
      end
    
      def new
        .
        .
        .
      end
    
      def create
        .
        .
        .
      end
    
      def edit
        .
        .
        .
      end
    
      def update
        .
        .
        .
      end
    
      def destroy
        .
        .
        .
      end
    end
    

You may notice that there are more actions than there are pages; the `index`,
`show`, `new`, and `edit` actions all correspond to pages from
Section2.2.1, but there
are additional `create`, `update`, and `destroy` actions as well. These
actions don't typically render pages (although they sometimes do); instead,
their main purpose is to modify information about users in the database. This
full suite of controller actions, summarized in
Table2.2,
represents the implementation of the REST architecture in Rails
(Box2.2, which is based on
the ideas of _representational state transfer_ identified and named by
computer scientist Roy Fielding.4
Note from [Table2.2](a-demo-app.html#table-
demo_RESTful_users) that there is some overlap in the URIs; for example, both
the user `show` action and the `update` action correspond to the URI /users/1.
The difference between them is the [HTTP request
method](http://en.wikipedia.org/wiki/HTTP_request#Request_methods) they
respond to. We'll learn more about HTTP request methods starting in
Section3.2.1.

**HTTP request****URI****Action****Purpose**

`GET`

/users

`index`

page to list all users

`GET`

/users/1

`show`

page to show user with id `1`

`GET`

/users/new

`new`

page to make a new user

`POST`

/users

`create`

create a new user

`GET`

/users/1/edit

`edit`

page to edit user with id `1`

`PUT`

/users/1

`update`

update user with id `1`

`DELETE`

/users/1

`destroy`

delete user with id `1`

Table 2.2: RESTful routes provided by the Users resource in
Listing2.2.

Box 2.2.REpresentational State Transfer (REST)

If you read much about Ruby on Rails web development, you'll see a lot of
references to "REST", which is an acronym for REpresentational State Transfer.
REST is an architectural style for developing distributed, networked systems
and software applications such as the World Wide Web and web applications.
Although REST theory is rather abstract, in the context of Rails applications
REST means that most application components (such as users and microposts) are
modeled as _resources_ that can be created, read, updated, and deleted--
operations that correspond both to the [CRUD operations of relational
databases](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) and
the four fundamental [HTTP request
methods](http://en.wikipedia.org/wiki/HTTP_request#Request_methods): `POST`,
`GET`, `PUT`, and `DELETE`. (We'll learn more about HTTP requests in
Section3.2.1 and especially
Box3.2

As a Rails application developer, the RESTful style of development helps you
make choices about which controllers and actions to write: you simply
structure the application using resources that get created, read, updated, and
deleted. In the case of users and microposts, this process is straightforward,
since they are naturally resources in their own right. In
Chapter11, we'll see an example
where REST principles allow us to model a subtler problem, "following users",
in a natural and convenient way.

To examine the relationship between the Users controller and the User model,
let's focus on a simplified version of the `index` action, shown in
Listing2.4. (The
scaffold code is ugly and confusing, so I've suppressed it.)

Listing 2.4. The simplified user `index` action for the demo application.

`app/controllers/users_controller.rb`

    
    class UsersController < ApplicationController
    
      def index
        @users = User.all
      end
      .
      .
      .
    end
    

This `index` action has the line `@users = User.all`
(Step3), which asks the User model to retrieve a list of
all the users from the database (Step4), and then places
them in the variable `@users` (pronounced "at-users")
(Step5). The User model itself appears in
Listing2.5;
although it is rather plain, it comes equipped with a large amount of
functionality because of inheritance ([Section2.3.4](a
-demo-app.html#sec-inheritance_hierarchies) and [Section4.4
](rails-flavored-ruby.html#sec-ruby_classes)). In particular, by using the
Rails library called _Active Record_, the code in
Listing2.5 arranges
for `User.all` to return all the users. (We'll learn about the
`attr_accessible` line in [Section6.1.2.2](modeling-
users.html#sec-accessible_attributes). _Note_: This line will not appear if
you are using Rails3.2.2 or earlier.)

Listing 2.5. The User model for the demo application.

`app/models/user.rb`

    
    class User < ActiveRecord::Base
      attr_accessible :email, :name
    end
    

Once the `@users` variable is defined, the controller calls the _view_
(Step6), shown in [Listing2.6](a-demo-
app.html#code-demo_index_view). Variables that start with the
`@`sign, called _instance variables_, are automatically
available in the view; in this case, the `index.html.erb` view in
Listing2.6 iterates
through the `@users` list and outputs a line of HTML for each one. (Remember,
you aren't supposed to understand this code right now. It is shown only for
purposes of illustration.)

Listing 2.6. The view for the user index.

`app/views/users/index.html.erb`

    
    <h1>Listing users</h1>
    
    <table>
      <tr>
        <th>Name</th>
        <th>Email</th>
        <th></th>
        <th></th>
        <th></th>
      </tr>
    
    <% @users.each do |user| %>
      <tr>
        <td><%= user.name %></td>
        <td><%= user.email %></td>
        <td><%= link_to 'Show', user %></td>
        <td><%= link_to 'Edit', edit_user_path(user) %></td>
        <td><%= link_to 'Destroy', user, method: :delete,
                                         data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
    </table>
    
    <br />
    
    <%= link_to 'New User', new_user_path %>
    

The view converts its contents to HTML (Step7), which is
then returned by the controller to the browser for display
(Step8).

### [2.2.3 Weaknesses of this Users resource](a-demo-app.html#sec-
weaknesses_of_this_users_resource)

Though good for getting a general overview of Rails, the scaffold Users
resource suffers from a number of severe weaknesses.

  * **No data validations.** Our User model accepts data such as blank names and invalid email addresses without complaint.
  * **No authentication.** We have no notion signing in or out, and no way to prevent any user from performing any operation.
  * **No tests.** This isn't technically true--the scaffolding includes rudimentary tests--but the generated tests are ugly and inflexible, and they don't test for data validation, authentication, or any other custom requirements.
  * **No layout.** There is no consistent site styling or navigation.
  * **No real understanding.** If you understand the scaffold code, you probably shouldn't be reading this book.

## 2.3 The Microposts resource

Having generated and explored the Users resource, we turn now to the
associated Microposts resource. Throughout this section, I recommend comparing
the elements of the Microposts resource with the analogous user elements from
Section2.2; you
should see that the two resources parallel each other in many ways. The
RESTful structure of Rails applications is best absorbed by this sort of
repetition of form; indeed, seeing the parallel structure of Users and
Microposts even at this early stage is one of the prime motivations for this
chapter. (As we'll see, writing applications more robust than the toy example
in this chapter takes considerable effort--we won't see the Microposts
resource again until [Chapter10](user-
microposts.html#top)--and I didn't want to defer its first appearance quite
that far.)

### 2.3.1 A micropost microtour

As with the Users resource, we'll generate scaffold code for the Microposts
resource using `rails generate scaffold`, in this case implementing the data
model from [Figure2.3](a-demo-app.html#fig-
demo_micropost_model):5

    
    $ rails generate scaffold Micropost content:string user_id:integer
          invoke  active_record
          create    db/migrate/20111123225811_create_microposts.rb
          create    app/models/micropost.rb
          invoke    test_unit
          create      test/unit/micropost_test.rb
          create      test/fixtures/microposts.yml
           route  resources :microposts
          invoke  scaffold_controller
          create    app/controllers/microposts_controller.rb
          invoke    erb
          create      app/views/microposts
          create      app/views/microposts/index.html.erb
          create      app/views/microposts/edit.html.erb
          create      app/views/microposts/show.html.erb
          create      app/views/microposts/new.html.erb
          create      app/views/microposts/_form.html.erb
          invoke    test_unit
          create      test/functional/microposts_controller_test.rb
          invoke    helper
          create      app/helpers/microposts_helper.rb
          invoke      test_unit
          create        test/unit/helpers/microposts_helper_test.rb
          invoke  assets
          invoke    coffee
          create      app/assets/javascripts/microposts.js.coffee
          invoke    scss
          create      app/assets/stylesheets/microposts.css.scss
          invoke  scss
       identical    app/assets/stylesheets/scaffolds.css.scss
    

To update our database with the new data model, we need to run a migration as
in Section2.2:

    
    $ bundle exec rake db:migrate
    ==  CreateMicroposts: migrating ===============================================
    -- create_table(:microposts)
       -> 0.0023s
    ==  CreateMicroposts: migrated (0.0026s) ======================================
    

Now we are in a position to create microposts in the same way we created users
in Section2.2.1. As you
might guess, the scaffold generator has updated the Rails routes file with a
rule for Microposts resource, as seen in [Listing2.7](a
-demo-app.html#code-demo_microposts_resource).6 As with users, the `resources
:microposts` routing rule maps micropost URIs to actions in the Microposts
controller, as seen in [Table2.3](a-demo-app.html#table-
demo_RESTful_microposts).

Listing 2.7. The Rails routes, with a new rule for Microposts resources.

`config/routes.rb`

    
    DemoApp::Application.routes.draw do
      resources :microposts
      resources :users
      .
      .
      .
    end
    

**HTTP request****URI****Action****Purpose**

`GET`

/microposts

`index`

page to list all microposts

`GET`

/microposts/1

`show`

page to show micropost with id `1`

`GET`

/microposts/new

`new`

page to make a new micropost

`POST`

/microposts

`create`

create a new micropost

`GET`

/microposts/1/edit

`edit`

page to edit micropost with id `1`

`PUT`

/microposts/1

`update`

update micropost with id `1`

`DELETE`

/microposts/1

`destroy`

delete micropost with id `1`

Table 2.3: RESTful routes provided by the Microposts resource in
[Listing2.7](a-demo-app.html#code-
demo_microposts_resource).

The Microposts controller itself appears in schematic form
[Listing2.8](a-demo-app.html#code-
demo_microposts_controller). Note that, apart from having
`MicropostsController` in place of `UsersController`,
[Listing2.8](a-demo-app.html#code-
demo_microposts_controller) is _identical_ to the code in
Listing2.3.
This is a reflection of the REST architecture common to both resources.

Listing 2.8. The Microposts controller in schematic form.

`app/controllers/microposts_controller.rb`

    
    class MicropostsController < ApplicationController
    
      def index
        .
        .
        .
      end
    
      def show
        .
        .
        .
      end
    
      def new
        .
        .
        .
      end
    
      def create
        .
        .
        .
      end
    
      def edit
        .
        .
        .
      end
    
      def update
        .
        .
        .
      end
    
      def destroy
        .
        .
        .
      end
    end
    

To make some actual microposts, we enter information at the new microposts
page, /microposts/new, as seen in
[Figure2.12](a-demo-app.html#fig-
demo_new_micropost_rails_3).

!demo_new_micropost_rails_3

Figure 2.12: The new micropost page ([/microposts/new](http://localhost:3000/m
icroposts/new)).[(full
size)](http://railstutorial.org/images/figures/demo_new_micropost-full.png)

At this point, go ahead and create a micropost or two, taking care to make
sure that at least one has a `user_id` of`1` to match the
id of the first user created in [Section2.2.1](a-demo-
app.html#sec-a_user_tour). The result should look something like
[Figure2.13](a-demo-app.html#fig-
demo_micropost_index_rails_3).

![demo_micropost_index_rails_3](images/figures/demo_micropost_index_rails_3.pn
g)

Figure 2.13: The micropost index page
(/microposts.[(full si
ze)](http://railstutorial.org/images/figures/demo_micropost_index_rails_3-full
.png)

### [2.3.2 Putting the _micro_ in microposts](a-demo-app.html#sec-
putting_the_micro_in_microposts)

Any _micro_post worthy of the name should have some means of enforcing the
length of the post. Implementing this constraint in Rails is easy with
_validations_; to accept microposts with at most 140 characters (a la
Twitter), we use a _length_ validation. At this point, you should open the
file `app/models/micropost.rb` in your text editor or IDE and fill it with the
contents of [Listing2.9](a-demo-app.html#code-
demo_length_validation). (The use of `validates` in
Listing2.9
is characteristic of Rails3; if you've previously worked
with Rails2.3, you should compare this to the use of
`validates_length_of`.)

Listing 2.9. Constraining microposts to be at most 140 characters.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      attr_accessible :content, :user_id
      validates :content, :length => { :maximum => 140 }
    end
    

The code in [Listing2.9](a-demo-app.html#code-
demo_length_validation) may look rather mysterious--we'll cover validations
more thoroughly starting in [Section6.2](modeling-
users.html#sec-user_validations)--but its effects are readily apparent if we
go to the new micropost page and enter more than 140 characters for the
content of the post. As seen in [Figure2.14](a-demo-
app.html#fig-micropost_length_error_rails_3), Rails renders _error messages_
indicating that the micropost's content is too long. (We'll learn more about
error messages in [Section7.3.2](sign-up.html#sec-
signup_error_messages).)

![micropost_length_error_rails_3](images/figures/micropost_length_error_rails_
3.png)

Figure 2.14: Error messages for a failed micropost
creation.[(full size)](http://railstutorial.org/images/figu
res/micropost_length_error_rails_3-full.png)

### [2.3.3 A user `has_many` microposts](a-demo-app.html#sec-
demo_user_has_many_microposts)

One of the most powerful features of Rails is the ability to form
_associations_ between different data models. In the case of our User model,
each user potentially has many microposts. We can express this in code by
updating the User and Micropost models as in
[Listing2.10](a-demo-app.html#code-
demo_user_has_many_microposts) and [Listing2.11](a-demo-
app.html#code-demo_micropost_belongs_to_user).

Listing 2.10. A user has many microposts.

`app/models/user.rb`

    
    class User < ActiveRecord::Base
      attr_accessible :email, :name
      has_many :microposts
    end
    

Listing 2.11. A micropost belongs to a user.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      attr_accessible :content, :user_id
    
      belongs_to :user
    
      validates :content, :length => { :maximum => 140 }
    end
    

We can visualize the result of this association in
[Figure2.15](a-demo-app.html#fig-
micropost_user_association). Because of the `user_id` column in the
`microposts` table, Rails (using Active Record) can infer the microposts
associated with each user.

!micropost_user_association

Figure 2.15: The association between microposts and users.

In Chapter10 and
Chapter11, we will use the
association of users and microposts both to display all a user's microposts
and to construct a Twitter-like micropost feed. For now, we can examine the
implications of the user-micropost association by using the _console_, which
is a useful tool for interacting with Rails applications. We first invoke the
console with `rails console` at the command line, and then retrieve the first
user from the database using `User.first` (putting the results in the variable
`first_user`):7

    
    $ rails console
    >> first_user = User.first
    => #<User id: 1, name: "Michael Hartl", email: "michael@example.org",
    created_at: "2011-11-03 02:01:31", updated_at: "2011-11-03 02:01:31">
    >> first_user.microposts
    => [#<Micropost id: 1, content: "First micropost!", user_id: 1, created_at:
    "2011-11-03 02:37:37", updated_at: "2011-11-03 02:37:37">, #<Micropost id: 2,
    content: "Second micropost", user_id: 1, created_at: "2011-11-03 02:38:54",
    updated_at: "2011-11-03 02:38:54">]
    >> exit
    

(I include the last line just to demonstrate how to exit the console, and on
most systems you can Ctrl-d for the same purpose.) Here we have accessed the
user's microposts using the code `first_user.microposts`: with this code,
Active Record automatically returns all the microposts with `user_id` equal to
the id of `first_user` (in this case,`1`). We'll learn much
more about the association facilities in Active Record in
Chapter10 and
Chapter11.

### [2.3.4 Inheritance hierarchies](a-demo-app.html#sec-
inheritance_hierarchies)

We end our discussion of the demo application with a brief description of the
controller and model class hierarchies in Rails. This discussion will only
make much sense if you have some experience with object-oriented programming
(OOP); if you haven't studied OOP, feel free to skip this section. In
particular, if you are unfamiliar with _classes_ (discussed in
Section4.4, I
suggest looping back to this section at a later time.

We start with the inheritance structure for models. Comparing
Listing2.12 and
Listing2.13,
we see that both the User model and the Micropost model inherit (via the left
angle bracket`<`) from `ActiveRecord::Base`, which is the
base class for models provided by ActiveRecord; a diagram summarizing this
relationship appears in [Figure2.16](a-demo-app.html#fig-
demo_model_inheritance). It is by inheriting from `ActiveRecord::Base` that
our model objects gain the ability to communicate with the database, treat the
database columns as Ruby attributes, and soon.

Listing 2.12. The `User` class, with inheritance.

`app/models/user.rb`

    
    class User < ActiveRecord::Base
      .
      .
      .
    end
    

Listing 2.13. The `Micropost` class, with inheritance.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      .
      .
      .
    end
    

!demo_model_inheritance

Figure 2.16: The inheritance hierarchy for the User and Micropost models.

The inheritance structure for controllers is only slightly more complicated.
Comparing [Listing2.14](a-demo-app.html#code-
demo_users_controller_class) and [Listing2.15](a-demo-
app.html#code-demo_microposts_controller_class), we see that both the Users
controller and the Microposts controller inherit from the Application
controller. Examining [Listing2.16](a-demo-app.html#code-
demo_application_controller_class), we see that `ApplicationController` itself
inherits from `ActionController::Base`; this is the base class for controllers
provided by the Rails library Action Pack. The relationships between these
classes is illustrated in [Figure2.17](a-demo-app.html#fig-
demo_controller_inheritance).

Listing 2.14. The `UsersController` class, with inheritance.

`app/controllers/users_controller.rb`

    
    class UsersController < ApplicationController
      .
      .
      .
    end
    

Listing 2.15. The `MicropostsController` class, with inheritance.

`app/controllers/microposts_controller.rb`

    
    class MicropostsController < ApplicationController
      .
      .
      .
    end
    

Listing 2.16. The `ApplicationController` class, with inheritance.

`app/controllers/application_controller.rb`

    
    class ApplicationController < ActionController::Base
      .
      .
      .
    end
    

!demo_controller_inheritance

Figure 2.17: The inheritance hierarchy for the Users and Microposts
controllers.

As with model inheritance, by inheriting ultimately from
`ActionController::Base` both the Users and Microposts controllers gain a
large amount of functionality, such as the ability to manipulate model
objects, filter inbound HTTP requests, and render views as HTML. Since all
Rails controllers inherit from `ApplicationController`, rules defined in the
Application controller automatically apply to every action in the application.
For example, in [Section8.2.1](sign-in-sign-out.html#sec-
remember_me) we'll see how to include helpers for signing in and signing out
of all of the sample application's controllers.

### 2.3.5 Deploying the demo app

With the completion of the Microposts resource, now is a good time to push the
repository up to GitHub:

    
    $ git add .
    $ git commit -m "Finish demo app"
    $ git push
    

Ordinarily, you should make smaller, more frequent commits, but for the
purposes of this chapter a single big commit at the end is fine.

At this point, you can also deploy the demo app to Heroku as in
Section1.4:

    
    $ heroku create --stack cedar
    $ git push heroku master
    

Finally, migrate the production database (see below if you get a deprecation
warning):

    
    $ heroku run rake db:migrate
    

This updates the database at Heroku with the necessary user/micropost data
model. You may get a deprecation warning regarding assets in `vendor/plugins`,
which you should ignore since there aren't any plugins in that directory.

## 2.4 Conclusion

We've come now to the end of the 30,000-foot view of a Rails application. The
demo app developed in this chapter has several strengths and a host of
weaknesses.

**Strengths**

  * High-level overview of Rails
  * Introduction to MVC
  * First taste of the REST architecture
  * Beginning data modeling
  * A live, database-backed web application in production

**Weaknesses**

  * No custom layout or styling
  * No static pages (like "Home" or "About")
  * No user passwords
  * No user images
  * No signing in
  * No security
  * No automatic user/micropost association
  * No notion of "following" or "followed"
  * No micropost feed
  * No test-driven development
  * **No real understanding**

The rest of this tutorial is dedicated to building on the strengths and
eliminating the weaknesses.

 «Chapter 1 From zero to deploy  [
Chapter 3 Mostly static pages» ](static-pages.html#top)

  1. When modeling longer posts, such as those for a normal (non-micro) blog, you should use the `text` type in place of `string`.↑
  2. The name of the scaffold follows the convention of _models_, which are singular, rather than resources and controllers, which are plural. Thus, we have `User` instead `Users`.↑
  3. Some references indicate that the view returns the HTML directly to the browser (via a web server such as Apache or Nginx). Regardless of the implementation details, I prefer to think of the controller as a central hub through which all the application's information flows.↑
  4. Fielding, Roy Thomas. _Architectural Styles and the Design of Network-based Software Architectures_. Doctoral dissertation, University of California, Irvine, 2000.↑
  5. As with the User scaffold, the scaffold generator for microposts follows the singular convention of Rails models; thus, we have `generate Micropost`.↑
  6. The scaffold code may have extra newlines compared to Listing2.7. This is not a cause for concern, as Ruby ignores extra newlines.↑
  7. Your console prompt might be something like `ruby-1.9.3-head >`, but the examples use`>>` since Ruby versions will vary.↑

