# Model functionality and testing

### Setup

If you want to start from this chapter, you can just git clone this branch down.
It has all the files that we created from chapter 2.

### Model methods and validations

Like we've already seen, models are just Ruby classes that talk to the database,
store and validate data, perform the business logic and otherwise do the heavy lifting.

Rails models are the point of entry to the database. In these classes, we have
access to the database via ActiveRecord, and it is where we write methods like
`belongs_to` and `has_many` which suggest to ActiveRecord how to generate the
correct SQL to query the appropriate rows in the actual database.

Let's add some model methods, validations, a callback, and some tests to give
an idea of some basic model functionality.

### Model methods

Say we want to create a method that returns us the user's first initial.
Let's write this method in our model:

```rails
class User < ApplicationRecord
  def first_initial
    name[0]
  end
end
```

Let's test this out:

```rails
rails console
[1] pry(main)> user = User.create(name: "Jeffe")
   (0.1ms)  BEGIN
  SQL (0.6ms)  INSERT INTO "users" ("name", "created_at", "updated_at") VALUES ($1, $2, $3) RETURNING "id"  [["name", "Jeffe"], ["created_at", "2017-06-19 15:28:42.294988"], ["updated_at", "2017-06-19 15:28:42.294988"]]
   (6.0ms)  COMMIT
=> #<User:0x007fb7e2f4cf80 id: 2, name: "Jeffe", created_at: Mon, 19 Jun 2017 15:28:42 UTC +00:00, updated_at: Mon, 19 Jun 2017 15:28:42 UTC +00:00>
[2] pry(main)> user.first_initial
=> "J"
```

So we created a `User` with a name of "Jeffe", assigned it to a variable called
`user`, and then called `first_initial` on the user. That returned us the
string "J". That seems to work!

But what if we create a user with no name and then call this method? What would happen?

```rails
rails console
user = User.create
  (0.2ms)  BEGIN
 SQL (0.3ms)  INSERT INTO "users" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", "2017-06-19 15:31:05.343247"], ["updated_at", "2017-06-19 15:31:05.343247"]]
  (1.9ms)  COMMIT
=> #<User:0x007fb7e2e5ebc8 id: 4, name: nil, created_at: Mon, 19 Jun 2017 15:31:05 UTC +00:00, updated_at: Mon, 19 Jun 2017 15:31:05 UTC +00:00>
[5] pry(main)> user.first_initial
NoMethodError: undefined method `[]' for nil:NilClass
from /Users/jeffrey.wan/dev/rails_5_ba_tutorial/app/models/user.rb:3:in `first_initial'
user.delete
```

