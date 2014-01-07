---
layout: post
title: Generate a Rails 3.2 project with RSpec & FactoryGirl
enki_id: 9
tags: rails rspec factory_girl
---

If you are starting a new Rails project and you want to use RSpec and FactoryGirl instead of Test::Unit and Fixtures, it's easy to get Rails to recognize this and generate the correct files for you.

1. Create a new rails app without the test directory (or if the app already exists, just delete the test directory)

    ```bash
    $ rails new myproject -T
	```

2. Add the gems to the `:development` and `:test` groups in your Gemfile

	```ruby
    group :test, :development do
      gem 'rspec-rails'
      gem 'factory_girl_rails'
    end
	```

That's it! Now when you generate your models and controllers using `$ rails g model my_model` you will also get the spec and factory created for free.
