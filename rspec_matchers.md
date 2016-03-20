# Creating your own RSpec Matchers

RSpec is a behavior-driven development (BDD) framework for the Ruby programming language. It assists with setting expectations for how exactly one expects ones code to behave. Tests (or specs) are first written and run leading to failures. Code is then written to pass those expectations earlier set.

RSpec by default provides a handful of matchers that can help you test your code. Matchers are sets of expressions which are used to compare between actual and expected values. The actual value is what is returned from the written code while the expected value is the value that satisfies the condition in the matcher.

### Why write your own matchers

Sometimes you might have written your code and could not find a straightforward matcher to test it. An example is in Rails Application when you like to test if a model class ```User``` has many ```Microposts``` i.e 

```ruby
class User < ActiveRecord::Base
  has_many :microposts
  ...
end
```

In your tests you may have to first create a new instance of a ```user``` using ```user = User.create(*args)``` and test if ```user.microposts``` exists. So for every other model class relations, say tens or hundreds of them, you have to continue this way. Hey!

You could abstract this in a matcher and write simple one-liner tests as you can find in [shoulda_matchers](https://github.com/thoughtbot/shoulda-matchers). Thus, you have DRY tests that strongly describe the behaviour of your code.

Afterwards your tests may read simply

```ruby

  it { should have_many :microposts }

```

### Examples of Default Matchers in RSpec

####  1. Equality
        expect(actual).to eq(expected)
        expect(actual).to eql(expected)
        expect(actual).not_to eql(not_expected)

####  2. Comparisons
        expect(actual).to be >  expected
        expect(actual).to be >= expected
        expect(actual).to be <= expected

In the above examples,the equality matcher returns true if both actual and expected values are the same else it returns false. As for the examples on comparisons, the actual value is expected to match the condition expressed in the matchers (>, >= or <=).

To build you own matchers, you need to have rspec installed.

  run ```gem install rspec```

Then create the directory for this experiment and name it ```rspec_matchers``` or whatever name you choose.

Enter into the new directory,  ```cd rspec_matchers``` and run ```rspec --init``` to generate ```spec_helper.rb``` and ```.rspec``` files. We will be doing our configuration in the ```spec_helper.rb``` file while arguments you can run on your terminal with ```rspec``` command should reside in the ```.rspec``` file.

#### Writing your Matchers

We'll be creating matchers to determine whether a given number is a square.

At the bottom of ```spec_helper.rb```, add ```require "rspec/expectations"``` and paste the following:

```ruby
RSpec::Matchers.define :be_the_square_of do |expected_value|
  match do |actual_value|
    expected_value**2 == actual_value
  end
end

RSpec::Matchers.define :not_be_the_square_of do |expected_value|
  match do |actual_value|
    expected_value**2 != actual_value
  end
end
```

In the above code, we simply used the define method in `RSpec` for defining a new matcher ```RSpec::Matchers.define``` which takes two arguments:

1. The name of the matcher to be defined eg :not_to_be_square
2. A block which has another method match with a block wherein the relationship between actual and expected values is described.


#### Writing your tests

Create a file called ```square_spec.rb``` in ```spec``` directory and paste the following tests into it:

```ruby

require 'spec_helper'

describe 'Squares' do
  context 2**2 do
    it { should be_the_square_of 2 }
  end

  context 9 do
    it { should be_the_square_of 3 }
  end

  context 20 do
    it { should not_be_the_square_of 4 }
  end

  # another way using the default message
  context '49' do
    it do
      expect(49).to not_be_the_square_of 8
    end
  end

  context '49' do
    it do
      expect(49).to be_the_square_of 7
    end
  end

  # override the default message

  context '2025' do
    it 'is big but still the square of 45' do
      expect(2025).to be_the_square_of 45
    end
  end
end
```

You can see that the name we give to our matcher will determine how the default message is. The 4th and 5th tests follow the the same message as the earlier ones and is very descriptive.  You can override the message as in the last.

#### Running your tests

Open the ```.rspec``` in your ```rspec_matchers``` directory. You should find this two commands by default

    --color                    # This adds color to your tests
    --require spec_helper      # requires spec_helper

    Add a third:

    -fd                 # the test will run with full documentation


#### Conclusion

We can do more of this next time. At the moment, enjoy your tests.


Check the test  code on [github](https://github.com/andela-ydaniju/rspec_matchers)
