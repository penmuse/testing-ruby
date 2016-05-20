##5 Potential problems and solutions when using RSpec with Rails 5

Few days ago, I ventured into using Rails 5 for a project. I kept getting lots of deprecation warnings and errors. After a while, I was able to fix them all but not without troubles. So, I decided to help out anyone who might be in same condition. Here are 5 possible problems and solutions for those using Rails 5 with RSpec.

###1. use transactional fixtures vs use transactional tests

####Problem
You have `gem rspec-rails` in your Gemfile and run `rspec`, you get the error:

```
DEPRECATION WARNING: use_transactional_fixtures= is deprecated and will be removed from Rails 5.1 (use use_transactional_tests= instead)
```

####Solution

You may try to follow the advice and go to rails_helper.rb to change `config.use_transactional_fixtures = true` to `config.use_transactional_tests = true`. This is the error you'd get

```
(NoMethodError)
Did you mean?  use_transactional_fixtures=
```

This is because what Rails 5 is asking for is not yet implemented in the RSpec gem at this moment.

**Verdict**: Add RSpec from master to your Gemfile thus:

```ruby
%w[rspec-core rspec-expectations rspec-mocks rspec-rails rspec-support].
  each do |lib|
  gem lib, git: "https://github.com/rspec/#{lib}.git", branch: "master"
end
```

**Caveat**: Running bundle install becomes very slow!

###2. Keyword arguments in tests

####Problem

You might be testing say a controller and you write the test thus:

```ruby
  it "redirects to moderator" do
    post :create, email: moderator.email, password: moderator.password

    expect(response).to redirect_to(moderator_path(moderator.id))
  end
```

You get the error

```
DEPRECATION WARNING: ActionController::TestCase HTTP request methods will accept only
keyword arguments in future Rails versions.
```

####Solution

Keywords should be used for every argument. By so doing, irrespective of your arrangement of the arguments it will still work. For example 

```ruby
it "redirects to moderator" do
  process :create, method: :post, params: { moderator: {
    email: moderator.email,
    password: moderator.password
  } }

  expect(response).to redirect_to(moderator_path(moderator.id))
end
```

or

```ruby
it "redirects to moderator" do
  post :create, params: { moderator: {
    email: moderator.email,
    password: moderator.password
  } }

  expect(response).to redirect_to(moderator_path(moderator.id))
end
```

Notice keywords like `process`, `method` and `params` in the first solution and `params` in the second and `moderator` being explicitly called for both. These are the keywords. You could read more about `kwargs`.

**Verdict**: I use the second option since the first is too verbose for me. If it's simple and correct, it's better for me.

###3. Extraction of some methods to another gem

####Problem

You may try to set expectation or (assert) using methods like `assigns`, `render_template` and so on.

```ruby
expect(response).to redirect_to(moderator_path(assigns(:moderator)))
```
or

```ruby
expect(response).to render_template("moderators/show")
```

You get error messages 

```
NoMethodError:
 assigns has been extracted to a gem. To continue using it,
  add `gem 'rails-controller-testing'` to your Gemfile.
```
for the first. The second will be

```
NoMethodError:
  assert_template has been extracted to a gem. To continue using it,
    add `gem 'rails-controller-testing'` to your Gemfile.
```

####Solution

Just follow the error message. It helps always. Simply add `gem rails-controller-testing` to your Gemfile.

###4. Not catering for associations

####Problem

You may have a `User` model which has many `Microposts` so you have a factory for microposts thus,

```ruby
FactoryGirl.define do
  factory :micropost do
    content Faker::Hipster.paragraphs(1)
  end
end
```

This would pass in Rails 4 but in Rails 5, you get the error,

```
ActiveRecord::RecordInvalid:
  Validation failed: User must exist
```

####Solution

Rails 5 enforces that you keep your promise. Since you declared that micropost `belongs_to :user` and so, user `has_many :microposts`, why wasn't this stated in the factory? No micropost deserves to be hanging in thin air without a parent, eh?

Simply add `user` to your factory if your user factory goes by that name

```ruby
FactoryGirl.define do
  factory :micropost do
    content Faker::Hipster.paragraphs(1)
    user
  end
end
```

That solves the problem.

###5. Getting too many deprecation errors/warnings than you can bear

####Problem
As you continue with Rails 5 and RSpec, it's possible you find more errors other than the ones listed above.

####Solution
You could stay with Rails 4 until you're ready to take the big leap to Rails 5. However, you'd have to move to Rails 5 someday as older versions of Rails fall into disuse. Like I said, the deprecation messages are often descriptive, use them well!