So that throws an error because the user has no name (it's nil actually) and we're trying
to call the array method (`[]`) with an argument of 0 (that's really what [0] is doing) on
nil. That `[]` method doesn't exist on nil.

This would throw an error in our application at runtime which is not a good thing.

So what can we do?

We can do a couple things. We can write a database-level guard to prevent users from
being created with no name and we can write an application-level guard to prevent users
from being created with no name. Let's do them both.

Let's write our database-level guard:

First in terminal (`g` is short for generate):

```bash
rails g migration ChangeNameOnUsers
```

And then let's edit our new migration in the `db/migrate` folder:

```rails
class ChangeNameOnUsers < ActiveRecord::Migration[5.0]
  def change
    change_column :users, :name, :string, null: false
  end
end
```

If we run rake `db:migrate`, we should see this in our structure.sql file:

```
CREATE TABLE users (
    id integer NOT NULL,
    name character varying NOT NULL,
    created_at timestamp without time zone NOT NULL,
    updated_at timestamp without time zone NOT NULL
);
```

Note: You might get errors when running rake db:migrate because of already pre-existing
users with nil in their name. If that happens, just go into your Rails console and clear
our your data:

```rails
rails console
OrderProduct.destroy_all
Order.destroy_all
User.destroy_all
```

Now once we run the migration, let's see if we can create a User with no name.
In `rails console`:

```rails
User.create
```

You should see:

```
User.create
   (0.1ms)  BEGIN
  SQL (7.3ms)  INSERT INTO "users" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", "2017-06-19 16:02:46.074883"], ["updated_at", "2017-06-19 16:02:46.074883"]]
   (0.2ms)  ROLLBACK
ActiveRecord::StatementInvalid: PG::NotNullViolation: ERROR:  null value in column "name" violates not-null constraint
User.count
   (0.6ms)  SELECT COUNT(*) FROM "users"
=> 0
```

We tried creating a user with no name, and it fails. An error is raised instead and
the count of Users is 0 after our failed attempt. Perfect!

Let's also add a Rails validation to ensure that created users have a name. Why do we need
both? Even though the database validation is sufficient to ensure that we do not
ever create "bad" data, the rails-level validation can provide an application
user useful error messages when they try to create "bad" data. More can be read [here](https://robots.thoughtbot.com/validation-database-constraint-or-both)

In our `User` model, let's write change our model to this:

```rails
class User < ApplicationRecord
  has_many :orders

  validates :name, presence: true
end
```

In `rails console`, run this:

```rails
user = User.create
   (0.1ms)  BEGIN
   (0.1ms)  ROLLBACK
=> #<User:0x007fb7eaa770e8 id: nil, name: nil, created_at: nil, updated_at: nil>
[5] pry(main)> user.errors.full_messages
=> ["Name can't be blank"]
```

Our application validation works? For whatever, reason, what if you don't want
your friend "Donald" to register in our application. Perhaps you don't like anyone
named "Donald" too. Well, we can write a custom validation for that. More on
custom validations [here](http://guides.rubyonrails.org/active_record_validations.html#performing-custom-validations)

Let's change our `User` model again:

```rails
class User < ApplicationRecord
  has_many :orders

  validates :name, presence: true
  validate :not_named_donald

  private

  def not_named_donald
    if name == "donald"
      errors.add(:name, "get away donald")
    end
  end
end
```

We created a custom validation method and we made it private because it would
only be used internally to this class. Now let's try to create a user named "donald":

In `rails console`:

```bash
user = User.create(name: "donald")
   (0.1ms)  BEGIN
   (0.1ms)  ROLLBACK
=> #<User:0x007fb7e51b2f48 id: nil, name: "donald", created_at: nil, updated_at: nil>
[2] pry(main)> user.errors[:name]
=> ["get away donald"]
```

But there's an obvious loophole here. What if someone's name was "Donald" with a
capitalized "D"? Our logic doesn't capture this. We could extend our custom
validator method to check to names that are == "Donald". However, then we have
database mixed with lowercase and uppercase names which might make
querying for them more complicated in the far future. Why not just clean
the data before it enters our database?

We can use built-in callback hooks to do some cleaning for us. These hooks
inject logic around the model object's creation, updating, and deletion methods
(the object's lifecycle). This is a way for us to have greater control over
the data that enters our database. (These lifecycle hooks are a good example
of the Observer pattern if you're so into design patterns). More about
these callbacks can be read [here](http://guides.rubyonrails.org/active_record_callbacks.html)

Let's write one! Let's create a `before_validation` callback that fires before a
user's validations are run (and therefore before a user is created):

```rails
class User < ApplicationRecord
  has_many :orders

  validates :name, presence: true
  validate :not_named_donald

  before_validation :normalize_name, on: :create

  private

  def not_named_donald
    if name == "donald"
      errors.add(:name, "get away donald")
    end
  end

  def normalize_name
    self.name = name.downcase
  end
end
```

Now let's try create a user named "Donald"

```rails
[4] pry(main)> user = User.new(name: "Donald")
=> #<User:0x007fb7e326ba68 id: nil, name: "Donald", created_at: nil, updated_at: nil>
[5] pry(main)> user.name
=> "Donald"
[6] pry(main)> user.valid? #this calls the validations
=> false
[7] pry(main)> user.errors[:name]
=> ["get away donald"]
```

(Do note that `new` does not attempt to save the new object to the database. Also,
`valid?` runs the validations which allows us to see what errors, if any, occur.
When you call `create` an object instead of `new`, under the hood it calls `new`, then
runs the validations, then saves the object if there are no errors on the object.
An object can only be saved if there are no errors on the object (i.e., it is valid)).

Now, creating a user with a name of "Donald" fails validations like we want and
adds our custom message to the user object.



### Adding RSpec tests

We have a file in our `spec/models` folder (where we keep our model specs) called
`user_spec.rb`. Let's open it up. We should see:

```rails
require 'rails_helper'

RSpec.describe User, type: :model do
  pending "add some examples to (or delete) #{__FILE__}"
end
```

#### TODO
