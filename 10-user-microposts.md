# Chapter 10 User microposts

Chapter9
saw the completion of the REST actions for the Users resource, so the time has
finally come to add a second full resource: user _microposts_.1 These are
short messages associated with a particular user, first seen in larval form in
Chapter2. In this chapter, we will
make a full-strength version of the sketch from
Section2.3 by
constructing the Micropost data model, associating it with the User model
using the `has_many` and `belongs_to` methods, and then making the forms and
partials needed to manipulate and display the results. In
Chapter11, we'll complete our
tiny Twitter clone by adding the notion of _following_ users in order to
receive a _feed_ of their microposts.

If you're using Git for version control, I suggest making a topic branch as
usual:

    
    $ git checkout -b user-microposts
    

## 10.1 A Micropost model

We begin the Microposts resource by creating a Micropost model, which captures
the essential characteristics of microposts. What follows builds on the work
from Section2.3;
as with the model in that section, our new Micropost model will include data
validations and an association with the User model. Unlike that model, the
present Micropost model will be fully tested, and will also have a default
_ordering_ and automatic _destruction_ if its parent user is destroyed.

### 10.1.1 The basic model

The Micropost model needs only two attributes: a `content` attribute to hold
the micropost's content,2 and a `user_id` to associate a micropost with a
particular user. As with the case of the User model
([Listing6.1](modeling-users.html#code-
generate_user_model)), we generate it using `generate model`:

    
    $ rails generate model Micropost content:string user_id:integer
    

This produces a migration to create a `microposts` table in the database
([Listing10.1](user-microposts.html#code-
micropost_migration)); compare it to the analogous migration for the `users`
table from [Listing6.2](modeling-users.html#code-
users_migration).

Listing 10.1. The Micropost migration. (Note the index on `user_id` and
`created_at`.)

`db/migrate/[timestamp]_create_microposts.rb`

    
    class CreateMicroposts < ActiveRecord::Migration
      def change
        create_table :microposts do |t|
          t.string :content
          t.integer :user_id
    
          t.timestamps
        end
        add_index :microposts, [:user_id, :created_at]
      end
    end
    

