# Chapter 1 From zero to deploy

Welcome to the [_Ruby on Rails Tutorial_](http://ruby.railstutorial.org/ruby-
on-rails-tutorial-book). The goal of this book is to be the best answer to the
question, "If I want to learn web development with [Ruby on
Rails](http://rubyonrails.org/), where should I start?" By the time you finish
the _Ruby on Rails Tutorial_, you will have all the skills you need to develop
and deploy your own custom web applications with Rails. You will also be ready
to benefit from the many more advanced books, blogs, and screencasts that are
part of the thriving Rails educational ecosystem. Finally, since the _Ruby on
Rails Tutorial_ uses Rails3, the knowledge you gain here
represents the state of the art in web development. (The most up-to-date
version of the _Ruby on Rails Tutorial_ can be found on the book's website at
http://railstutorial.org/; if you are reading
this book offline, be sure to check the [online version of the Rails Tutorial
book](http://railstutorial.org/book) at
http://railstutorial.org/book for the latest
updates.)

Note that the goal of this book is _not_ merely to teach Rails, but rather to
teach _web development with Rails_, which means acquiring (or expanding) the
skills needed to develop software for the World Wide Web. In addition to Ruby
on Rails, this skillset includes HTML & CSS, databases, version control,
testing, and deployment. To accomplish this goal, the _Ruby on Rails Tutorial_
takes an integrated approach: you will learn Rails by example by building a
substantial sample application from scratch. As [Derek
Sivers](http://sivers.org) notes in the foreword, this book is structured as a
linear narrative, designed to be read from start to finish. If you are used to
skipping around in technical books, taking this linear approach might require
some adjustment, but I suggest giving it a try. You can think of the _Ruby on
Rails Tutorial_ as a video game where you are the main character, and where
you level up as a Rails developer in each chapter. (The exercises are the
minibosses

In this first chapter, we'll get started with Ruby on Rails by installing all
the necessary software and by setting up our development environment
(Section1.2. We'll
then create our first Rails application, called (appropriately enough)
`first_app`. The _Rails Tutorial_ emphasizes good software development
practices, so immediately after creating our fresh new Rails project we'll put
it under version control with Git
(Section1.3. And,
believe it or not, in this chapter we'll even put our first app on the wider
web by _deploying_ it to production
(Section1.4.

In Chapter2, we'll make a second
project, whose purpose is to demonstrate the basic workings of a Rails
application. To get up and running quickly, we'll build this _demo app_
(called `demo_app`) using scaffolding
(Box1.1 to generate
code; since this code is both ugly and complex,
Chapter2 will focus on interacting
with the demo app through its _URIs_ (sometimes called _URLs_)1 using a web
browser.

The rest of the tutorial focuses on developing a single large _sample
application_ (called `sample_app`), writing all the code from scratch. We'll
develop the sample app using _test-driven development_ (TDD), getting started
in Chapter3 by creating static
pages and then adding a little dynamic content. We'll take a quick detour in
Chapter4 to learn a little
about the Ruby language underlying Rails. Then, in
Chapter5 through
Chapter9,
we'll complete the foundation for the sample application by making a site
layout, a user data model, and a full registration and authentication system.
Finally, in Chapter10 and
Chapter11 we'll add
microblogging and social features to make a working example site.

The final sample application will bear more than a passing resemblance to a
certain popular social microblogging site--a site
which, coincidentally, was also originally written in Rails. Though of
necessity our efforts will focus on this specific sample application, the
emphasis throughout the _Rails Tutorial_ will be on general principles, so
that you will have a solid foundation no matter what kinds of web applications
you want to build.

Box 1.1.Scaffolding: Quicker, easier, more seductive

From the beginning, Rails has benefited from a palpable sense of excitement,
starting with the famous [15-minute weblog
video](http://media.rubyonrails.org/video/rails_take2_with_sound.mov) by Rails
creator David Heinemeier Hansson. That video and its successors are a great
way to get a taste of Rails' power, and I recommend watching them. But be
warned: they accomplish their amazing fifteen-minute feat using a feature
called _scaffolding_, which relies heavily on _generated code_, magically
created by the Rails `generate` command.

When writing a Ruby on Rails tutorial, it is tempting to rely on the
scaffolding approach--it's [quicker, easier, more
seductive](http://en.wikipedia.org/wiki/Dark_side_(Star_Wars)). But the
complexity and sheer amount of code in the scaffolding can be utterly
overwhelming to a beginning Rails developer; you may be able to use it, but
you probably won't understand it. Following the scaffolding approach risks
turning you into a virtuoso script generator with little (and brittle) actual
knowledge of Rails.

In the _Ruby on Rails Tutorial_, we'll take the (nearly) polar opposite
approach: although Chapter2 will
develop a small demo app using scaffolding, the core of the _Rails Tutorial_
is the sample app, which we'll start writing in [Chapter3
](static-pages.html#top). At each stage of developing the sample application,
we will write _small, bite-sized_ pieces of code--simple enough to understand,
yet novel enough to be challenging. The cumulative effect will be a deeper,
more flexible knowledge of Rails, giving you a good background for writing
nearly any type of web application.

## 1.1 Introduction

Since its debut in 2004, Ruby on Rails has rapidly become one of the most
powerful and popular frameworks for building dynamic web applications.
Everyone from scrappy startups to huge companies have used Rails:
37signals](http://37signals.com/), [GitHub,
Shopify](http://shopify.com/), [Scribd,
Twitter](http://twitter.com/), [LivingSocial,
Groupon](http://groupon.com/), [Hulu, the [Yellow
Pages](http://yellowpages.com/)--the [list of sites using
Rails](http://rubyonrails.org/applications) goes on and on. There are also
many web development shops that specialize in Rails, such as
ENTP](http://entp.com/), [thoughtbot, [Pivotal
Labs](http://pivotallabs.com/), and Hashrocket, plus
innumerable independent consultants, trainers, and contractors.

What makes Rails so great? First of all, Ruby on Rails is 100% open-source,
available under the permissive [MIT
License](http://www.opensource.org/licenses/mit-license.php), and as a result
it also costs nothing to download or use. Rails also owes much of its success
to its elegant and compact design; by exploiting the malleability of the
underlying Ruby language, Rails effectively creates a
[domain-specific
language](http://en.wikipedia.org/wiki/Domain_Specific_Language) for writing
web applications. As a result, many common web programming tasks--such as
generating HTML, making data models, and routing URIs--are easy with Rails,
and the resulting application code is concise and readable.

Rails also adapts rapidly to new developments in web technology and framework
design. For example, Rails was one of the first frameworks to fully digest and
implement the REST architectural style for structuring web applications (which
we'll be learning about throughout this tutorial). And when other frameworks
develop successful new techniques, Rails creator [David Heinemeier
Hansson](http://loudthinking.com/) and the [Rails core
team](http://rubyonrails.org/core) don't hesitate to incorporate their ideas.
Perhaps the most dramatic example is the merger of Rails and Merb, a rival
Ruby web framework, so that Rails now benefits from Merb's modular design,
stable API,
and improved performance.

Finally, Rails benefits from an unusually enthusiastic and diverse community.
The results include hundreds of open-source
contributors, well-attended
conferences, a huge number of
plugins and
gems (self-contained solutions to specific problems
such as pagination and image upload), a rich variety of informative blogs, and
a cornucopia of discussion forums and IRC channels. The large number of Rails
programmers also makes it easier to handle the inevitable application errors:
the "Google the error message" algorithm nearly always produces a relevant
blog post or discussion-forum thread.

### [1.1.1 Comments for various readers](beginning.html#sec-
comments_for_various_readers)

The _Rails Tutorial_ contains integrated tutorials not only for Rails, but
also for the underlying Ruby language, the RSpec testing framework,
HTML,
CSS, a small amount of
JavaScript, and even a little
SQL. This means that, no matter where you
currently are in your knowledge of web development, by the time you finish
this tutorial you will be ready for more advanced Rails resources, as well as
for the more systematic treatments of the other subjects mentioned. It also
means that there's a _lot_ of material to cover; if you don't already have
much experience programming computers, you might find it overwhelming. The
comments below contain some suggestions for approaching the _Rails Tutorial_
depending on your background.

**All readers:** One common question when learning Rails is whether to learn Ruby first. The answer depends on your personal learning style and how much programming experience you already have. If you prefer to learn everything systematically from the ground up, or if you have never programmed before, then learning Ruby first might work well for you, and in this case I recommend _Beginning Ruby_](http://www.amazon.com/gp/product/1430223634) by Peter Cooper. On the other hand, many beginning Rails developers are excited about making _web_ applications, and would rather not slog through a 500-page book on pure Ruby before ever writing a single web page. In this case, I recommend following the short interactive tutorial at Try Ruby,2 and then optionally do the free tutorial at [Rails for Zombies3 to get a taste of what Rails can do.

Another common question is whether to use tests from the start. As noted in
the introduction, the _Rails Tutorial_ uses test-driven development (also
called test-first development), which in my view is the best way to develop
Rails applications, but it does introduce a substantial amount of overhead and
complexity. If you find yourself getting bogged down by the tests, I suggest
either skipping them on a first reading or (even better) using them as a tool
to verify your code's correctness without worrying about how they work. This
latter strategy involves creating the necessary test files (called _specs_)
and filling them with the test code _exactly_ as it appears in the book. You
can then run the test suite (as described in [Chapter5
](filling-in-the-layout.html#top)) to watch it fail, then write the
application code as described in the tutorial, and finally re-run the test
suite to watch it pass.

**Inexperienced programmers:** The _Rails Tutorial_ is not aimed principally at beginning programmers, and web applications, even relatively simple ones, are by their nature fairly complex. If you are completely new to web programming and find the _Rails Tutorial_ too difficult, I suggest learning the basics of HTML and CSS and then giving the _Rails Tutorial_ another go. (Unfortunately, I don't have a personal recommendation here, but _Head First HTML_](http://headfirstlabs.com/books/hfhtml/) looks promising, and one reader recommends _CSS: The Missing Manual_](http://www.amazon.com/gp/product/0596526873) by David Sawyer McFarland.) You might also consider reading the first few chapters of [_Beginning Ruby_ by Peter Cooper, which starts with sample applications much smaller than a full-blown web app. That said, a surprising number of beginners have used this tutorial to learn web development, so I suggest giving it a try, and I especially recommend the [_Rails Tutorial_ screencast series4 to give you an "over-the-shoulder" look at Rails software development.   

**Experienced programmers new to web development:** Your previous experience means you probably already understand ideas like classes, methods, data structures, etc., which is a big advantage. Be warned that if your background is in C/C++ or Java, you may find Ruby a bit of an odd duck, and it might take time to get used to it; just stick with it and eventually you'll be fine. (Ruby even lets you put semicolons at the ends of lines if you miss them too much.) The _Rails Tutorial_ covers all the web-specific ideas you'll need, so don't worry if you don't currently know a `PUT` from a `POST`.  

**Experienced web developers new to Rails:** You have a great head start, especially if you have used a dynamic language such as PHP or (even better) Python. The basics of what we cover will likely be familiar, but test-driven development may be new to you, as may be the structured REST style favored by Rails. Ruby has its own idiosyncrasies, so those will likely be new, too.  

**Experienced Ruby programmers:** The set of Ruby programmers who don't know Rails is a small one nowadays, but if you are a member of this elite group you can fly through this book and then move on to _The Rails3 Way_ by Obie Fernandez.   

**Inexperienced Rails programmers:** You've perhaps read some other tutorials and made a few small Rails apps yourself. Based on reader feedback, I'm confident that you can still get a lot out of this book. Among other things, the techniques here may be more up-to-date than the ones you picked up when you originally learned Rails.   

**Experienced Rails programmers:** This book is unnecessary for you, but many experienced Rails developers have expressed surprise at how much they learned from this book, and you might enjoy seeing Rails from a different perspective.   

After finishing the _Ruby on Rails Tutorial_, I recommend that experienced
programmers read [_The Well-Grounded
Rubyist_](http://www.amazon.com/gp/product/1933988657) by
DavidA. Black, which is an excellent in-depth discussion of
Ruby from the ground up, or [_The Ruby
Way_](http://www.amazon.com/gp/product/0672328844) by Hal Fulton, which is
also fairly advanced but takes a more topical approach. Then move on to [_The
Rails3 Way_](http://www.amazon.com/gp/product/0321601661)
to deepen your Rails expertise.

At the end of this process, no matter where you started, you should be ready
for the many more intermediate-to-advanced Rails resources out there. Here are
some I particularly recommend:

  * RailsCasts free Rails screencasts
  * PeepCode: Excellent commercial screencasts
  * Code School: Interactive programming courses
  * Rails Guides: Good topical and up-to-date Rails references
  * RailsCasts](http://railscasts.com/) by Ryan Bates: Did I already mention RailsCasts? Seriously: [_RailsCasts_.

### 1.1.2 "Scaling" Rails

Before moving on with the rest of the introduction, I'd like to take a moment
to address the one issue that dogged the Rails framework the most in its early
days: the supposed inability of Rails to "scale"--i.e., to handle large
amounts of traffic. Part of this issue relied on a misconception; [you scale a
_site_, not a framework](http://idleprocess.wordpress.com/2009/11/24
/presentation-summary-high-performance-at-massive-scale-lessons-learned-at-
facebook/), and Rails, as awesome as it is, is only a framework. So the real
question should have been, "Can a site built with Rails scale?" In any case,
the question has now been definitively answered in the affirmative: some of
the most heavily trafficked sites in the world use Rails. Actually _doing_ the
scaling is beyond the scope of just Rails, but rest assured that if _your_
application ever needs to handle the load of Hulu or the Yellow Pages, Rails
won't stop you from taking over the world.

### 1.1.3 Conventions in this book

The conventions in this book are mostly self-explanatory. In this section,
I'll mention some that may not be.

Both the HTML and
PDF editions of this book are full of links, both
to internal sections (such as [Section1.2](beginning.html
#sec-up_and_running)) and to external sites (such as the main [Ruby on Rails
download](http://rubyonrails.org/download) page).5

Many examples in this book use command-line commands. For simplicity, all
command line examples use a Unix-style command line prompt (a dollar sign), as
follows:

    
    $ echo "hello, world"
    hello, world
    

Windows users should understand that their systems will use the analogous
angle prompt`>`:

    
    C:\Sites> echo "hello, world"
    hello, world
    

On Unix systems, some commands should be executed with `sudo`, which stands
for "substitute user do". By default, a command executed with `sudo` is run as
an administrative user, which has access to files and directories that normal
users can't touch, such as in this example from
Section1.2.2:

    
    $ sudo ruby setup.rb
    

Most Unix/Linux/OSX systems require `sudo` by default,
unless you are using Ruby Version Manager as suggested in
Section1.2.2.3; in this
case, you would type this instead:

    
    $ ruby setup.rb
    

Rails comes with lots of commands that can be run at the command line. For
example, in Section1.2.5
we'll run a local development web server as follows:

    
    $ rails server
    

As with the command-line prompt, the _Rails Tutorial_ uses the Unix convention
for directory separators (i.e., a forward slash`/`). My
Rails Tutorial sample application, for instance, lives in

    
    /Users/mhartl/rails_projects/sample_app
    

On Windows, the analogous directory would be

    
    C:\Sites\sample_app
    

The root directory for any given app is known as the _Rails root_, but this
terminology is confusing and many people mistakenly believe that the "Rails
root" is the root directory for Rails itself. For clarity, the _Rails
Tutorial_ will refer to the Rails root as the _application root_, and
henceforth all directories will be relative to this directory. For example,
the `config` directory of my sample application is

    
    /Users/mhartl/rails_projects/sample_app/config
    

The application root directory here is everything before `config`, i.e.,

    
    /Users/mhartl/rails_projects/sample_app
    

For brevity, when referring to the file

    
    /Users/mhartl/rails_projects/sample_app/config/routes.rb
    

I'll omit the application root and simply write `config/routes.rb`.

The _Rails Tutorial_ often shows output from various programs (shell commands,
version control status, Ruby programs, etc.). Because of the innumerable small
differences between different computer systems, the output you see may not
always agree exactly with what is shown in the text, but this is not cause for
concern.

Some commands may produce errors depending on your system; rather than attempt
the Sisyphean task of documenting all
such errors in this tutorial, I will delegate to the "Google the error
message" algorithm, which among other things is good practice for real-life
software development. If you run into any problems while following the
tutorial, I suggest consulting the resources listed on the [Rails Tutorial
help page](http://railstutorial.org/help).6

## 1.2 Up and running

> I think of Chapter 1 as the "weeding out phase" in law school--if you can
get your dev environment set up, the rest is easy to get through.

--Bob Cavezza, _Rails Tutorial_ reader

It's time now to get going with a Ruby on Rails development environment and
our first application. There is quite a bit of overhead here, especially if
you don't have extensive programming experience, so don't get discouraged if
it takes a while to get started. It's not just you; every developer goes
through it (often more than once), but rest assured that the effort will be
richly rewarded.

### 1.2.1 Development environments

Considering various idiosyncratic customizations, there are probably as many
development environments as there are Rails programmers, but there are at
least two broad types: text editor/command line environments, and integrated
development environments (IDEs). Let's consider the latter first.

#### IDEs

There is no shortage of Rails IDEs, including
RadRails,
RubyMine, and [3rd
Rail](http://www.codegear.com/products/3rdrail). I've heard especially good
things about RubyMine, and one reader (David Loeffler) has assembled [notes on
how to use RubyMine with this
tutorial](https://github.com/perfectionist/sample_project/wiki).7 If you're
comfortable using an IDE, I suggest taking a look at the options mentioned to
see what fits with the way you work.

#### Text editors and command lines

Instead of using an IDE, I prefer to use a _text editor_ to edit text, and a
_command line_ to issue commands
(Figure1.1. Which
combination you use depends on your tastes and your platform.

  * **Text editor:** I recommend Sublime Text2](http://www.sublimetext.com/2), an outstanding cross-platform text editor that is in beta as of this writing but has already proven to be exceptionally powerful. Sublime Text is heavily influenced by TextMate A second excellent choice is [Vim,8 versions of which are available for all major platforms. Sublime Text can be obtained commercially, whereas Vim can be obtained at no cost; both are industrial-strength editors, but in my experience Sublime Text is _much_ more accessible to beginners. 
  * **Terminal:** On OSX, I recommend either use iTerm](http://iterm.sourceforge.net/) or the native Terminal app. On Linux, the default terminal is fine. On Windows, many users prefer to develop Rails applications in a virtual machine running Linux, in which case your command-line options reduce to the previous case. If developing within Windows itself, I recommend using the command prompt that comes with Rails Installer ([Section1.2.2.1. 

If you decide to use Sublime Text, you want to follow the setup instructions
for [Rails Tutorial Sublime
Text](https://github.com/mhartl/rails_tutorial_sublime_text).9 _Note_: Such
configuration settings are fiddly and error-prone, so this step should only be
attempted by advanced users.

!editor_shell

Figure 1.1: A text editor/command line development environment
(TextMate/iTerm). [(full size)](http://railstutorial.org/images/figures
/editor_shell-full.png)

#### Browsers

Although there are many web browsers to choose from, the vast majority of
Rails programmers use Firefox, Safari, or Chrome when developing. The
screenshots in Rails Tutorial will generally be of a Firefox browser. If you
use Firefox, I suggest using the Firebug add-on,
which lets you perform all sorts of magic, such as dynamically inspecting (and
even editing) the HTML structure and CSS rules on any page. For those not
using Firefox, both Safari and Chrome have a built-in "Inspect element"
feature available by right-clicking on any part of the page.

#### A note about tools

In the process of getting your development environment up and running, you may
find that you spend a _lot_ of time getting everything just right. The
learning process for editors and IDEs is particularly long; you can spend
weeks on Sublime Text or Vim tutorials alone. If you're new to this game, I
want to assure you that _spending time learning tools is normal_. Everyone
goes through it. Sometimes it is frustrating, and it's easy to get impatient
when you have an awesome web app in your head and you _just want to learn
Rails already_, but have to spend a week learning some weird ancient Unix
editor just to get started. But a craftsman has to know his tools, and in the
end the reward is worth the effort.

### 1.2.2 Ruby, RubyGems, Rails, and Git

> Practically all the software in the world is either broken or very difficult
to use. So users dread software. They've been trained that whenever they try
to install something, or even fill out a form online, it's not going to work.
_I_ dread installing stuff, and I have a Ph.D. in computer science.

--Paul Graham, _Founders at Work_

Now it's time to install Ruby and Rails. I've done my best to cover as many
bases as possible, but systems vary, and many things can go wrong during these
steps. Be sure to Google the error message or consult the [Rails Tutorial help
page](http://railstutorial.org/help) if you run into trouble.

**Unless otherwise noted, you should use the exact versions of all software used in the tutorial, including Rails itself, if you want the same results.** Sometimes minor version differences will yield identical results, but you shouldn't count on this, especially with respect to Rails versions. The main exception is Ruby itself: 1.9.2 and 1.9.3 are virtually identical for the purposes of this tutorial, so feel free to use either one.

#### Rails Installer (Windows)

Installing Rails on Windows used to be a real pain, but thanks to the efforts
of the good people at Engine Yard--especially
Dr.Nic Williams and WayneE. Seguin--
installing Rails and related software on Windows is now easy. If you are using
Windows, go to Rails Installer and download the
Rails Installer executable and view the excellent installation video. Double-
click the executable and follow the instructions to install Git (so you can
skip Section1.2.2.2,
Ruby (skip [Section1.2.2.3](beginning.html#sec-
install_ruby)), RubyGems (skip
Section1.2.2.4, and
Rails itself (skip [Section1.2.2.5](beginning.html#sec-
install_rails)). Once the installation has finished, you can skip right to the
creation of the first application in
Section1.2.3.

Bear in mind that the Rails Installer might use a slightly different version
of Rails from the one installed in
Section1.2.2.5, which
might cause incompatibilities. To fix this, I am currently working with Nic
and Wayne to create a list of Rails Installers ordered by Rails version
number.

#### Install Git

Much of the Rails ecosystem depends in one way or another on a [version
control system](http://en.wikipedia.org/wiki/Revision_control) called
Git (covered in more detail in
Section1.3. Because
its use is ubiquitous, you should install Git even at this early stage; I
suggest following the installation instructions for your platform at the
[Installing Git section of _Pro Git_](http://www.git-scm.com/book/en/Getting-
Started-Installing-Git).

#### Install Ruby

The next step is to install Ruby. It's possible that your system already has
it; try running

    
    $ ruby -v
    ruby 1.9.3
    

to see the version number. Rails3 requires
Ruby1.8.7 or later and works best with
Ruby1.9.x. This tutorial assumes that most readers are
using Ruby1.9.2 or 1.9.3, but Ruby1.8.7
should work as well (although there is one syntax difference, covered in
Chapter4, and assorted
minor differences in output).

As part of installing Ruby, if you are using OSX or Linux I
strongly recommend using Ruby Version Manager (RVM), which
allows you to install and manage multiple versions of Ruby on the same
machine. (The Pik project accomplishes a
similar feat on Windows.) This is particularly important if you want to run
different versions of Ruby or Rails on the same machine. If you run into any
problems with RVM, you can often find its creator, WayneE.
Seguin, on the RVM IRC channel ([#rvm on
freenode.net](http://webchat.freenode.net/?channels=rvm)).10 If you are
running Linux, I particularly recommend [How to install Ruby on Rails in
Ubuntu on the Sudobits Blog](http://blog.sudobits.com/2012/05/02/how-to-
install-ruby-on-rails-in-ubuntu-12-04-lts/).

After installing RVM, you can install Ruby as
follows:11

    
    $ rvm get head && rvm reload
    $ rvm install 1.9.3
    <wait a while>
    

Here the first command updates and reloads RVM itself, which is a good
practice since RVM gets updated frequently. The second installs the 1.9.3
version of Ruby; depending on your system, they might take a while to download
and compile, so don't worry if it seems to be taking forever.

Some OSX users have trouble with the lack of an `autoconf`
executable, which you can fix by installing
Homebrew12 (a package management system
for OSX) and then running

    
    $ brew install automake
    $ rvm install 1.9.3
    

Some Linux users report having to include the path to a library called
OpenSSL:

    
    $ rvm install 1.9.3 --with-openssl-dir=$HOME/.rvm/
    

On some older OSX systems, you might have to include the
path to the readline library:

    
    $ rvm install 1.9.3 --with-readline-dir=/opt/local
    

(Like I said, lots of things can go wrong. The only solution is web searches
and determination.)

After installing Ruby, you should configure your system for the other software
needed to run Rails applications. This typically involves installing _gems_,
which are self-contained packages of Ruby code. Since gems with different
version numbers sometimes conflict, it is often convenient to create separate
_gemsets_, which are self-contained bundles of gems. For the purposes of this
tutorial, I suggest creating a gemset called `rails3tutorial2ndEd`:

    
    $ rvm use 1.9.3@rails3tutorial2ndEd --create --default
    Using /Users/mhartl/.rvm/gems/ruby-1.9.3 with gemset rails3tutorial2ndEd
    

This command creates (`--create`) the gemset `rails3tutorial2ndEd` associated
with Ruby1.9.3 while arranging to start using it
immediately (`use`) and setting it as the default (`--default`) gemset, so
that any time we open a new terminal window the `1.9.3@rails3tutorial2ndEd`
Ruby/gemset combination is automatically selected. RVM supports a large
variety of commands for manipulating gemsets; see the documentation at
http://rvm.beginrescueend.com/gemsets/. If you ever
get stuck with RVM, running commands like these should help you get your
bearings:

    
    $ rvm --help
    $ rvm gemset --help
    

#### Install RubyGems

RubyGems is a package manager for Ruby projects, and there are many useful
libraries (including Rails) available as Ruby packages, or _gems_. Installing
RubyGems should be easy once you install Ruby. In fact, if you have [installed
RVM](http://rvm.io/rvm/install/), you already have RubyGems, since RVM
includes it automatically:

    
    $ which gem
    /Users/mhartl/.rvm/rubies/ruby-1.9.3-p0/bin/gem
    

If you don't already have it, you should [download
RubyGems](http://rubyforge.org/frs/?group_id=126), extract it, and then go to
the `rubygems` directory and run the setup program:

    
    $ ruby setup.rb
    

(If you get a permissions error here, recall from
Section1.1.3 that you may
have to use `sudo`.)

If you already have RubyGems installed, you should make sure your system uses
the version used in this tutorial:

    
    $ gem update --system 1.8.24
    

Freezing your system to this particular version will help prevent conflicts as
RubyGems changes in the future.

When installing gems, by default RubyGems generates two different kinds of
documentation (called ri and rdoc), but many Ruby and Rails developers find
that the time to build them isn't worth the benefit. (Many programmers rely on
online documentation instead of the native ri and rdoc documents.) To prevent
the automatic generation of the documentation, I recommend making a gem
configuration file called `.gemrc` in your home directory as in
Listing1.1 with the
line in Listing1.2. (The
tilde"`~`" means "home directory", while the
dot`.` in `.gemrc` makes the file hidden, which is a common
convention for configuration files. )

Listing 1.1. Creating a gem configuration file.

    
    $ subl ~/.gemrc
    

Here `subl` is the command-line command to launch Sublime Text on
OSX, which you can set up using the [Sublime
Text2 documentation for the OSX command
line](http://www.sublimetext.com/docs/2/osx_command_line.html). If you're on a
different platform, or if you're using a different editor, you should replace
this command as necessary (i.e., by double-clicking the application icon or by
using an alternate command such as `mate`, `vim`, `gvim`, or `mvim`). For
brevity, throughout the rest of this tutorial I'll use `subl` as a shorthand
for "open with your favorite text editor."

Listing 1.2. Suppressing the ri and rdoc documentation in `.gemrc`.

    
    install: --no-rdoc --no-ri
    update: --no-rdoc --no-ri
    

#### Install Rails

Once you've installed RubyGems, installing Rails should be easy. This tutorial
standardizes on Rails3.2, which we can install as follows:

    
    $ gem install rails -v 3.2.8
    

To check your Rails installation, run the following command to print out the
version number:

    
    $ rails -v
    Rails 3.2.8
    

_Note:_ If you installed Rails using the Rails Installer in
[Section1.2.2.1](beginning.html#sec-
rails_installer_windows), there might be slight version differences. As of
this writing, those differences are not relevant, but in the future, as the
current Rails version diverges from the one used in this tutorial, these
differences may become significant. I am currently working with Engine Yard to
create links to specific versions of the Rails Installer.

If you're running Linux, you might have to install a couple of other packages
at this point:

    
    $ sudo apt-get install libxslt-dev libxml2-dev libsqlite3-dev # Linux only
    

### 1.2.3 The first application

Virtually all Rails applications start the same way, with the `rails` command.
This handy program creates a skeleton Rails application in a directory of your
choice. To get started, make a directory for your Rails projects and then run
the `rails` command to make the first application
(Listing1.3:

Listing 1.3. Running `rails` to generate a new application.

    
    $ mkdir rails_projects
    $ cd rails_projects
    $ rails new first_app
          create  
          create  README.rdoc
          create  Rakefile
          create  config.ru
          create  .gitignore
          create  Gemfile
          create  app
          create  app/assets/images/rails.png
          create  app/assets/javascripts/application.js
          create  app/assets/stylesheets/application.css
          create  app/controllers/application_controller.rb
          create  app/helpers/application_helper.rb
          create  app/mailers
          create  app/models
          create  app/views/layouts/application.html.erb
          create  app/mailers/.gitkeep
          create  app/models/.gitkeep
          create  config
          create  config/routes.rb
          create  config/application.rb
          create  config/environment.rb
          .
          .
          .
          create  vendor/plugins
          create  vendor/plugins/.gitkeep
             run  bundle install
    Fetching source index for https://rubygems.org/
    .
    .
    .
    Your bundle is complete! Use `bundle show [gemname]` to see where a bundled
    gem is installed.
    

As seen at the end of [Listing1.3](beginning.html#code-
rails_command), running `rails` automatically runs the `bundle install`
command after the file creation is done. If that step doesn't work right now,
don't worry; follow the steps in
Section1.2.4 and you should
be able to get it to work.

Notice how many files and directories the `rails` command creates. This
standard directory and file structure
([Figure1.2](beginning.html#fig-
directory_structure_rails_3_2)) is one of the many advantages of Rails; it
immediately gets you from zero to a functional (if minimal) application.
Moreover, since the structure is common to all Rails apps, you can immediately
get your bearings when looking at someone else's code. A summary of the
default Rails files appears in [Table1.1](beginning.html
#table-rails_directory_structure); we'll learn about most of these files and
directories throughout the rest of this book. In particular, starting in
[Section5.2.1](filling-in-the-layout.html#sec-
the_asset_pipeline) we'll discuss the `app/assets` directory, part of the
_asset pipeline_ (new as of Rails3.1) that makes it easier
than ever to organize and deploy assets such as cascading style sheets and
JavaScript files.

![directory_structure_rails_31](images/figures/directory_structure_rails_31.pn
g)

Figure 1.2: The directory structure for a newly hatched Rails
app.[(full size)](http://railstutorial.org/images/figures/d
irectory_structure_rails_31-full.png)

**File/Directory****Purpose**

`app/`

Core application (app) code, including models, views, controllers, and helpers

`app/assets`

Applications assets such as cascading style sheets (CSS), JavaScript files,
and images

`config/`

Application configuration

`db/`

Database files

`doc/`

Documentation for the application

`lib/`

Library modules

`lib/assets`

Library assets such as cascading style sheets (CSS), JavaScript files, and
images

`log/`

Application log files

`public/`

Data accessible to the public (e.g., web browsers), such as error pages

`script/rails`

A script for generating code, opening console sessions, or starting a local
server

`test/`

Application tests (made obsolete by the `spec/` directory in
[Section3.1.2](static-pages.html#sec-
static_pages_with_rails))

`tmp/`

Temporary files

`vendor/`

Third-party code such as plugins and gems

`vendor/assets`

Third-party assets such as cascading style sheets (CSS), JavaScript files, and
images

`README.rdoc`

A brief description of the application

`Rakefile`

Utility tasks available via the `rake` command

`Gemfile`

Gem requirements for this app

`Gemfile.lock`

A list of gems used to ensure that all copies of the app use the same gem
versions

`config.ru`

A configuration file for Rack middleware

`.gitignore`

Patterns for files that should be ignored by Git

Table 1.1: A summary of the default Rails directory structure.

### 1.2.4 Bundler

After creating a new Rails application, the next step is to use _Bundler_ to
install and include the gems needed by the app. As noted briefly in
Section1.2.3,
Bundler is run automatically (via `bundle install`) by the `rails` command,
but in this section we'll make some changes to the default application gems
and run Bundler again. This involves opening the `Gemfile` with your favorite
text editor:

    
    $ cd first_app/
    $ subl Gemfile
    

The result should look something like
Listing1.4. The code
in this file is Ruby, but don't worry at this point about the syntax;
Chapter4 will cover Ruby in
more depth.

Listing 1.4. The default `Gemfile` in the `first_app` directory.

    
    source 'https://rubygems.org'
    
    gem 'rails', '3.2.8'
    
    # Bundle edge Rails instead:
    # gem 'rails', :git => 'git://github.com/rails/rails.git'
    
    gem 'sqlite3'
    
    
    # Gems used only for assets and not required
    # in production environments by default.
    group :assets do
      gem 'sass-rails',   '~> 3.2.3'
      gem 'coffee-rails', '~> 3.2.2'
    
      gem 'uglifier', '>= 1.2.3'
    end
    
    gem 'jquery-rails'
    
    # To use ActiveModel has_secure_password
    # gem 'bcrypt-ruby', '~> 3.0.0'
    
    # To use Jbuilder templates for JSON
    # gem 'jbuilder'
    
    # Use unicorn as the web server
    # gem 'unicorn'
    
    # Deploy with Capistrano
    # gem 'capistrano'
    
    # To use debugger
    # gem 'ruby-debug19', :require => 'ruby-debug'
    

Many of these lines are commented out with the hash
symbol`#`; they are there to show you some commonly needed
gems and to give examples of the Bundler syntax. For now, we won't need any
gems other than the defaults: Rails itself, some gems related to the asset
pipeline ([Section5.2.1](filling-in-the-layout.html#sec-
the_asset_pipeline)), the gem for the jQuery JavaScript library, and the gem
for the Ruby interface to the SQLite database.

Unless you specify a version number to the `gem` command, Bundler will
automatically install the latest version of the gem. Unfortunately, gem
updates often cause minor but potentially confusing breakage, so in this
tutorial we'll include explicit version numbers known to work, as seen in
Listing1.5
(which also omits the commented-out lines from
Listing1.4.

Listing 1.5. A `Gemfile` with an explicit version of each Ruby gem.

    
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
    

Listing1.5
changes the line for jQuery, the default JavaScript library used by Rails,
from

    
    gem 'jquery-rails'
    

to

    
    gem 'jquery-rails', '2.0.2'
    

We've also changed

    
    gem 'sqlite3'
    

to

    
    group :development do
      gem 'sqlite3', '1.3.5'
    end
    

which forces Bundler to install version `1.3.5` of the `sqlite3` gem. Note
that we've also taken this opportunity to arrange for SQLite to be included
only in a development environment ([Section7.1.1](sign-
up.html#sec-rails_environments)), which prevents potential conflicts with the
database used by Heroku ([Section1.4](beginning.html#sec-
deploying)).

Listing1.5
also changes a few other lines, converting

    
    group :assets do
      gem 'sass-rails',   '~> 3.2.3'
      gem 'coffee-rails', '~> 3.2.2'
      gem 'uglifier', '>= 1.2.3'
    end
    

to

    
    group :assets do
      gem 'sass-rails',   '3.2.5'
      gem 'coffee-rails', '3.2.2'
      gem 'uglifier', '1.2.3'
    end
    

The syntax

    
    gem 'uglifier', '>= 1.2.3'
    

installs the latest version of the `uglifier` gem (which handles file
compression for the asset pipeline) as long as it's greater than
version`1.2.3`--even if it's, say,
version`7.2`. Meanwhile, the code

    
    gem 'coffee-rails', '~> 3.2.2'
    

installs the gem `coffee-rails` (also needed by the asset pipeline) as long as
it's lower than version`3.3`. In other words, the `>=`
notation always performs upgrades, whereas the `~> 3.2.2` notation only
performs upgrades to minor point releases (e.g., from `3.1.1` to `3.1.2`), but
not to major point releases (e.g., from `3.1` to `3.2`). Unfortunately,
experience shows that even minor point releases often break things, so for the
_Rails Tutorial_ we'll err on the side of caution by including exact version
numbers for virtually all gems. (The only exception is gems that are in
release candidate or beta stage as of this writing; for those gems, we'll use
`~>` so that the final versions will be loaded once they're done.)

Once you've assembled the proper `Gemfile`, install the gems using `bundle
install`:

    
    $ bundle install
    Fetching source index for https://rubygems.org/
    .
    .
    .
    

If you're running OSX and you get an error about missing
Ruby header files (e.g., `ruby.h`) at this point, you may need to install
Xcode. These are developer tools that came with your OSX
installation disk, but to avoid the full installation I recommend the much
smaller [Command Line Tools for
Xcode](https://developer.apple.com/downloads/).13 If you get a libxslt error
when installing the Nokogiri gem, try reinstalling Ruby:

    
    $ rvm reinstall 1.9.3
    $ bundle install
    

The `bundle install` command might take a few moments, but when it's done our
application will be ready to run. _Note:_ This setup is fine for the first
app, but it isn't ideal. Chapter3
covers a more powerful (and slightly more advanced) method for installing Ruby
gems with Bundler.

### 1.2.5 `rails server`

Thanks to running `rails new` in
Section1.2.3
and `bundle install` in [Section1.2.4](beginning.html#sec-
bundler), we already have an application we can run--but how? Happily, Rails
comes with a command-line program, or _script_, that runs a _local_ web
server, visible only from your development machine:14

    
    $ rails server
    => Booting WEBrick
    => Rails application starting on http://0.0.0.0:3000
    => Call with -d to detach
    => Ctrl-C to shutdown server
    

(If your system complains about the lack of a JavaScript runtime, visit the
execjs page at GitHub for a list of
possibilities. I particularly recommend installing
Node.js This tells us that the application is running
on port number 300015 at the
address `0.0.0.0`. This address tells the computer to listen on every
available IP address configured on that specific machine; in particular, we
can view the application using the special address `127.0.0.1`, which is also
known as `localhost`. We can see the result of visiting
http://localhost:3000/ in
Figure1.3.

!riding_rails_31

Figure 1.3: The default Rails page.[(full
size)](http://railstutorial.org/images/figures/riding_rails_31-full.png)

To see information about our first application, click on the link "About your
application's environment". The result is shown in
[Figure1.4](beginning.html#fig-
riding_rails_32_environment). ([Figure1.4](beginning.html
#fig-riding_rails_32_environment) represents the environment on my machine
when I made the screenshot; your results may differ.)

!riding_rails_32_environment

Figure 1.4: The default page with the app
environment.[(full
size)](http://railstutorial.org/images/figures/riding_rails_32_environment-
full.png)

Of course, we don't need the default Rails page in the long run, but it's nice
to see it working for now. We'll remove the default page (and replace it with
a custom home page) in [Section5.3.2](filling-in-the-
layout.html#sec-rails_routes).

### 1.2.6 Model-view-controller (MVC)

Even at this early stage, it's helpful to get a high-level overview of how
Rails applications work ([Figure1.5](beginning.html#fig-
MVC)). You might have noticed that the standard Rails application structure
([Figure1.2](beginning.html#fig-
directory_structure_rails_3_2)) has an application directory called `app/`
with three subdirectories: `models`, `views`, and `controllers`. This is a
hint that Rails follows the [model-view-
controller](http://en.wikipedia.org/wiki/Model-view-controller) (MVC)
architectural pattern, which enforces a separation between "domain logic"
(also called "business logic") from the input and presentation logic
associated with a graphical user interface (GUI). In the case of web
applications, the "domain logic" typically consists of data models for things
like users, articles, and products, and the GUI is just a web page in a web
browser.

When interacting with a Rails application, a browser sends a _request_, which
is received by a web server and passed on to a Rails _controller_, which is in
charge of what to do next. In some cases, the controller will immediately
render a _view_, which is a template that gets converted to HTML and sent back
to the browser. More commonly for dynamic sites, the controller interacts with
a _model_, which is a Ruby object that represents an element of the site (such
as a user) and is in charge of communicating with the database. After invoking
the model, the controller then renders the view and returns the complete web
page to the browser as HTML.

!mvc_schematic

Figure 1.5: A schematic representation of the model-view-controller (MVC)
architecture.

If this discussion seems a bit abstract right now, worry not; we'll refer back
to this section frequently. In addition, [Section2.2.2](a
-demo-app.html#sec-mvc_in_action) has a more detailed discussion of MVC in the
context of the demo app. Finally, the sample app will use all aspects of MVC;
we'll cover controllers and views starting in [Section3.1.2
](static-pages.html#sec-static_pages_with_rails), models starting in
Section6.1, and we'll
see all three working together in [Section7.1.2](sign-
up.html#sec-a_users_resource).

## 1.3 Version control with Git

Now that we have a fresh and working Rails application, we'll take a moment
for a step that, while technically optional, would be viewed by many Rails
developers as practically essential, namely, placing our application source
code under _version control_. Version control systems allow us to track
changes to our project's code, collaborate more easily, and roll back any
inadvertent errors (such as accidentally deleting files). Knowing how to use a
version control system is a required skill for every software developer.

There are many options for version control, but the Rails community has
largely standardized on Git, a distributed version
control system originally developed by Linus Torvalds to host the Linux
kernel. Git is a large subject, and we'll only be scratching the surface in
this book, but there are many good free resources online; I especially
recommend _Pro Git_.
Putting your source code under version control with Git is _strongly_
recommended, not only because it's nearly a universal practice in the Rails
world, but also because it will allow you to share your code more easily
(Section1.3.4 and deploy your
application right here in the first chapter
(Section1.4.

### 1.3.1 Installation and setup

The first step is to install Git if you haven't yet followed the steps in
Section1.2.2.2. (As noted
in that section, this involves following the instructions in the [Installing
Git section of _Pro Git_](http://progit.org/book/ch1-4.html).)

#### First-time system setup

After installing Git, you should perform a set of one-time setup steps. These
are _system_ setups, meaning you only have to do them once per computer:

    
    $ git config --global user.name "Your Name"
    $ git config --global user.email your.email@example.com
    

I also like to use `co` in place of the more verbose `checkout` command, which
we can arrange as follows:

    
    $ git config --global alias.co checkout
    

This tutorial will usually use the full `checkout` command, which works for
systems that don't have `co` configured, but in real life I nearly always use
`git co`.

As a final setup step, you can optionally set the editor Git will use for
commit messages. If you use a graphical editor such as Sublime Text, TextMate,
gVim, or MacVim, you need to use a flag to make sure that the editor stays
attached to the shell instead of detaching immediately:16

    
    $ git config --global core.editor "subl -w"
    

Replace `"subl -w"` with `"mate -w"` for TextMate, `"gvim -f"` for gVim, or
`"mvim -f"` for MacVim.

#### First-time repository setup

Now we come to some steps that are necessary each time you create a new
_repository_. First navigate to the root directory of the first app and
initialize a new repository:

    
    $ git init
    Initialized empty Git repository in /Users/mhartl/rails_projects/first_app/.git/
    

The next step is to add the project files to the repository. There's a minor
complication, though: by default Git tracks the changes of _all_ the files,
but there are some files we don't want to track. For example, Rails creates
log files to record the behavior of the application; these files change
frequently, and we don't want our version control system to have to update
them constantly. Git has a simple mechanism to ignore such files: simply
include a file called `.gitignore` in the application root directory with some
rules telling Git which files to ignore.17

Looking again at [Table1.1](beginning.html#table-
rails_directory_structure), we see that the `rails` command creates a default
`.gitignore` file in the application root directory, as shown in
Listing1.6.

Listing 1.6. The default `.gitignore` created by the `rails` command.

    
    # See http://help.github.com/ignore-files/ for more about ignoring files.
    #
    # If you find yourself ignoring temporary files generated by your text editor
    # or operating system, you probably want to add a global ignore instead:
    #   git config --global core.excludesfile ~/.gitignore_global
    
    # Ignore bundler config
    /.bundle
    
    # Ignore the default SQLite database.
    /db/*.sqlite3
    
    # Ignore all logfiles and tempfiles.
    /log/*.log
    /tmp
    

Listing1.6 causes
Git to ignore files such as log files, Rails temporary (`tmp`) files, and
SQLite databases. (For example, to ignore log files, which live in the `log/`
directory, we use `log/*.log` to ignore all files that end in `.log`.) Most of
these ignored files change frequently and automatically, so including them
under version control is inconvenient; moreover, when collaborating with
others they can cause frustrating and irrelevant conflicts.

The `.gitignore` file in [Listing1.6](beginning.html#code-
default_gitignore) is probably sufficient for this tutorial, but depending on
your system you may find [Listing1.7](beginning.html#code-
gitignore) more convenient. This augmented `.gitignore` arranges to ignore
Rails documentation files, Vim and Emacs swap files, and (for
OSX users) the weird `.DS_Store` directories created by the
Mac Finder application. If you want to use this broader set of ignored files,
open up `.gitignore` in your favorite text editor and fill it with the
contents of Listing1.7.

Listing 1.7. An augmented `.gitignore` file.

    
    # Ignore bundler config
    /.bundle
    
    # Ignore the default SQLite database.
    /db/*.sqlite3
    
    # Ignore all logfiles and tempfiles.
    /log/*.log
    /tmp
    
    # Ignore other unneeded files.
    doc/
    *.swp
    *~
    .project
    .DS_Store
    .idea
    

### 1.3.2 Adding and committing

Finally, we'll add the files in your new Rails project to Git and then commit
the results. You can add all the files (apart from those that match the ignore
patterns in `.gitignore`) as follows:

    
    $ git add .
    

Here the dot '`.`' represents the current directory, and Git is smart enough
to add the files _recursively_, so it automatically includes all the
subdirectories. This command adds the project files to a _staging area_, which
contains pending changes to your project; you can see which files are in the
staging area using the `status` command:18

    
    $ git status
    # On branch master
    #
    # Initial commit
    #
    # Changes to be committed:
    #   (use "git rm --cached <file>..." to unstage)
    #
    #       new file:   README.rdoc
    #       new file:   Rakefile
    .
    .
    .
    

(The results are long, so I've used vertical dots to indicate omitted output.)

To tell Git you want to keep the changes, use the `commit` command:

    
    $ git commit -m "Initial commit"
    [master (root-commit) df0a62f] Initial commit
    42 files changed, 8461 insertions(+), 0 deletions(-)
    create mode 100644 README.rdoc
    create mode 100644 Rakefile
    .
    .
    .
    

The `-m` flag lets you add a message for the commit; if you omit `-m`, Git
will open the editor you set in
Section1.3.1 and have you
enter the message there.

It is important to note that Git commits are _local_, recorded only on the
machine on which the commits occur. This is in contrast to the popular open-
source version control system called Subversion, in which a commit necessarily
makes changes on a remote repository. Git divides a Subversion-style commit
into its two logical pieces: a local recording of the changes (`git commit`)
and a push of the changes up to a remote repository (`git push`). We'll see an
example of the push step in [Section1.3.5](beginning.html
#sec-git_commands).

By the way, you can see a list of your commit messages using the `log`
command:

    
    $ git log
    commit df0a62f3f091e53ffa799309b3e32c27b0b38eb4
    Author: Michael Hartl <michael@michaelhartl.com>
    Date:   Thu Oct 15 11:36:21 2009 -0700
    
      Initial commit
    

To exit `git log`, you may have to type `q` to quit.

### 1.3.3 What good does Git do you?

It's probably not entirely clear at this point why putting your source under
version control does you any good, so let me give just one example. (We'll see
many others in the chapters ahead.) Suppose you've made some accidental
changes, such as (D'oh!) deleting the critical `app/controllers/` directory:

    
    $ ls app/controllers/
    application_controller.rb
    $ rm -rf app/controllers/
    $ ls app/controllers/
    ls: app/controllers/: No such file or directory
    

Here we're using the Unix `ls` command to list the contents of the
`app/controllers/` directory and the `rm` command to remove it. The `-rf` flag
means "recursive force", which recursively removes all files, directories,
subdirectories, and so on, without asking for explicit confirmation of each
deletion.

Let's check the status to see what's up:

    
    $ git status
    # On branch master
    # Changed but not updated:
    #   (use "git add/rm <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #       deleted:    app/controllers/application_controller.rb
    #
    no changes added to commit (use "git add" and/or "git commit -a")
    

We see here that a file has been deleted, but the changes are only on the
"working tree"; they haven't been committed yet. This means we can still undo
the changes easily by having Git check out the previous commit with the
`checkout` command (and a `-f` flag to force overwriting the current changes):

    
    $ git checkout -f
    $ git status
    # On branch master
    nothing to commit (working directory clean)
    $ ls app/controllers/
    application_controller.rb
    

The missing directory and file are back. That's a relief!

### 1.3.4 GitHub

Now that you've put your project under version control with Git, it's time to
push your code up to GitHub, a social code site optimized
for hosting and sharing Git repositories. Putting a copy of your Git
repository at GitHub serves two purposes: it's a full backup of your code
(including the full history of commits), and it makes any future collaboration
much easier. This step is optional, but being a GitHub member will open the
door to participating in a wide variety of open-source projects.

!create_first_repository_new

Figure 1.6: Creating the first app repository at
GitHub.[(full
size)](http://railstutorial.org/images/figures/create_first_repository_new-
full.png)

GitHub has a variety of paid plans, but for open-source code their services
are free, so sign up for a [free GitHub
account](https://github.com/signup/free) if you don't have one already. (You
might have to follow the [GitHub tutorial on creating SSH
keys](http://help.github.com/key-setup-redirect) first.) After signing up,
click on the link to create a repository and fill in
the information as in [Figure1.6](beginning.html#fig-
create_first_repository). (Take care _not_ to initialize the repository with a
`README` file, as `rails new` creates one of those automatically.) After
submitting the form, push up your first application as follows:

    
    $ git remote add origin git@github.com:<username>/first_app.git
    $ git push -u origin master
    

These commands tell Git that you want to add GitHub as the origin for your
main (_master_) branch and then push your repository up to GitHub. (Don't
worry about what the `-u` flag does; if you're curious, do a web search for
"git set upstream".) Of course, you should replace `<username>` with your
actual username. For example, the command I ran for the `railstutorial` user
was

    
    $ git remote add origin git@github.com:railstutorial/first_app.git
    

The result is a page at GitHub for the first application repository, with file
browsing, full commit history, and lots of other goodies
(Figure1.7.

!github_repository_page

Figure 1.7: A GitHub repository page.[(full
size)](http://railstutorial.org/images/figures/github_repository_page-
full.png)

GitHub also has native applications to augment the command-line interface, so
if you're more comfortable with GUI apps you might want to check out [GitHub
for Windows](http://windows.github.com/) or [GitHub for
Mac](http://mac.github.com/). (GitHub for Linux is still just Git, it seems.)

### 1.3.5 Branch, edit, commit, merge

If you've followed the steps in
Section1.3.4, you might notice
that GitHub automatically shows the contents of the `README` file on the main
repository page. In our case, since the project is a Rails application
generated using the `rails` command, the `README` file is the one that comes
with Rails (Figure1.8.
Because of the `.rdoc` extension on the file, GitHub ensures that it is
formatted nicely, but the contents aren't helpful at all, so in this section
we'll make our first edit by changing the `README` to describe our project
rather than the Rails framework itself. In the process, we'll see a first
example of the branch, edit, commit, merge workflow that I recommend using
with Git.

!rails_readme_3_2

Figure 1.8: The initial (rather useless) `README` file for our project at
GitHub.[(full
size)](http://railstutorial.org/images/figures/rails_readme_3_2-full.png)

#### Branch

Git is incredibly good at making _branches_, which are effectively copies of a
repository where we can make (possibly experimental) changes without modifying
the parent files. In most cases, the parent repository is the _master_ branch,
and we can create a new topic branch by using `checkout` with the `-b` flag:

    
    $ git checkout -b modify-README
    Switched to a new branch 'modify-README'
    $ git branch
    master
    * modify-README
    

Here the second command, `git branch`, just lists all the local branches, and
the asterisk`*` identifies which branch we're currently on.
Note that `git checkout -b modify-README` both creates a new branch and
switches to it, as indicated by the asterisk in front of the `modify-README`
branch. (If you set up the `co` alias in
Section1.3, you can
use `git co -b modify-README` instead.)

The full value of branching only becomes clear when working on a project with
multiple developers,19 but branches are helpful even for a single-developer
tutorial such as this one. In particular, the master branch is insulated from
any changes we make to the topic branch, so even if we _really_ screw things
up we can always abandon the changes by checking out the master branch and
deleting the topic branch. We'll see how to do this at the end of the section.

By the way, for a change as small as this one I wouldn't normally bother with
a new branch, but it's never too early to start practicing good habits.

#### Edit

After creating the topic branch, we'll edit it to make it a little more
descriptive. I prefer the [Markdown markup
language](http://daringfireball.net/projects/markdown/) to the default RDoc
for this purpose, and if you use the file extension `.md` then GitHub will
automatically format it nicely for you. So, first we'll use Git's version of
the Unix `mv` ("move") command to change the name, and then fill it in with
the contents of [Listing1.8](beginning.html#code-
new_readme):

    
    $ git mv README.rdoc README.md
    $ subl README.md
    

Listing 1.8. The new `README` file, `README.md`.

    
    # Ruby on Rails Tutorial: first application
    
    This is the first application for
    *Ruby on Rails Tutorial: Learn Rails by Example*
    by Michael Hartl.
    

#### Commit

With the changes made, we can take a look at the status of our branch:

    
    $ git status
    # On branch modify-README
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #       renamed:    README.rdoc -> README.md
    #
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #       modified:   README.md
    #
    

At this point, we could use `git add .` as in
Section1.3.2,
but Git provides the `-a` flag as a shortcut for the (very common) case of
committing all modifications to existing files (or files created using `git
mv`, which don't count as new files to Git):

    
    $ git commit -a -m "Improve the README file"
    2 files changed, 5 insertions(+), 243 deletions(-)
    delete mode 100644 README.rdoc
    create mode 100644 README.md
    

Be careful about using the `-a` flag improperly; if you have added any new
files to the project since the last commit, you still have to tell Git about
them using `git add` first.

Note that we write the commit message in the _present_ tense. Git models
commits as a series of patches, and in this context it makes sense to describe
what each commit _does_, rather than what it did. Moreover, this usage matches
up with the commit messages generated by Git commands themselves. See the
GitHub post [Shiny new commit styles](https://github.com/blog/926-shiny-new-
commit-styles) for more information.

#### Merge

Now that we've finished making our changes, we're ready to _merge_ the results
back into our master branch:

    
    $ git checkout master
    Switched to branch 'master'
    $ git merge modify-README
    Updating 34f06b7..2c92bef
    Fast forward
    README.rdoc     |  243 --------------------------------------------------
    README.md       |    5 +
    2 files changed, 5 insertions(+), 243 deletions(-)
    delete mode 100644 README.rdoc
    create mode 100644 README.md
    

Note that the Git output frequently includes things like `34f06b7`, which are
related to Git's internal representation of repositories. Your exact results
will differ in these details, but otherwise should essentially match the
output shown above.

After you've merged in the changes, you can tidy up your branches by deleting
the topic branch using `git branch -d` if you're done with it:

    
    $ git branch -d modify-README
    Deleted branch modify-README (was 2c92bef).
    

This step is optional, and in fact it's quite common to leave the topic branch
intact. This way you can switch back and forth between the topic and master
branches, merging in changes every time you reach a natural stopping point.

As mentioned above, it's also possible to abandon your topic branch changes,
in this case with `git branch -D`:

    
    # For illustration only; don't do this unless you mess up a branch
    $ git checkout -b topic-branch
    $ <really screw up the branch>
    $ git add .
    $ git commit -a -m "Major screw up"
    $ git checkout master
    $ git branch -D topic-branch
    

Unlike the `-d` flag, the `-D` flag will delete the branch even though we
haven't merged in the changes.

#### Push

Now that we've updated the `README`, we can push the changes up to GitHub to
see the result. Since we have already done one push
(Section1.3.4, on most
systems we can omit `origin master`, and simply run `git push`:

    
    $ git push
    

As promised, GitHub nicely formats the new file using Markdown
(Figure1.9.

!new_readme

Figure 1.9: The improved `README` file formatted with
Markdown.[(full
size)](http://railstutorial.org/images/figures/new_readme-full.png)

## 1.4 Deploying

Even at this early stage, we're already going to deploy our (still-empty)
Rails application to production. This step is optional, but deploying early
and often allows us to catch any deployment problems early in our development
cycle. The alternative--deploying only after laborious effort sealed away in a
development environment--often leads to terrible integration headaches when
launch time comes.20

Deploying Rails applications used to be a pain, but the Rails deployment
ecosystem has matured rapidly in the past few years, and now there are several
great options. These include shared hosts or virtual private servers running
Phusion Passenger (a module for the Apache and
Nginx21 web servers), full-service deployment companies such as [Engine
Yard](http://engineyard.com/) and Rails Machine,
and cloud deployment services such as [Engine Yard
Cloud](http://cloud.engineyard.com) and Heroku.

My favorite Rails deployment option is Heroku, which is a hosted platform
built specifically for deploying Rails and other web applications.22 Heroku
makes deploying Rails applications ridiculously easy--as long as your source
code is under version control with Git. (This is yet another reason to follow
the Git setup steps in [Section1.3](beginning.html#sec-
version_control) if you haven't already.) The rest of this section is
dedicated to deploying our first application to Heroku.

### 1.4.1 Heroku setup

Heroku uses the PostgreSQL database (pronounced
"post-gres-cue-ell", and often called "Postgres" for short), which means that
we need to add the `pg`gem in the production environment to
allow Rails to talk to Postgres:

    
    group :production do
      gem 'pg', '0.12.2'
    end
    

Appending this code to the `Gemfile` from
Listing1.5
yields Listing1.9.

Listing 1.9. A `Gemfile` with an added `pg` gem for PostgreSQL.

    
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
    

To install it, we run `bundle install` with a special flag:

    
    $ bundle install --without production
    

The `--without production` option prevents the local installation of any
production gems, which in this case is just`pg`.23

Next we have to create and configure a new Heroku account. The first step is
to sign up for Heroku; after checking your
email to complete the creation of your account, install the necessary Heroku
software using the Heroku Toolbelt.24 Then use
the `heroku` command to log in at the command line (you may have to exit and
restart your terminal program first):

    
    $ heroku login
    

Finally, navigate back to your Rails project directory and use the `heroku`
command to create a place on the Heroku servers for the sample app to live
(Listing1.10.

Listing 1.10. Creating a new application at Heroku.

    
    $ cd ~/rails_projects/first_app
    $ heroku create
    Created http://stormy-cloud-5881.herokuapp.com/ |
    git@heroku.com:stormy-cloud-5881.herokuapp.com
    Git remote heroku added
    

The `heroku` command creates a new subdomain just for our application,
available for immediate viewing. There's nothing there yet, though, so let's
get busy deploying.

### 1.4.2 Heroku deployment, step one

To deploy the application, the first step is to use Git to push it up to
Heroku:

    
    $ git push heroku master
    

### 1.4.3 Heroku deployment, step two

There is no step two! We're already done
(Figure1.10. To see
your newly deployed application, you can visit the address that you saw when
you ran `heroku create` (i.e., [Listing1.10](beginning.html
#code-heroku_create), but with the address for your app, not the address for
mine). You can also use an argument to the `heroku` command that automatically
opens your browser with the right address:

    
    $ heroku open
    

Because of the details of their setup, the "About your application's
environment" link doesn't work on Heroku. Don't worry; this is normal. The
error will go away (in the context of the full sample application) when we
remove the default Rails page in [Section5.3.2](filling-in-
the-layout.html#sec-rails_routes).

!heroku_app_31

Figure 1.10: The first Rails Tutorial application running on
Heroku.[(full
size)](http://railstutorial.org/images/figures/heroku_app_31-full.png)

Once you've deployed successfully, Heroku provides a beautiful interface for
administering and configuring your application
(Figure1.11.

!heroku_info

Figure 1.11: The beautiful interface at Heroku.[(full
size)](http://railstutorial.org/images/figures/heroku_info-full.png)

### 1.4.4 Heroku commands

There are many Heroku commands,
and we'll barely scratch the surface in this book. Let's take a minute to show
just one of them by renaming the application as follows:

    
    $ heroku rename railstutorial
    

Don't use this name yourself; it's already taken by me! In fact, you probably
shouldn't bother with this step right now; using the default address supplied
by Heroku is fine. But if you do want to rename your application, you can
arrange for it to be reasonably secure by using a random or obscure subdomain,
such as the following:

    
    hwpcbmze.heroku.com
    seyjhflo.heroku.com
    jhyicevg.heroku.com

With a random subdomain like this, someone could visit your site only if you
gave them the address. (By the way, as a preview of Ruby's compact
awesomeness, here's the code I used to generate the random subdomains:

    
    ('a'..'z').to_a.shuffle[0..7].join
    

Pretty sweet.)

In addition to supporting subdomains, Heroku also supports custom domains. (In
fact, the Ruby on Rails Tutorial site lives at
Heroku; if you're reading this book online, you're looking at a Heroku-hosted
site right now!) See the Heroku documentation
for more information about custom domains and other Heroku topics.

## 1.5 Conclusion

We've come a long way in this chapter: installation, development environment
setup, version control, and deployment. If you want to share your progress at
this point, feel free to send a tweet or Facebook status update with something
like this:

[I'm learning Ruby on Rails with @railstutorial! http://railstutorial.org/](ht
tp://twitter.com/?status=I'm%20learning%20Ruby%20on%20Rails%20with%20@railstut
orial!%20http://railstutorial.org/)



All that's left is to actually start learning Rails! Let's get to it.

 Chapter 2 A demo app» 

  1. _URI_ stands for Uniform Resource Identifier, while the slightly less general _URL_ stands for Uniform Resource Locator. In practice, the URI is usually equivalent to "the thing you see in the address bar of your browser".↑
  2. http://tryruby.org/↑
  3. http://railsforzombies.org/↑
  4. http://railstutorial.org/screencasts↑
  5. When reading the _Rails Tutorial_, you may find it convenient to follow an internal section link to look at the reference and then immediately go back to where you were before. This is easy when reading the book as a web page, since you can just use the Back button of your browser, but both Adobe Reader and OSX's Preview allow you to do this with the PDF as well. In Reader, you can right-click on the document and select "Previous View" to go back. In Preview, use the Go menu: `Go > Back`.↑
  6. http://railstutorial.org/help↑
  7. https://github.com/perfectionist/sample_project/wiki↑
  8. The vi editor is one of the most ancient yet powerful weapons in the Unix arsenal, and Vim is "vi improved".↑
  9. https://github.com/mhartl/rails_tutorial_sublime_text↑
  10. If you haven't used IRC before, I suggest you start by searching the web for "irc client <your platform>". Two good native clients for OSX are Colloquy](http://colloquy.info/) and LimeChat. And of course there's always the web interface at [http://webchat.freenode.net/?channels=rvm.↑
  11. You might have to install the Subversion version control system to get this to work.↑
  12. http://mxcl.github.com/homebrew/↑
  13. https://developer.apple.com/downloads/↑
  14. Recall from Section1.1.3 that Windows users might have to type `ruby rails server` instead.↑
  15. Normally, websites run on port 80, but this usually requires special privileges, so Rails picks a less restricted higher-numbered port for the development server.↑
  16. Normally this is a feature, since it lets you continue to use the command line after launching your editor, but Git interprets the detachment as closing the file with an empty commit message, which prevents the commit from going through. I only mention this point because it can be seriously confusing if you try to set your editor to `subl` or `gvim` without the flag. If you find this note confusing, feel free to ignore it.↑
  17. If you can't see the `.gitignore` file in your directory, you may need to configure your directory viewer to show hidden files.↑
  18. If in the future any unwanted files start showing up when you type `git status`, just add them to your `.gitignore` file from Listing1.7.↑
  19. See the chapter Git Branching in _Pro Git_ for details.↑
  20. Though it shouldn't matter for the example applications in the _Rails Tutorial_, if you're worried about accidentally making your app public too soon there are several options; see Section1.4.4 for one.↑
  21. Pronounced "Engine X".↑
  22. Heroku works with any Ruby web platform that uses Rack middleware](http://rack.rubyforge.org/), which provides a standard interface between web frameworks and web servers. Adoption of the Rack interface has been extraordinarily strong in the Ruby community, including frameworks as varied as Sinatra](http://www.sinatrarb.com/), [Ramaze, [Camping, and Rails, which means that Heroku basically supports any Ruby web app.↑
  23. Because the only gem we've added is restricted to a production environment, right now this command doesn't actually install any additional local gems, but for technical reasons it's needed to update `Gemfile.lock`.↑
  24. https://toolbelt.heroku.com/↑

