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

Now once we run the migration, let's see if we can create a User with no name:

```rails
rails console
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