Note that, since we expect to retrieve all the microposts associated with a
given userid in reverse order of creation,
[Listing10.1](user-microposts.html#code-
micropost_migration) adds an index ([Box6.2](modeling-
users.html#sidebar-database_indices)) on the `user_id` and `created_at`
columns:

    
    add_index :microposts, [:user_id, :created_at]
    

By including both the `user_id` and `created_at` columns as an array, we
arrange for Rails to create a _multiple key index_, which means that Active
Record uses _both_ keys at the same time. Note also the `t.timestamps` line,
which (as mentioned in [Section6.1.1](modeling-users.html
#sec-database_migrations)) adds the magic `created_at` and `updated_at`
columns. We'll put the `created_at` column to work in
[Section10.1.4](user-microposts.html#sec-
ordering_and_dependency) and [Section10.2.1](user-
microposts.html#sec-augmenting_the_user_show_page).

We'll start with some minimal tests for the Micropost model based on the
analogous tests for the User model ([Listing6.8](modeling-
users.html#code-user_spec)). In particular, we verify that a micropost object
responds to the `content` and `user_id` attributes, as shown in
[Listing10.2](user-microposts.html#code-
initial_micropost_spec).

Listing 10.2. The initial Micropost spec.

`spec/models/micropost_spec.rb`

    
    require 'spec_helper'
    
    describe Micropost do
    
      let(:user) { FactoryGirl.create(:user) }
      before do
        # This code is wrong!
        @micropost = Micropost.new(content: "Lorem ipsum", user_id: user.id)
      end
    
      subject { @micropost }
    
      it { should respond_to(:content) }
      it { should respond_to(:user_id) }
    end
    

We can get these tests to pass by running the microposts migration and
preparing the test database:

    
    $ bundle exec rake db:migrate
    $ bundle exec rake db:test:prepare
    

The result is a Micropost model with the structure shown in
Figure10.1.

!micropost_model

Figure 10.1: The Micropost data model.

You should verify that the tests pass:

    
    $ bundle exec rspec spec/models/micropost_spec.rb
    

Even though the tests are passing, you might have noticed this code:

    
    let(:user) { FactoryGirl.create(:user) }
    before do
      # This code is wrong!
      @micropost = Micropost.new(content: "Lorem ipsum", user_id: user.id)
    end
    

The comment indicates that the code in the `before` block is wrong. See if you
can guess why. We'll see the answer in the next section.

### [10.1.2 Accessible attributes and the first validation](user-
microposts.html#sec-accessible_attribute)

To see why the code in the `before` block is wrong, we first start with
validation tests for the Micropost model ([Listing10.3
](user-microposts.html#code-micropost_validity_test)). (Compare with the User
model tests in [Listing6.11](modeling-users.html#code-
failing_validates_name_spec).)

Listing 10.3. Tests for the validity of a new micropost.

`spec/models/micropost_spec.rb`

    
    require 'spec_helper'
    
    describe Micropost do
    
      let(:user) { FactoryGirl.create(:user) }
      before do
        # This code is wrong!
        @micropost = Micropost.new(content: "Lorem ipsum", user_id: user.id)
      end
    
      subject { @micropost }
    
      it { should respond_to(:content) }
      it { should respond_to(:user_id) }
    
      it { should be_valid }
    
      describe "when user_id is not present" do
        before { @micropost.user_id = nil }
        it { should_not be_valid }
      end
    end
    

This code requires that the micropost be valid and tests for the presence of
the `user_id` attribute. We can get these tests to pass with the simple
presence validation shown in [Listing10.4](user-
microposts.html#code-micropost_user_id_validation).

Listing 10.4. A validation for the micropost's `user_id`.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      attr_accessible :content, :user_id  
      validates :user_id, presence: true
    end
    

Now we're prepared to see why

    
    @micropost = Micropost.new(content: "Lorem ipsum", user_id: user.id)
    

is wrong. The problem is that by default (as of Rails3.2.3)
_all_ of the attributes for our Micropost model are accessible. As discussed
in [Section6.1.2.2](modeling-users.html#sec-
accessible_attributes) and [Section9.4.1.1](updating-
showing-and-deleting-users.html#sec-revisiting_attr_accessible), this means
that anyone could change any aspect of a micropost object simply by using a
command-line client to issue malicious requests. For example, a malicious user
could change the `user_id` attributes on microposts, thereby associating
microposts with the wrong users. This means that we should remove `:user_id`
from the `attr_accessible` list, and once we do, the code above will fail.
We'll fix this issue in [Section10.1.3](user-
microposts.html#sec-user_micropost_associations).

### [10.1.3 User/Micropost associations](user-microposts.html#sec-
user_micropost_associations)

When constructing data models for web applications, it is essential to be able
to make _associations_ between individual models. In the present case, each
micropost is associated with one user, and each user is associated with
(potentially) many microposts--a relationship seen briefly in
[Section2.3.3](a-demo-app.html#sec-
demo_user_has_many_microposts) and shown schematically in
[Figure10.2](user-microposts.html#fig-
micropost_belongs_to_user) and [Figure10.3](user-
microposts.html#fig-user_has_many_microposts). As part of implementing these
associations, we'll write tests for the Micropost model that, unlike
[Listing10.2](user-microposts.html#code-
initial_micropost_spec), are compatible with the use of `attr_accessible` in
[Listing10.7](user-microposts.html#code-
micropost_accessible_attribute).

!micropost_belongs_to_user

Figure 10.2: The `belongs_to` relationship between a micropost and its user.

!user_has_many_microposts

Figure 10.3: The `has_many` relationship between a user and its microposts.

Using the `belongs_to`/`has_many` association defined in this section, Rails
constructs the methods shown in [Table10.1](user-
microposts.html#table-association_methods).

**Method****Purpose**

`micropost.user`

Return the User object associated with the micropost.

`user.microposts`

Return an array of the user's microposts.

`user.microposts.create(arg)`

Create a micropost (`user_id = user.id`).

`user.microposts.create!(arg)`

Create a micropost (exception on failure).

`user.microposts.build(arg)`

Return a new Micropost object (`user_id = user.id`).

Table 10.1: A summary of user/micropost association methods.

Note from [Table10.1](user-microposts.html#table-
association_methods) that instead of

    
    Micropost.create
    Micropost.create!
    Micropost.new
    

we have

    
    user.microposts.create
    user.microposts.create!
    user.microposts.build
    

This pattern is the canonical way to make a micropost: _through_ its
association with a user. When a new micropost is made in this way, its
`user_id` is _automatically_ set to the right value, which fixes the issue
noted in [Section10.1.2](user-microposts.html#sec-
accessible_attribute). In particular, we can replace the code

    
    let(:user) { FactoryGirl.create(:user) }
    before do
      # This code is wrong!
      @micropost = Micropost.new(content: "Lorem ipsum", user_id: user.id)
    end
    

from [Listing10.3](user-microposts.html#code-
micropost_validity_test) with

    
    let(:user) { FactoryGirl.create(:user) }
    before { @micropost = user.microposts.build(content: "Lorem ipsum") }
    

Once we define the proper associations, the resulting `@micropost` variable
will automatically have `user_id` equal to its associated user.

Building the micropost through the User association doesn't fix the security
problem of having an accessible `user_id`, and because this is such an
important security concern we'll add a failing test to catch it, as shown in
[Listing10.5](user-microposts.html#code-
attr_accessible_user_id_test).

Listing 10.5. A test to ensure that the `user_id` isn't accessible.

`spec/models/micropost_spec.rb`

    
    require 'spec_helper'
    
    describe Micropost do
    
      let(:user) { FactoryGirl.create(:user) }
      before { @micropost = user.microposts.build(content: "Lorem ipsum") }
    
      subject { @micropost }
      .
      .
      .
      describe "accessible attributes" do
        it "should not allow access to user_id" do
          expect do
            Micropost.new(user_id: user.id)
          end.to raise_error(ActiveModel::MassAssignmentSecurity::Error)
        end    
      end
    end
    

This test verifies that calling `Micropost.new` with a nonempty `user_id`
raises a mass assignment security error exception. This behavior is on by
default as of Rails3.2.3, but previous versions had it off,
so you should make sure that your application is configured properly, as shown
in [Listing10.6](user-microposts.html#code-
application_whitelist).

Listing 10.6. Ensuring that Rails throws errors on invalid mass assignment.

`config/application.rb`

    
    .
    .
    .
    module SampleApp
      class Application < Rails::Application
        .
        .
        .
        config.active_record.whitelist_attributes = true
        .
        .
        .
      end
    end
    

In the case of the Micropost model, there is only _one_ attribute that needs
to be editable through the web, namely, the `content` attribute, so we need to
remove `:user_id` from the accessible list, as shown in
[Listing10.7](user-microposts.html#code-
micropost_accessible_attribute).

Listing 10.7. Making the `content` attribute (and _only_ the `content`
attribute) accessible.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      attr_accessible :content
    
      validates :user_id, presence: true
    end
    

As seen in [Table10.1](user-microposts.html#table-
association_methods), another result of the user/micropost association is
`micropost.user`, which simply returns the micropost's user. We can test this
with the `it` and `its` methods as follows:

    
    it { should respond_to(:user) }
    its(:user) { should == user }
    

The resulting Micropost model tests are shown in
[Listing10.8](user-microposts.html#code-
micropost_belongs_to_user_spec).

Listing 10.8. Tests for the micropost's user association.

`spec/models/micropost_spec.rb`

    
    require 'spec_helper'
    
    describe Micropost do
    
      let(:user) { FactoryGirl.create(:user) }
      before { @micropost = user.microposts.build(content: "Lorem ipsum") }
    
      subject { @micropost }
    
      it { should respond_to(:content) }
      it { should respond_to(:user_id) }
      it { should respond_to(:user) }
      its(:user) { should == user }
    
      it { should be_valid }
    
      describe "accessible attributes" do
        it "should not allow access to user_id" do
          expect do
            Micropost.new(user_id: user.id)
          end.to raise_error(ActiveModel::MassAssignmentSecurity::Error)
        end    
      end
    
      describe "when user_id is not present" do
        before { @micropost.user_id = nil }
        it { should_not be_valid }
      end
    end
    

On the User model side of the association, we'll defer the more detailed tests
to [Section10.1.4](user-microposts.html#sec-
ordering_and_dependency); for now, we'll simply test for the presence of a
`microposts` attribute ([Listing10.9](user-microposts.html
#code-user_has_many_microposts_spec)).

Listing 10.9. A test for the user's `microposts` attribute.

`spec/models/user_spec.rb`

    
    require 'spec_helper'
    
    describe User do
    
      before do
        @user = User.new(name: "Example User", email: "user@example.com", 
                         password: "foobar", password_confirmation: "foobar")
      end
    
      subject { @user }
      .
      .
      .
      it { should respond_to(:authenticate) }
      it { should respond_to(:microposts) }
      .
      .
      .
    end
    

After all that work, the code to implement the association is almost comically
short: we can get the tests in both [Listing10.8](user-
microposts.html#code-micropost_belongs_to_user_spec) and
[Listing10.9](user-microposts.html#code-
user_has_many_microposts_spec) to pass by adding just two lines: `belongs_to
:user` ([Listing10.10](user-microposts.html#code-
micropost_belongs_to_user)) and `has_many :microposts`
([Listing10.11](user-microposts.html#code-
user_has_many_microposts)).

Listing 10.10. A micropost `belongs_to` a user.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      attr_accessible :content
      belongs_to :user
    
      validates :user_id, presence: true  
    end
    

Listing 10.11. A user `has_many` microposts.

`app/models/user.rb`

    
    class User < ActiveRecord::Base
      attr_accessible :name, :email, :password, :password_confirmation
      has_secure_password
      has_many :microposts
      .
      .
      .
    end
    

At this point, you should compare the entries in [Table10.1
](user-microposts.html#table-association_methods) with the code in
[Listing10.8](user-microposts.html#code-
micropost_belongs_to_user_spec) and [Listing10.9](user-
microposts.html#code-user_has_many_microposts_spec) to satisfy yourself that
you understand the basic nature of the associations. You should also check
that the tests pass:

    
    $ bundle exec rspec spec/models
    

### [10.1.4 Micropost refinements](user-microposts.html#sec-
ordering_and_dependency)

The test in [Listing10.9](user-microposts.html#code-
user_has_many_microposts_spec) of the `has_many` association doesn't test for
much--it merely verifies the _existence_ of a `microposts` attribute. In this
section, we'll add _ordering_ and _dependency_ to microposts, while also
testing that the `user.microposts` method actually returns an array of
microposts.

We will need to construct some microposts in the User model test, which means
that we should make a micropost factory at this point. To do this, we need a
way to make an association in Factory Girl. Happily, this is easy, as seen in
[Listing10.12](user-microposts.html#code-
micropost_factory).

Listing 10.12. The complete factory file, including a new factory for
microposts.

`spec/factories.rb`

    
    FactoryGirl.define do
      factory :user do
        sequence(:name)  { |n| "Person #{n}" }
        sequence(:email) { |n| "person_#{n}@example.com"}   
        password "foobar"
        password_confirmation "foobar"
    
        factory :admin do
          admin true
        end
      end
    
      factory :micropost do
        content "Lorem ipsum"
        user
      end
    end
    

Here we tell Factory Girl about the micropost's associated user just by
including a user in the definition of the factory:

    
    factory :micropost do
      content "Lorem ipsum"
      user
    end
    

As we'll see in the next section, this allows us to define factory microposts
as follows:

    
    FactoryGirl.create(:micropost, user: @user, created_at: 1.day.ago)
    

#### Default scope

By default, using `user.microposts` to pull a user's microposts from the
database makes no guarantees about the order of the posts, but (following the
convention of blogs and Twitter) we want the microposts to come out in reverse
order of when they were created, i.e., most recent first. To test this
ordering, we first create a couple of microposts as follows:

    
    FactoryGirl.create(:micropost, user: @user, created_at: 1.day.ago)
    FactoryGirl.create(:micropost, user: @user, created_at: 1.hour.ago)
    

Here we indicate (using the time helpers discussed in
Box8.1 that
the second post was created more recently, i.e., `1.hour.ago`, while the first
post was created `1.day.ago`. Note how convenient the use of Factory Girl is:
not only can we assign the user using mass assignment (since factories bypass
`attr_accessible`), we can also set `created_at` manually, which Active Record
won't allow us to do. (Recall that `created_at` and `updated_at` are "magic"
columns, automatically set to the proper creation and update timestamps, so
any explicit initialization values are overwritten by the magic.)

Most database adapters (including the one for SQLite) return the microposts in
order of their ids, so we can arrange for an initial test that almost
certainly fails using the code in [Listing10.13](user-
microposts.html#code-micropost_ordering_test). This uses the `let!` (read "let
bang") method in place of `let`; the reason is that `let` variables are
_lazy_, meaning that they only spring into existence when referenced. The
problem is that we want the microposts to exist immediately, so that the
timestamps are in the right order and so that `@user.microposts` isn't empty.
We accomplish this with `let!`, which forces the corresponding variable to
come into existence immediately.

Listing 10.13. Testing the order of a user's microposts.

`spec/models/user_spec.rb`

    
    require 'spec_helper'
    
    describe User do
      .
      .
      .
      describe "micropost associations" do
    
        before { @user.save }
        let!(:older_micropost) do 
          FactoryGirl.create(:micropost, user: @user, created_at: 1.day.ago)
        end
        let!(:newer_micropost) do
          FactoryGirl.create(:micropost, user: @user, created_at: 1.hour.ago)
        end
    
        it "should have the right microposts in the right order" do
          @user.microposts.should == [newer_micropost, older_micropost]
        end
      end
    end
    

The key line here is

    
    @user.microposts.should == [newer_micropost, older_micropost]
    

indicating that the posts should be ordered newest first. This should fail
because by default the posts will be ordered byid, i.e.,
`[older_micropost, newer_micropost]`. This test also verifies the basic
correctness of the `has_many` association itself, by checking (as indicated in
[Table10.1](user-microposts.html#table-
association_methods)) that `user.microposts` is an array of microposts.

To get the ordering test to pass, we use a Rails facility called
`default_scope` with an `:order` parameter, as shown in
[Listing10.14](user-microposts.html#code-
micropost_ordering). (This is our first example of the notion of _scope_. We
will learn about scope in a more general context in
Chapter11

Listing 10.14. Ordering the microposts with `default_scope`.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      .
      .
      .
      default_scope order: 'microposts.created_at DESC'
    end
    

The order here is `'microposts.created_at DESC'`, where `DESC` is SQL for
"descending", i.e., in descending order from newest to oldest.

#### Dependent: destroy

Apart from proper ordering, there is a second refinement we'd like to add to
microposts. Recall from [Section9.4](updating-showing-and-
deleting-users.html#sec-destroying_users) that site administrators have the
power to _destroy_ users. It stands to reason that, if a user is destroyed,
the user's microposts should be destroyed as well. We can test for this by
first destroying a micropost's user and then verifying that the associated
microposts are no longer in the database.

In order to test destroying microposts properly, we first need to capture a
given user's posts in a local variable, and then destroy the user. A
straighforward implementation looks like this:

    
    microposts = @user.microposts
    @user.destroy
    microposts.each do |micropost|
      # Make sure the micropost doesn't appear in the database.
    end
    

Unfortunately, this doesn't work, due to a subtlety about Ruby arrays. Array
assignment in Ruby copies a _reference_ to the array, not the full array
itself, which means that changes to the original array also affect the copy.
For example, suppose we create an array, assign a second variable to it, and
then reverse the first array in place using the `reverse!` method:

    
    $ rails console
    >> a = [1, 2, 3]
    => [1, 2, 3]
    >> b = a
    => [1, 2, 3]
    >> a.reverse!
    => [3, 2, 1]
    >> a
    => [3, 2, 1]
    >> b
    => [3, 2, 1]
    

Somewhat suprisingly, here `b` gets reversed as well as `a`. This is because
both `a` and `b` point to the same array. (The same thing happens with other
Ruby data structures, such as strings and hashes.)

In the case of a user's microposts, we would have this:

    
    $ rails console --sandbox
    >> @user = User.first
    >> microposts = @user.microposts
    >> @user.destroy
    >> microposts
    => []
    

(Because we haven't implemented the destruction of associated microposts yet,
this code won't currently work, and is included only to illustrate the
principle.) Here we see that destroying a user leaves the `microposts`
variable with no elements; i.e., it's the empty array`[]`.

This behavior means that we must take great care when making duplicates of
Ruby objects. To duplicate relatively simple objects such as arrays, we can
use the `dup` method:

    
    $ rails console
    >> a = [1, 2, 3]
    => [1, 2, 3]
    >> b = a.dup
    => [1, 2, 3]
    >> a.reverse!
    => [3, 2, 1]
    >> a
    => [3, 2, 1]
    >> b
    => [1, 2, 3]
    

(This is known as a "shallow copy". Making a "deep copy" is a much more
difficult problem, and in fact has no general solution, but dropping "ruby
deep copy" into a search engine should be enough get you started if you need
to copy a more complicated structure such as a nested array.) Applying the
`dup` method to the user's microposts gives us code like this:

    
    microposts = @user.microposts.dup
    @user.destroy
    microposts.should_not be_empty
    microposts.each do |micropost|
      # Make sure the micropost doesn't appear in the database.
    end
    

Here we've included the line

    
    microposts.should_not be_empty
    

as a safety check to catch any errors should the `dup` ever be accidentally
removed.3 The full implementation appears in [Listing10.15
](user-microposts.html#code-micropost_dependency_test).

Listing 10.15. Testing that microposts are destroyed when users are.

`spec/models/user_spec.rb`

    
    require 'spec_helper'
    
    describe User do
      .
      .
      .
      describe "micropost associations" do
    
        before { @user.save }
        let!(:older_micropost) do 
          FactoryGirl.create(:micropost, user: @user, created_at: 1.day.ago)
        end
        let!(:newer_micropost) do
          FactoryGirl.create(:micropost, user: @user, created_at: 1.hour.ago)
        end
        .
        .
        .
        it "should destroy associated microposts" do
          microposts = @user.microposts.dup
          @user.destroy
          microposts.should_not be_empty
          microposts.each do |micropost|
            Micropost.find_by_id(micropost.id).should be_nil
          end
        end
      end
      .
      .
      .
    end
    

Here we have used `Micropost.find_by_id`, which returns `nil` if the record is
not found, whereas `Micropost.find` raises an exception on failure, which is a
bit harder to test for. (In case you're curious,

    
    lambda do 
      Micropost.find(micropost.id)
    end.should raise_error(ActiveRecord::RecordNotFound)
    

does the trick in this case.)

The application code to get [Listing10.15](user-
microposts.html#code-micropost_dependency_test) to pass is less than one line;
in fact, it's just an option to the `has_many` association method, as shown in
[Listing10.16](user-microposts.html#code-
micropost_dependency).

Listing 10.16. Ensuring that a user's microposts are destroyed along with the
user.

`app/models/user.rb`

    
    class User < ActiveRecord::Base
      attr_accessible :name, :email, :password, :password_confirmation
      has_secure_password
      has_many :microposts, dependent: :destroy
      .
      .
      .
    end
    

Here the option `dependent: :destroy` in

    
    has_many :microposts, dependent: :destroy
    

arranges for the dependent microposts (i.e., the ones belonging to the given
user) to be destroyed when the user itself is destroyed. This prevents
userless microposts from being stranded in the database when admins choose to
remove users from the system.

With that, the final form of the user/micropost association is in place, and
all the tests should be passing:

    
    $ bundle exec rspec spec/
    

### [10.1.5 Content validations](user-microposts.html#sec-
micropost_validations)

Before leaving the Micropost model, we'll add validations for the micropost
`content` (following the example from [Section2.3.2](a
-demo-app.html#sec-putting_the_micro_in_microposts)). Like the `user_id`, the
`content` attribute must be present, and it is further constrained to be no
longer than 140 characters, making it an honest _micro_post. The tests
generally follow the examples from the User model validation tests in
Section6.2, as
shown in [Listing10.17](user-microposts.html#code-
micropost_validations_tests).

Listing 10.17. Tests for the Micropost model validations.

`spec/models/micropost_spec.rb`

    
    require 'spec_helper'
    
    describe Micropost do
    
      let(:user) { FactoryGirl.create(:user) }
      before { @micropost = user.microposts.build(content: "Lorem ipsum") }
      .
      .
      .
      describe "when user_id is not present" do
        before { @micropost.user_id = nil }
        it { should_not be_valid }
      end
    
      describe "with blank content" do
        before { @micropost.content = " " }
        it { should_not be_valid }
      end
    
      describe "with content that is too long" do
        before { @micropost.content = "a" * 141 }
        it { should_not be_valid }
      end
    end
    

As in [Section6.2](modeling-users.html#sec-
user_validations), the code in [Listing10.17](user-
microposts.html#code-micropost_validations_tests) uses string multiplication
to test the micropost length validation:

    
    $ rails console
    >> "a" * 10
    => "aaaaaaaaaa"
    >> "a" * 141
    => "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    

The application code is a one-liner:

    
    validates :content, presence: true, length: { maximum: 140 }
    

The resulting Micropost model is shown in [Listing10.18
](user-microposts.html#code-micropost_validations).

Listing 10.18. The Micropost model validations.

`app/models/micropost.rb`

    
    class Micropost < ActiveRecord::Base
      attr_accessible :content
    
      belongs_to :user
    
      validates :content, presence: true, length: { maximum: 140 }
      validates :user_id, presence: true
    
      default_scope order: 'microposts.created_at DESC'
    end
    

## 10.2 Showing microposts

Although we don't yet have a way to create microposts through the web--that
comes in [Section10.3.2](user-microposts.html#sec-
creating_microposts)--that won't stop us from displaying them (and testing
that display). Following Twitter's lead, we'll plan to display a user's
microposts not on a separate microposts `index` page, but rather directly on
the user `show` page itself, as mocked up in [Figure10.4
](user-microposts.html#fig-user_microposts_mockup). We'll start with fairly
simple ERb templates for adding a micropost display to the user profile, and
then we'll add microposts to the sample data populator from
[Section9.3.2](updating-showing-and-deleting-users.html
#sec-sample_users) so that we have something to display.

![user_microposts_mockup_bootstrap](images/figures/user_microposts_mockup_boot
strap.png)

Figure 10.4: A mockup of a profile page with
microposts.[(full
size)](http://railstutorial.org/images/figures
/user_microposts_mockup_bootstrap-full.png)

As with the discussion of the signin machinery in
Section8.2.1,
[Section10.2.1](user-microposts.html#sec-
augmenting_the_user_show_page) will often push several elements onto the
stack at a time, and
then pop them off one by one. If you start getting bogged down, be patient;
there's some nice payoff in [Section10.2.2](user-
microposts.html#sec-sample_microposts).

### [10.2.1 Augmenting the user show page](user-microposts.html#sec-
augmenting_the_user_show_page)

We begin with tests for displaying the user's microposts, which we'll create
in the request spec for Users. Our strategy is to create a couple of factory
microposts associated with the user, and then verify that the show page
contains each post's content. We'll also verify that, as in
[Figure10.4](user-microposts.html#fig-
user_microposts_mockup), the total number of microposts also gets displayed.

We can create the posts with the `let` method, but as in
[Listing10.13](user-microposts.html#code-
micropost_ordering_test) we want the association to exist immediately so that
the posts appear on the user show page. To accomplish this, we use the `let!`
variant:

    
    let(:user) { FactoryGirl.create(:user) }
    let!(:m1) { FactoryGirl.create(:micropost, user: user, content: "Foo") }
    let!(:m2) { FactoryGirl.create(:micropost, user: user, content: "Bar") }
    
    before { visit user_path(user) }
    

With the microposts so defined, we can test for their appearance on the
profile page using the code in [Listing10.19](user-
microposts.html#code-user_show_microposts_test).

Listing 10.19. A test for showing microposts on the user `show` page.

`spec/requests/user_pages_spec.rb`

    
    require 'spec_helper'
    
    describe "User pages" do
      .
      .
      .
      describe "profile page" do
        let(:user) { FactoryGirl.create(:user) }
        let!(:m1) { FactoryGirl.create(:micropost, user: user, content: "Foo") }
        let!(:m2) { FactoryGirl.create(:micropost, user: user, content: "Bar") }
    
        before { visit user_path(user) }
    
        it { should have_selector('h1',    text: user.name) }
        it { should have_selector('title', text: user.name) }
    
        describe "microposts" do
          it { should have_content(m1.content) }
          it { should have_content(m2.content) }
          it { should have_content(user.microposts.count) }
        end
      end
      .
      .
      .
    end
    

Note here that we can use the `count` method _through_ the association:

    
    user.microposts.count
    

The association `count` method is smart, and performs the count directly in
the database. In particular, it does _not_ pull all the microposts out of the
database and then call `length` on the resulting array, as this could become
inefficient as the number of microposts grew. Instead, it asks the database to
count the microposts with the given `user_id`. In the unlikely event that
finding the count is still a bottleneck in your application, you can make it
even faster with a [_counter cache_](http://railscasts.com/episodes/23
-counter-cache-column).

Although the tests in [Listing10.19](user-microposts.html
#code-user_show_microposts_test) won't pass until
[Listing10.21](user-microposts.html#code-
micropost_partial), we'll get started on the application code by inserting a
list of microposts into the user profile page, as shown in
[Listing10.20](user-microposts.html#code-
user_show_microposts).

Listing 10.20. Adding microposts to the user `show` page.

`app/views/users/show.html.erb`

    
    <% provide(:title, @user.name) %>
    <div class="row">
      .
      .
      .
      <aside>
        .
        .
        .
      </aside>
      <div class="span8">
        <% if @user.microposts.any? %>
          <h3>Microposts (<%= @user.microposts.count %>)</h3>
          <ol class="microposts">
            <%= render @microposts %>
          </ol>
          <%= will_paginate @microposts %>
        <% end %>
      </div>
    </div>
    

We'll deal with the microposts list momentarily, but there are several other
things to note first. In [Listing10.20](user-
microposts.html#code-user_show_microposts), the use of `if
@user.microposts.any?` (a construction we saw before in
Listing7.23 makes sure
that an empty list won't be displayed when the user has no microposts.

Also note from [Listing10.20](user-microposts.html#code-
user_show_microposts) that we've preemptively added pagination for microposts
through

    
    <%= will_paginate @microposts %>
    

If you compare this with the analogous line on the user index page,
[Listing9.34](updating-showing-and-deleting-users.html
#code-will_paginate_index_view), you'll see that before we had just

    
    <%= will_paginate %>
    

This worked because, in the context of the Users controller, `will_paginate`
_assumes_ the existence of an instance variable called `@users` (which, as we
saw in [Section9.3.3](updating-showing-and-deleting-
users.html#sec-pagination), should be of class `ActiveRecord::Relation`). In
the present case, since we are still in the Users controller but want to
paginate _microposts_ instead, we pass an explicit `@microposts` variable to
`will_paginate`. Of course, this means that we will have to define such a
variable in the user `show` action ([Listing10.22](user-
microposts.html#code-user_show_microposts_instance)).

Finally, note that we have taken this opportunity to add a count of the
current number of microposts:

    
    <h3>Microposts (<%= @user.microposts.count %>)</h3>
    

As noted, `@user.microposts.count` is the analogue of the `User.count` method,
except that it counts the microposts belonging to a given user through the
user/micropost association.

We come finally to the micropost list itself:

    
    <ol class="microposts">
      <%= render @microposts %>
    </ol>
    

This code, which uses the _ordered list_ tag`ol`, is
responsible for generating the list of microposts, but you can see that it
just defers the heavy lifting to a micropost partial. We saw in
[Section9.3.4](updating-showing-and-deleting-users.html
#sec-partial_refactoring) that the code

    
    <%= render @users %>
    

automatically renders each of the users in the `@users` variable using the
`_user.html.erb` partial. Similarly, the code

    
    <%= render @microposts %>
    

does exactly the same thing for microposts. This means that we must define a
`_micropost.html.erb` partial (along with a `micropost` views directory), as
shown in [Listing10.21](user-microposts.html#code-
micropost_partial).

Listing 10.21. A partial for showing a single micropost.

`app/views/microposts/_micropost.html.erb`

    
    <li>
      <span class="content"><%= micropost.content %></span>
      <span class="timestamp">
        Posted <%= time_ago_in_words(micropost.created_at) %> ago.
      </span>
    </li>
    

This uses the awesome `time_ago_in_words` helper method, whose effect we will
see in [Section10.2.2](user-microposts.html#sec-
sample_microposts).

Thus far, despite defining all the relevant ERb templates, the test in
[Listing10.19](user-microposts.html#code-
user_show_microposts_test) should have been failing for want of an
`@microposts` variable. We can get it to pass with
[Listing10.22](user-microposts.html#code-
user_show_microposts_instance).

Listing 10.22. Adding an `@microposts` instance variable to the user `show`
action.

`app/controllers/users_controller.rb`

    
    class UsersController < ApplicationController
      .
      .
      .
      def show
        @user = User.find(params[:id])
        @microposts = @user.microposts.paginate(page: params[:page])
      end
    end
    

Notice here how clever `paginate` is--it even works through the microposts
association, reaching into the `microposts` table and pulling out the desired
page of microposts.

At this point, we can get a look at our new user profile page in
[Figure10.5](user-microposts.html#fig-
user_profile_no_microposts). It's rather… disappointing. Of course, this is
because there are not currently any microposts. It's time to change that.

![user_profile_no_microposts_bootstrap](images/figures/user_profile_no_micropo
sts_bootstrap.png)

Figure 10.5: The user profile page with code for microposts--but no
microposts.[(full
size)](http://railstutorial.org/images/figures
/user_profile_no_microposts_bootstrap-full.png)

### 10.2.2 Sample microposts

With all the work making templates for user microposts in
[Section10.2.1](user-microposts.html#sec-
augmenting_the_user_show_page), the ending was rather anticlimactic. We can
rectify this sad situation by adding microposts to the sample populator from
[Section9.3.2](updating-showing-and-deleting-users.html
#sec-sample_users). Adding sample microposts for _all_ the users actually
takes a rather long time, so first we'll select just the first six users4
using the `:limit` option to the `User.all` method:5

    
    users = User.all(limit: 6)
    

We then make 50 microposts for each user (plenty to overflow the pagination
limit of30), generating sample content for each micropost
using the Faker gem's handy [`Lorem.sentence`
method](http://faker.rubyforge.org/rdoc/classes/Faker/Lorem.html).
(`Faker::Lorem.sentence` returns _lorem ipsum_ text; as noted in
Chapter6, _lorem ipsum_ has a
[fascinating back story](http://www.straightdope.com/columns/read/2290/what-
does-the-filler-text-lorem-ipsum-mean).) The result is the new sample data
populator shown in [Listing10.23](user-microposts.html
#code-sample_microposts).

Listing 10.23. Adding microposts to the sample data.

`lib/tasks/sample_data.rake`

    
    namespace :db do
      desc "Fill database with sample data"
      task populate: :environment do
        .
        .
        .
        users = User.all(limit: 6)
        50.times do
          content = Faker::Lorem.sentence(5)
          users.each { |user| user.microposts.create!(content: content) }
        end
      end
    end
    

Of course, to generate the new sample data we have to run the `db:populate`
Rake task:

    
    $ bundle exec rake db:reset
    $ bundle exec rake db:populate
    $ bundle exec rake db:test:prepare
    

With that, we are in a position to enjoy the fruits of our
[Section10.2.1](user-microposts.html#sec-
augmenting_the_user_show_page) labors by displaying information for each
micropost.6 The preliminary results appear in [Figure10.6
](user-microposts.html#fig-user_profile_microposts_no_styling).

![user_profile_microposts_no_styling_bootstrap](images/figures/user_profile_mi
croposts_no_styling_bootstrap.png)

Figure 10.6: The user profile (/users/1 with
unstyled microposts.[(full
size)](http://railstutorial.org/images/figures
/user_profile_microposts_no_styling_bootstrap-full.png)

The page shown in [Figure10.6](user-microposts.html#fig-
user_profile_microposts_no_styling) has no micropost-specific styling, so
let's add some ([Listing10.24](user-microposts.html#code-
micropost_css)) and take a look the resulting pages.7
[Figure10.7](user-microposts.html#fig-
user_profile_with_microposts), which displays the user profile page for the
first (signed-in) user, while [Figure10.8](user-
microposts.html#fig-other_profile_with_microposts) shows the profile for a
second user. Finally, [Figure10.9](user-microposts.html
#fig-user_profile_microposts_page_2_rails_3) shows the _second_ page of
microposts for the first user, along with the pagination links at the bottom
of the display. In all three cases, observe that each micropost display
indicates the time since it was created (e.g., "Posted 1 minute ago."); this
is the work of the `time_ago_in_words` method from
[Listing10.21](user-microposts.html#code-
micropost_partial). If you wait a couple minutes and reload the pages, you'll
see how the text gets automatically updated based on the new time.

Listing 10.24. The CSS for microposts (including all the CSS for this
chapter).

`app/assets/stylesheets/custom.css.scss`

    
    .
    .
    .
    
    /* microposts */
    
    .microposts {
      list-style: none;
      margin: 10px 0 0 0;
    
      li {
        padding: 10px 0;
        border-top: 1px solid #e8e8e8;
      }
    }
    .content {
      display: block;
    }
    .timestamp {
      color: $grayLight;
    }
    .gravatar {
      float: left;
      margin-right: 10px;
    }
    aside {
      textarea {
        height: 100px;
        margin-bottom: 5px;
      }
    }
    

![user_profile_with_microposts_bootstrap](images/figures/user_profile_with_mic
roposts_bootstrap.png)

Figure 10.7: The user profile (/users/1 with
microposts.[(full
size)](http://railstutorial.org/images/figures
/user_profile_with_microposts_bootstrap-full.png)

![other_profile_with_microposts_bootstrap](images/figures/other_profile_with_m
icroposts_bootstrap.png)

Figure 10.8: The profile of a different user, also with microposts
(/users/5.[(full
size)](http://railstutorial.org/images/figures
/other_profile_with_microposts_bootstrap-full.png)

![user_profile_microposts_page_2_rails_3_bootstrap](images/figures/user_profil
e_microposts_page_2_rails_3_bootstrap.png)

Figure 10.9: Micropost pagination links ([/users/1?page=2](http://localhost:30
00/users/1?page=2)).[(full
size)](http://railstutorial.org/images/figures
/user_profile_microposts_page_2_rails_3_bootstrap-full.png)

## [10.3 Manipulating microposts](user-microposts.html#sec-
manipulating_microposts)

Having finished both the data modeling and display templates for microposts,
we now turn our attention to the interface for creating them through the web.
The result will be our third example of using an HTML form to create a
resource--in this case, a Microposts resource.8 In this section, we'll also
see the first hint of a _status feed_--a notion brought to full fruition in
Chapter11. Finally, as with
users, we'll make it possible to destroy microposts through the web.

There is one break with past convention worth noting: the interface to the
Microposts resource will run principally through the Users and StaticPages
controllers, rather than relying on a controller of its own. This means that
the routes for the Microposts resource are unusually simple, as seen in
[Listing10.25](user-microposts.html#code-
microposts_resource). The code in [Listing10.25](user-
microposts.html#code-microposts_resource) leads in turn to the RESTful routes
shown in [Table10.2](user-microposts.html#table-
RESTful_microposts), which is a small subset of the full set of routes seen in
Table2.3.
Of course, this simplicity is a sign of being _more_ advanced, not less--we've
come a long way since our reliance on scaffolding in
Chapter2, and we no longer need most
of its complexity.

Listing 10.25. Routes for the Microposts resource.

`config/routes.rb`

    
    SampleApp::Application.routes.draw do
      resources :users
      resources :sessions,   only: [:new, :create, :destroy]
      resources :microposts, only: [:create, :destroy]
      .
      .
      .
    end
    

**HTTP request****URI****Action****Purpose**

`POST`

/microposts

`create`

create a new micropost

`DELETE`

/microposts/1

`destroy`

delete micropost with id `1`

Table 10.2: RESTful routes provided by the Microposts resource in
[Listing10.25](user-microposts.html#code-
microposts_resource).

### 10.3.1 Access control

We begin our development of the Microposts resource with some access control
in the Microposts controller. The idea is simple: both the `create` and
`destroy` actions should require users to be signed in. The RSpec code to test
for this appears in [Listing10.26](user-microposts.html
#code-micropost_access_control). (We'll test for and add a third protection--
ensuring that only a micropost's user can destroy it--in
[Section10.3.4](user-microposts.html#sec-
destroying_microposts).)

Listing 10.26. Access control tests for microposts.

`spec/requests/authentication_pages_spec.rb`

    
    require 'spec_helper'
    
    describe "Authentication" do
      .
      .
      .
      describe "authorization" do
    
        describe "for non-signed-in users" do
          let(:user) { FactoryGirl.create(:user) }
          .
          .
          .
          describe "in the Microposts controller" do
    
            describe "submitting to the create action" do
              before { post microposts_path }
              specify { response.should redirect_to(signin_path) }
            end
    
            describe "submitting to the destroy action" do
              before { delete micropost_path(FactoryGirl.create(:micropost)) }
              specify { response.should redirect_to(signin_path) }
            end
          end
          .
          .
          .
        end
      end
    end
    

Rather than using the (yet-to-be-built) web interface for microposts, the code
in [Listing10.26](user-microposts.html#code-
micropost_access_control) operates at the level of the individual micropost
actions, a strategy we first saw in [Listing9.14](updating-
showing-and-deleting-users.html#code-edit_update_wrong_user_tests). In this
case, a non-signed-in user is redirected upon submitting a `POST` request to
/microposts (`post microposts_path`, which hits the `create` action) or
submitting a `DELETE` request to /microposts/1 (`delete
micropost_path(micropost)`, which hits the `destroy` action).

Writing the application code needed to get the tests in
[Listing10.26](user-microposts.html#code-
micropost_access_control) to pass requires a little refactoring first. Recall
from [Section9.2.1](updating-showing-and-deleting-
users.html#sec-requiring_signed_in_users) that we enforced the signin
requirement using a before filter that called the `signed_in_user` method
([Listing9.12](updating-showing-and-deleting-users.html
#code-authorize_before_filter)). At the time, we only needed that method in
the Users controller, but now we find that we need it in the Microposts
controller as well, so we'll move it into the Sessions helper, as shown in
[Listing10.27](user-microposts.html#code-
sessions_helper_authenticate).9

Listing 10.27. Moving the `signed_in_user` method into the Sessions helper.

`app/helpers/sessions_helper.rb`

    
    module SessionsHelper
      .
      .
      .
      def current_user?(user)
        user == current_user
      end
    
      def signed_in_user
        unless signed_in?
          store_location
          redirect_to signin_url, notice: "Please sign in."
        end
      end
      .
      .
      .
    end
    

To avoid code repetition, you should also remove `signed_in_user` from the
Users controller at this time.

With the code in [Listing10.27](user-microposts.html#code-
sessions_helper_authenticate), the `signed_in_user` method is now available in
the Microposts controller, which means that we can restrict access to the
`create` and `destroy` actions with the before filter shown in
[Listing10.28](user-microposts.html#code-
microposts_controller_access_control). (Since we didn't generate it at the
command line, you will have to create the Microposts controller file by hand.)

Listing 10.28. Adding authentication to the Microposts controller actions.

`app/controllers/microposts_controller.rb`

    
    class MicropostsController < ApplicationController
      before_filter :signed_in_user
    
      def create
      end
    
      def destroy
      end
    end
    

Note that we haven't restricted the actions the before filter applies to since
it applies to them both by default. If we were to add, say, an `index` action
accessible even to non-signed-in users, we would need to specify the protected
actions explicitly:

    
    class MicropostsController < ApplicationController
      before_filter :signed_in_user, only: [:create, :destroy]
    
      def index
      end
    
      def create
      end
    
      def destroy
      end
    end
    

At this point, the tests should pass:

    
    $ bundle exec rspec spec/requests/authentication_pages_spec.rb
    

### 10.3.2 Creating microposts

In Chapter7, we implemented user signup
by making an HTML form that issued an HTTP `POST` request to the `create`
action in the Users controller. The implementation of micropost creation is
similar; the main difference is that, rather than using a separate page at
/microposts/new, we will (following Twitter's convention) put the form on the
Home page itself (i.e., the root path/), as mocked up in
[Figure10.10](user-microposts.html#fig-
home_page_with_micropost_form_mockup).

![home_page_with_micropost_form_mockup_bootstrap](images/figures/home_page_wit
h_micropost_form_mockup_bootstrap.png)

Figure 10.10: A mockup of the Home page with a form for creating
microposts.[(full
size)](http://railstutorial.org/images/figures
/home_page_with_micropost_form_mockup_bootstrap-full.png)

When we last left the Home page, it appeared as in
[Figure5.6](filling-in-the-layout.html#fig-
sample_app_logo)--that is, it had a "Sign up now!" button in the middle. Since
a micropost creation form only makes sense in the context of a particular
signed-in user, one goal of this section will be to serve different versions
of the Home page depending on a visitor's signin status. We'll implement this
in [Listing10.31](user-microposts.html#code-
microposts_home_page) below, but we can still write the tests now. As with the
Users resource, we'll use an integration test:

    
    $ rails generate integration_test micropost_pages
    

The micropost creation tests then parallel those for user creation from
Listing7.16; the
result appears in [Listing10.29](user-microposts.html#code-
microposts_create_tests).

Listing 10.29. Tests for creating microposts.

`spec/requests/micropost_pages_spec.rb`

    
    require 'spec_helper'
    
    describe "Micropost pages" do
    
      subject { page }
    
      let(:user) { FactoryGirl.create(:user) }
      before { sign_in user }
    
      describe "micropost creation" do
        before { visit root_path }
    
        describe "with invalid information" do
    
          it "should not create a micropost" do
            expect { click_button "Post" }.not_to change(Micropost, :count)
          end
    
          describe "error messages" do
            before { click_button "Post" }
            it { should have_content('error') } 
          end
        end
    
        describe "with valid information" do
    
          before { fill_in 'micropost_content', with: "Lorem ipsum" }
          it "should create a micropost" do
            expect { click_button "Post" }.to change(Micropost, :count).by(1)
          end
        end
      end
    end
    

We'll start with the `create` action for microposts, which is similar to its
user analogue ([Listing7.25](sign-up.html#code-
user_create_action)); the principal difference lies in using the
user/micropost association to `build` the new micropost, as seen in
[Listing10.30](user-microposts.html#code-
microposts_create_action).

Listing 10.30. The Microposts controller `create` action.

`app/controllers/microposts_controller.rb`

    
    class MicropostsController < ApplicationController
      before_filter :signed_in_user
    
      def create
        @micropost = current_user.microposts.build(params[:micropost])
        if @micropost.save
          flash[:success] = "Micropost created!"
          redirect_to root_url
        else
          render 'static_pages/home'
        end
      end
    
      def destroy
      end
    end
    

To build a form for creating microposts, we use the code in
[Listing10.31](user-microposts.html#code-
microposts_home_page), which serves up different HTML based on whether the
site visitor is signed in or not.

Listing 10.31. Adding microposts creation to the Home page
(/.

`app/views/static_pages/home.html.erb`

    
    <% if signed_in? %>
      <div class="row">
        <aside class="span4">
          <section>
            <%= render 'shared/user_info' %>
          </section>
          <section>
            <%= render 'shared/micropost_form' %>
          </section>
        </aside>
      </div>  
    <% else %>
      <div class="center hero-unit">
        <h1>Welcome to the Sample App</h1>
    
        <h2>
          This is the home page for the
          <a href="http://railstutorial.org/">Ruby on Rails Tutorial</a>
          sample application.
        </h2>
    
        <%= link_to "Sign up now!", signup_path, 
                                    class: "btn btn-large btn-primary" %>
      </div>
    
      <%= link_to image_tag("rails.png", alt: "Rails"), 'http://rubyonrails.org/' %>
    <% end %> 
    

Having so much code in each branch of the `if`-`else` conditional is a bit
messy, and cleaning it up using partials is left as an exercise
([Section10.5](user-microposts.html#sec-
micropost_exercises)). Filling in the necessary partials from
[Listing10.31](user-microposts.html#code-
microposts_home_page) isn't an exercise, though; we fill in the new Home page
sidebar in [Listing10.32](user-microposts.html#code-
user_info) and the micropost form partial in [Listing10.33
](user-microposts.html#code-micropost_form).

Listing 10.32. The partial for the user info sidebar.

`app/views/shared/_user_info.html.erb`

    
    <a href="<%= user_path(current_user) %>">
      <%= gravatar_for current_user, size: 52 %>
    </a>
    <h1>
      <%= current_user.name %>
    </h1>
    <span>
      <%= link_to "view my profile", current_user %>
    </span>
    <span>
      <%= pluralize(current_user.microposts.count, "micropost") %>
    </span>
    

As in [Listing9.25](updating-showing-and-deleting-
users.html#code-user_index_view), the code in [Listing10.32
](user-microposts.html#code-user_info) uses the version of the `gravatar_for`
helper defined in [Listing7.29](sign-up.html#code-
gravatar_option).

Note that, as in the profile sidebar ([Listing10.20](user-
microposts.html#code-user_show_microposts)), the user info in
Listing10.32
displays the total number of microposts for the user. There's a slight
difference in the display, though; in the profile sidebar, "Microposts" is a
label, and showing "Microposts (1)" makes sense. In the present case, though,
saying "1 microposts" is ungrammatical, so we arrange to display "1 micropost"
(but "2 microposts") using `pluralize`.

We next define the form for creating microposts
(Listing10.33,
which is similar to the signup form in [Listing7.17](sign-
up.html#code-signup_form).

Listing 10.33. The form partial for creating microposts.

`app/views/shared/_micropost_form.html.erb`

    
    <%= form_for(@micropost) do |f| %>
      <%= render 'shared/error_messages', object: f.object %>
      <div class="field">
        <%= f.text_area :content, placeholder: "Compose new micropost..." %>
      </div>
      <%= f.submit "Post", class: "btn btn-large btn-primary" %>
    <% end %>
    

We need to make two changes before the form in
Listing10.33
will work. First, we need to define `@micropost`, which (as before) we do
through the association:

    
    @micropost = current_user.microposts.build
    

The result appears in [Listing10.34](user-microposts.html
#code-micropost_instance_variable).

Listing 10.34. Adding a micropost instance variable to the `home` action.

`app/controllers/static_pages_controller.rb`

    
    class StaticPagesController < ApplicationController
    
      def home
        @micropost = current_user.microposts.build if signed_in?
      end
      .
      .
      .
    end
    

The code in [Listing10.34](user-microposts.html#code-
micropost_instance_variable) has the advantage that it will break the test
suite if we forget to require the user to sign in.

The second change needed to get [Listing10.33](user-
microposts.html#code-micropost_form) to work is to redefine the error messages
partial so that

    
    <%= render 'shared/error_messages', object: f.object %>
    

works. You may recall from [Listing7.22](sign-up.html#code-
f_error_messages) that the error messages partial references the `@user`
variable explicitly, but in the present case we have an `@micropost` variable
instead. We should define an error messages partial that works regardless of
the kind of object passed to it. Happily, the form
variable`f` can access the associated object through
`f.object`, so that in

    
    form_for(@user) do |f|
    

`f.object` is `@user`, and in

    
    form_for(@micropost) do |f|
    

`f.object` is `@micropost`.

To pass the object to the partial, we use a hash with value equal to the
object and key equal to the desired name of the variable in the partial, which
is what this code accomplishes:

    
    <%= render 'shared/error_messages', object: f.object %>
    

In other words, `object: f.object` creates a variable called `object` in the
`error_messages` partial. We can use this object to construct a customized
error message, as shown in [Listing10.35](user-
microposts.html#code-updated_error_messages_partial).

Listing 10.35. Updating the error-messages partial from
Listing7.23 to work
with other objects.

`app/views/shared/_error_messages.html.erb`

    
    <% if object.errors.any? %>
      <div id="error_explanation">
        <div class="alert alert-error">
          The form contains <%= pluralize(object.errors.count, "error") %>.
        </div>
        <ul>
        <% object.errors.full_messages.each do |msg| %>
          <li>* <%= msg %></li>
        <% end %>
        </ul>
      </div>
    <% end %>
    

As this point, the tests in [Listing10.29](user-
microposts.html#code-microposts_create_tests) should be passing:

    
    $ bundle exec rspec spec/requests/micropost_pages_spec.rb
    

Unfortunately, the User request spec is now broken because the signup and edit
forms use the old version of the error messages partial. To fix them, we'll
update them with the more general version, as shown in
[Listing10.36](user-microposts.html#code-
signup_errors_updated) and [Listing10.37](user-
microposts.html#code-edit_errors_updated). (_Note_: Your code will differ if
you implemented [Listing9.50](updating-showing-and-
deleting-users.html#code-new_edit_partial) and [Listing9.51
](updating-showing-and-deleting-users.html#code-new_user_with_partial) from
the exercises in [Section9.6](updating-showing-and-
deleting-users.html#sec-updating_deleting_exercises). [_Mutatis
mutandis_.](http://en.wikipedia.org/wiki/Mutatis_mutandis))

Listing 10.36. Updating the rendering of user signup errors.

`app/views/users/new.html.erb`

    
    <% provide(:title, 'Sign up') %>
    <h1>Sign up</h1>
    
    <div class="row">
      <div class="span6 offset3">
        <%= form_for(@user) do |f| %>
          <%= render 'shared/error_messages', object: f.object %>
          .
          .
          .
        <% end %>
      </div>
    </div>
    

Listing 10.37. Updating the errors for editing users.

`app/views/users/edit.html.erb`

    
    <% provide(:title, "Edit user") %> 
    <h1>Update your profile</h1>
    
    <div class="row">
      <div class="span6 offset3">
        <%= form_for(@user) do |f| %>
          <%= render 'shared/error_messages', object: f.object %>
          .
          .
          .
        <% end %>
    
        <%= gravatar_for(@user) %>
        <a href="http://gravatar.com/emails">change</a>
      </div>
    </div>
    

At this point, all the tests should be passing:

    
    $ bundle exec rspec spec/
    

Additionally, all the HTML in this section should render properly, showing the
form as in [Figure10.11](user-microposts.html#fig-
home_with_form), and a form with a submission error as in
Figure10.12.
You are invited at this point to create a new post for yourself and verify
that everything is working--but you should probably wait until after
Section10.3.3.

!home_with_form_bootstrap

Figure 10.11: The Home page (/ with a new micropost
form.[(full size)](http://railstutorial.org/images/figures
/home_with_form_bootstrap-full.png)

!home_form_errors_bootstrap

Figure 10.12: The home page with form errors.[(full
size)](http://railstutorial.org/images/figures/home_form_errors_bootstrap-
full.png)

### 10.3.3 A proto-feed

The comment at the end of [Section10.3.2](user-
microposts.html#sec-creating_microposts) alluded to a problem: the current
Home page doesn't display any microposts. If you like, you can verify that the
form shown in [Figure10.11](user-microposts.html#fig-
home_with_form) is working by submitting a valid entry and then navigating to
the profile page to see the post, but that's
rather cumbersome. It would be far better to have a _feed_ of microposts that
includes the user's own posts, as mocked up in [Figure10.13
](user-microposts.html#fig-proto_feed_mockup). (In
Chapter11, we'll generalize
this feed to include the microposts of users being _followed_ by the current
user.)

!proto_feed_mockup_bootstrap

Figure 10.13: A mockup of the Home page with a proto-
feed.[(full size)](http://railstutorial.org/images/figures
/proto_feed_mockup_bootstrap-full.png)

Since each user should have a feed, we are led naturally to a `feed` method in
the User model. Eventually, we will test that the feed returns the microposts
of the users being followed, but for now we'll just test that the `feed`
method _includes_ the current user's microposts but _excludes_ the posts of a
different user. We can express these requirements in code with
Listing10.38.

Listing 10.38. Tests for the (proto-)status feed.

`spec/models/user_spec.rb`

    
    require 'spec_helper'
    
    describe User do
      .
      .
      .
      it { should respond_to(:microposts) }
      it { should respond_to(:feed) }
      .
      .
      .
      describe "micropost associations" do
    
        before { @user.save }
        let!(:older_micropost) do 
          FactoryGirl.create(:micropost, user: @user, created_at: 1.day.ago)
        end
        let!(:newer_micropost) do
          FactoryGirl.create(:micropost, user: @user, created_at: 1.hour.ago)
        end
        .
        .
        .
        describe "status" do
          let(:unfollowed_post) do
            FactoryGirl.create(:micropost, user: FactoryGirl.create(:user))
          end
    
          its(:feed) { should include(newer_micropost) }
          its(:feed) { should include(older_micropost) }
          its(:feed) { should_not include(unfollowed_post) }
        end
      end
    end
    

These tests introduce (via the RSpec boolean convention) the array `include?`
method, which simply checks if an array includes the given element:10

    
    $ rails console
    >> a = [1, "foo", :bar]
    >> a.include?("foo")
    => true
    >> a.include?(:bar)
    => true
    >> a.include?("baz")
    => false
    

This example shows just how flexible the RSpec boolean convention is; even
though `include` is already a Ruby keyword (used to include a module, as seen
in, e.g., [Listing8.14](sign-in-sign-out.html#code-
sessions_helper_include)), in this context RSpec correctly guesses that we
want to test array inclusion.

We can arrange for an appropriate micropost `feed` method by selecting all the
microposts with `user_id` equal to the current user's id, which we accomplish
using the `where` method on the `Micropost` model, as shown in
[Listing10.39](user-microposts.html#code-
proto_status_feed).11

Listing 10.39. A preliminary implementation for the micropost status feed.

`app/models/user.rb`

    
    class User < ActiveRecord::Base
      .
      .
      .
      def feed
        # This is preliminary. See "Following users" for the full implementation.
        Micropost.where("user_id = ?", id)
      end
      .
      .
      .
    end
    

The question mark in

    
    Micropost.where("user_id = ?", id)
    

ensures that `id`is properly _escaped_ before being
included in the underlying SQL query, thereby avoiding a serious security hole
called _SQL injection_.
The`id` attribute here is just an integer, so there is no
danger in this case, but _always_ escaping variables injected into SQL
statements is a good habit to cultivate.

Alert readers might note at this point that the code in
Listing10.39
is essentially equivalent to writing

    
    def feed
      microposts
    end
    

We've used the code in [Listing10.39](user-microposts.html
#code-proto_status_feed) instead because it generalizes much more naturally to
the full status feed needed in [Chapter11](following-
users.html#top).

To test the display of the status feed, we first create a couple of microposts
and then verify that a list element (`li`) appears on the page for each one
([Listing10.40](user-microposts.html#code-
home_page_feed_test)).

Listing 10.40. A test for rendering the feed on the Home page.

`spec/requests/static_pages_spec.rb`

    
    require 'spec_helper'
    
    describe "Static pages" do
    
      subject { page }
    
      describe "Home page" do
        .
        .
        .
        describe "for signed-in users" do
          let(:user) { FactoryGirl.create(:user) }
          before do
            FactoryGirl.create(:micropost, user: user, content: "Lorem ipsum")
            FactoryGirl.create(:micropost, user: user, content: "Dolor sit amet")
            sign_in user
            visit root_path
          end
    
          it "should render the user's feed" do
            user.feed.each do |item|
              page.should have_selector("li##{item.id}", text: item.content)
            end
          end
        end
      end
      .
      .
      .
    end
    

[Listing10.40](user-microposts.html#code-
home_page_feed_test) assumes that each feed item has a unique
CSSid, so that

    
    page.should have_selector("li##{item.id}", text: item.content)
    

will generate a match for each item. (Note that the
first`#` in `li##{item.id}` is Capybara syntax for a
CSSid, whereas the second`#` is the
beginning of a Ruby string interpolation`#{}`.)

To use the feed in the sample application, we add an `@feed_items` instance
variable for the current user's (paginated) feed, as in
[Listing10.41](user-microposts.html#code-
feed_instance_variable), and then add a feed partial
(Listing10.42 to
the Home page ([Listing10.44](user-microposts.html#code-
home_with_feed)). (Adding tests for pagination is left as an exercise; see
[Section10.5](user-microposts.html#sec-
micropost_exercises).)

Listing 10.41. Adding a feed instance variable to the `home` action.

`app/controllers/static_pages_controller.rb`

    
    class StaticPagesController < ApplicationController
    
      def home
        if signed_in?
          @micropost  = current_user.microposts.build
          @feed_items = current_user.feed.paginate(page: params[:page])
        end
      end
      .
      .
      .
    end
    

Listing 10.42. The status feed partial.

`app/views/shared/_feed.html.erb`

    
    <% if @feed_items.any? %>
      <ol class="microposts">
        <%= render partial: 'shared/feed_item', collection: @feed_items %>
      </ol>
      <%= will_paginate @feed_items %>
    <% end %>
    

The status feed partial defers the feed item rendering to a feed item partial
using the code

    
    <%= render partial: 'shared/feed_item', collection: @feed_items %>
    

Here we pass a `:collection` parameter with the feed items, which causes
`render` to use the given partial (`'feed_item'` in this case) to render each
item in the collection. (We have omitted the `:partial` parameter in previous
renderings, writing, e.g., `render 'shared/micropost'`, but with a
`:collection` parameter that syntax doesn't work.) The feed item partial
itself appears in [Listing10.43](user-microposts.html#code-
feed_item_partial).

Listing 10.43. A partial for a single feed item.

`app/views/shared/_feed_item.html.erb`

    
    <li id="<%= feed_item.id %>">
      <%= link_to gravatar_for(feed_item.user), feed_item.user %>
      <span class="user">
        <%= link_to feed_item.user.name, feed_item.user %>
      </span>
      <span class="content"><%= feed_item.content %></span>
      <span class="timestamp">
        Posted <%= time_ago_in_words(feed_item.created_at) %> ago.
      </span>
    </li>
    

Listing10.43
also adds a CSSid for each feed item using

    
    <li id="<%= feed_item.id %>">
    

as required by the test in [Listing10.40](user-
microposts.html#code-home_page_feed_test).

We can then add the feed to the Home page by rendering the feed partial as
usual ([Listing10.44](user-microposts.html#code-
home_with_feed)). The result is a display of the feed on the Home page, as
required ([Figure10.14](user-microposts.html#fig-
home_with_proto_feed)).

Listing 10.44. Adding a status feed to the Home page.

`app/views/static_pages/home.html.erb`

    
    <% if signed_in? %>
      <div class="row">
        .
        .
        .
        <div class="span8">
          <h3>Micropost Feed</h3>
          <%= render 'shared/feed' %>
        </div>
      </div>
    <% else %>
      .
      .
      .
    <% end %>
    

![home_with_proto_feed_bootstrap](images/figures/home_with_proto_feed_bootstra
p.png)

Figure 10.14: The Home page (/ with a proto-
feed.[(full size)](http://railstutorial.org/images/figures
/home_with_proto_feed-full.png)

At this point, creating a new micropost works as expected, as seen in
Figure10.15.
There is one subtlety, though: on _failed_ micropost submission, the Home page
expects an `@feed_items` instance variable, so failed submissions currently
break (as you should be able to verify by running your test suite). The
easiest solution is to suppress the feed entirely by assigning it an empty
array, as shown in [Listing10.45](user-microposts.html
#code-microposts_create_action_with_feed).12

!micropost_created_bootstrap

Figure 10.15: The Home page after creating a new
micropost.[(full
size)](http://railstutorial.org/images/figures/micropost_created_bootstrap-
full.png)

Listing 10.45. Adding an (empty) `@feed_items` instance variable to the
`create` action.

`app/controllers/microposts_controller.rb`

    
    class MicropostsController < ApplicationController
      .
      .
      .
      def create
        @micropost = current_user.microposts.build(params[:micropost])
        if @micropost.save
          flash[:success] = "Micropost created!"
          redirect_to root_url
        else
          @feed_items = []
          render 'static_pages/home'
        end
      end
      .
      .
      .
    end
    

At this point, the proto-feed should be working, and the test suite should
pass:

    
    $ bundle exec rspec spec/
    

### [10.3.4 Destroying microposts](user-microposts.html#sec-
destroying_microposts)

The last piece of functionality to add to the Microposts resource is the
ability to destroy posts. As with user deletion
([Section9.4.2](updating-showing-and-deleting-users.html
#sec-the_destroy_action)), we accomplish this with "delete" links, as mocked
up in [Figure10.16](user-microposts.html#fig-
micropost_delete_links_mockup). Unlike that case, which restricted user
destruction to admin users, the delete links will work only for microposts
created by the current user.

![micropost_delete_links_mockup_bootstrap](images/figures/micropost_delete_lin
ks_mockup_bootstrap.png)

Figure 10.16: A mockup of the proto-feed with micropost delete
links.[(full size)](http://railstutorial.org/images/figures
/micropost_delete_links_mockup_bootstrap-full.png)

Our first step is to add a delete link to the micropost partial as in
[Listing10.43](user-microposts.html#code-
feed_item_partial), and while we're at it we'll add a similar link to the feed
item partial from [Listing10.43](user-microposts.html#code-
feed_item_partial). The results appear in [Listing10.46
](user-microposts.html#code-micropost_partial_with_delete) and
[Listing10.47](user-microposts.html#code-
feed_item_partial_with_delete). (The two cases are almost identical, and
eliminating this duplication is left as an exercise
([Section10.5](user-microposts.html#sec-
micropost_exercises)).)

Listing 10.46. Adding a delete link to the micropost partial.

`app/views/microposts/_micropost.html.erb`

    
    <li>
      <span class="content"><%= micropost.content %></span>
      <span class="timestamp">
        Posted <%= time_ago_in_words(micropost.created_at) %> ago.
      </span>
      <% if current_user?(micropost.user) %>
        <%= link_to "delete", micropost, method: :delete,
                                         data: { confirm: "You sure?" },
                                         title: micropost.content %>
      <% end %>
    </li>
    

Listing 10.47. The feed item partial with added delete link.

`app/views/shared/_feed_item.html.erb`

    
    <li id="<%= feed_item.id %>">
      <%= link_to gravatar_for(feed_item.user), feed_item.user %>
        <span class="user">
          <%= link_to feed_item.user.name, feed_item.user %>
        </span>
        <span class="content"><%= feed_item.content %></span>
        <span class="timestamp">
          Posted <%= time_ago_in_words(feed_item.created_at) %> ago.
        </span>
      <% if current_user?(feed_item.user) %>
        <%= link_to "delete", feed_item, method: :delete,
                                         data: { confirm: "You sure?" },
                                         title: feed_item.content %>
      <% end %>
    </li>
    

The test for destroying microposts uses Capybara to click the "delete" link
and expects the Micropost count to decrease by1
([Listing10.48](user-microposts.html#code-
micropost_destroy_specs)).

Listing 10.48. Tests for the Microposts controller `destroy` action.

`spec/requests/micropost_pages_spec.rb`

    
    require 'spec_helper'
    
    describe "Micropost pages" do
      .
      .
      .
      describe "micropost destruction" do
        before { FactoryGirl.create(:micropost, user: user) }
    
        describe "as correct user" do
          before { visit root_path }
    
          it "should delete a micropost" do
            expect { click_link "delete" }.to change(Micropost, :count).by(-1)
          end
        end
      end
    end
    

The application code is also analogous to the user case in
[Listing9.48](updating-showing-and-deleting-users.html
#code-admin_destroy_before_filter); the main difference is that, rather than
using an `admin_user` before filter, in the case of microposts we have a
`correct_user` before filter to check that the current user actually has a
micropost with the given id. The code appears in
[Listing10.49](user-microposts.html#code-
microposts_destroy_action), and the result of destroying the second-most-
recent post appears in [Figure10.17](user-microposts.html
#fig-home_post_delete).

Listing 10.49. The Microposts controller `destroy` action.

`app/controllers/microposts_controller.rb`

    
    class MicropostsController < ApplicationController
      before_filter :signed_in_user, only: [:create, :destroy]
      before_filter :correct_user,   only: :destroy
      .
      .
      .
      def destroy
        @micropost.destroy
        redirect_to root_url
      end
    
      private
    
        def correct_user
          @micropost = current_user.microposts.find_by_id(params[:id])
          redirect_to root_url if @micropost.nil?
        end
    end
    

In the `correct_user` before filter, note that we find microposts _through_
the association:

    
    current_user.microposts.find_by_id(params[:id])
    

This automatically ensures that we find only microposts belonging to the
current user. In this case, we use `find_by_id` instead of `find` because the
latter raises an exception when the micropost doesn't exist instead of
returning `nil`. By the way, if you're comfortable with exceptions in Ruby,
you could also write the `correct_user` filter like this:

    
    def correct_user
      @micropost = current_user.microposts.find(params[:id])
    rescue
      redirect_to root_url
    end
    

It might occur to you that we could implement the `correct_user` filter using
the `Micropost` model directly, like this:

    
    @micropost = Micropost.find_by_id(params[:id])
    redirect_to root_url unless current_user?(@micropost.user)
    

This would be equivalent to the code in [Listing10.49
](user-microposts.html#code-microposts_destroy_action), but, as explained by
Wolfram Arnold in the blog post [Access Control
101 in Rails and the Citibank Hack](http://www.rubyfocus.biz/blog/2011/06/15
/access_control_101_in_rails_and_the_citibank-hack.html), for security
purposes it is a good practice always to run lookups through the association.

!home_post_delete_bootstrap

Figure 10.17: The user home page after deleting the second-most-recent
micropost.[(full
size)](http://railstutorial.org/images/figures/home_post_delete_bootstrap-
full.png)

With the code in this section, our Micropost model and interface are complete,
and the test suite should pass:

    
    $ bundle exec rspec spec/
    

## 10.4 Conclusion

With the addition of the Microposts resource, we are nearly finished with our
sample application. All that remains is to add a social layer by letting users
follow each other. We'll learn how to model such user relationships, and see
the implications for the status feed, in [Chapter11
](following-users.html#top).

Before proceeding, be sure to commit and merge your changes if you're using
Git for version control:

    
    $ git add .
    $ git commit -m "Add user microposts"
    $ git checkout master
    $ git merge user-microposts
    $ git push
    

You can also push the app up to Heroku at this point. Because the data model
has changed through the addition of the `microposts` table, you will also need
to migrate the production database:

    
    $ git push heroku
    $ heroku pg:reset <DATABASE>
    $ heroku run rake db:migrate
    $ heroku run rake db:populate
    

Follow the instructions in [Section9.5](updating-showing-
and-deleting-users.html#sec-updating_and_deleting_users_conclusion) to find
the right replacement for the `DATABASE` argument in the second command above.

## 10.5 Exercises

We've covered enough material now that there is a combinatorial explosion of
possible extensions to the application. Below are just a few of the many
possibilities.

  1. Add tests for the sidebar micropost counts (including proper pluralization).
  2. Add tests for micropost pagination.
  3. Refactor the Home page to use separate partials for the two branches of the `if`-`else` statement.
  4. Write a test to make sure delete links do not appear for microposts not created by the current user.
  5. Using partials, eliminate the duplication in the delete links from Listing10.46](user-microposts.html#code-micropost_partial_with_delete) and [Listing10.47.
  6. Very long words currently break our layout, as shown in Figure10.18](user-microposts.html#fig-long_word_micropost). Fix this problem using the `wrap` helper defined in [Listing10.50](user-microposts.html#code-wrap). Note the use of the `raw` method to prevent Rails from escaping the resulting HTML, together with the `sanitize` method needed to prevent cross-site scripting. This code also uses the strange-looking but useful _ternary operator_ ([Box10.1.
  7. **(challenging)** Add a JavaScript display to the Home page to count down from140 characters.

Box 10.1.10 types of people

There are 10 kinds of people in the world: Those who like the ternary
operator, those who don't, and those who don't know about it. (If you happen
to be in the third category, soon you won't be any longer.)

When you do a lot of programming, you quickly learn that one of the most
common bits of control flow goes something like this:

    
      if boolean? 
        do_one_thing 
      else 
        do_something_else 
      end 

Ruby, like many other languages (including C/C++, Perl, PHP, and Java), allows
you to replace this with a much more compact expression using the _ternary
operator_ (so called because it consists of three parts):

    
      boolean? ? do_one_thing : do_something_else 

You can also use the ternary operator to replace assignment:

    
      if boolean?
        var = foo
      else 
        var = bar
      end 

becomes

    
      var = boolean? ? foo : bar

Another common use is in a function's return value:

    
      def foo
        do_stuff
        boolean? ? "bar" : "baz"
      end

Since Ruby implicitly returns the value of the last expression in a function,
here the `foo` method returns `"bar"` or `"baz"` depending on the value of
`boolean?`. It is this final construction that appears in
Listing10.50.

![long_word_micropost_bootstrap](images/figures/long_word_micropost_bootstrap.
png)

Figure 10.18: The (broken) site layout with a particularly long
word.[(full size)](http://railstutorial.org/images/figures
/long_word_micropost_bootstrap-full.png)

Listing 10.50. A helper to wrap long words.

`app/helpers/microposts_helper.rb`

    
    module MicropostsHelper
    
      def wrap(content)
        sanitize(raw(content.split.map{ |s| wrap_long_string(s) }.join(' ')))
      end
    
      private
    
        def wrap_long_string(text, max_width = 30)
          zero_width_space = "&#8203;"
          regex = /.{1,#{max_width}}/
          (text.length < max_width) ? text : 
                                      text.scan(regex).join(zero_width_space)
        end
    end
    

[ «Chapter 9 Updating, showing, and deleting users
](updating-showing-and-deleting-users.html#top) [ Chapter 11 Following
users» ](following-users.html#top)

  1. Technically, we treated sessions as a resource in Chapter8, but sessions are not saved to the database the way users and microposts are.↑
  2. The `content` attribute will be a `string`, but, as noted briefly in Section2.1.2, for longer text fields you should use the `text` data type.↑
  3. Initially, I forgot to duplicate the microposts property, and in fact previous versions of the tutorial had a bug in this test. Having a safety check would have revealed the error. Thanks to alert reader Jacob Turino for catching the bug and bringing it to my attention.↑
  4. (i.e., the five users with custom Gravatars, and one with the default Gravatar)↑
  5. Tail your `log/development.log` file if you're curious about the SQL this method generates.↑
  6. By design, the Faker gem's _lorem ipsum_ text is randomized, so the contents of your sample microposts will differ.↑
  7. For convenience, Listing10.24 actually has _all_ the CSS needed for this chapter.↑
  8. The other two resources are Users in Section7.2](sign-up.html#sec-signup_form) and Sessions in [Section8.1.↑
  9. We noted in Section8.2.1](sign-in-sign-out.html#sec-remember_me) that helper methods are available only in _views_ by default, but we arranged for the Sessions helper methods to be available in the controllers as well by adding `include SessionsHelper` to the Application controller ([Listing8.14.↑
  10. Learning about methods such as `include?` is one reason why, as noted in Section1.1.1, I recommend reading a pure Ruby book after finishing this one.↑
  11. See the Rails Guide on the Active Record Query Interface for more on `where` and the like.↑
  12. Unfortunately, returning a paginated feed doesn't work in this case. Implement it and click on a pagination link to see why.↑

