---
layout:     post
title:      "Input fields in rake task"
date:       2014-09-21
---

Rake is an awesome tool to automate workflows during development or for some other stuff.

But lets say we have a task called `customer:add` where we want to assign name
and email to the new customer. In this case we must be able to interact with
our task.

So we have this rake file as you can se below which just adds an new customer to
an YAML file but they all will have the same name and email until we manually
changes the file. Lets take a look how we could assign then while running the
taks in next section.

```ruby
# Rakefile
require "yaml"

namespace :customer do
  task :add do
    name = "Example name"
    email = "Example email"

    # TODO: Here we need to customize the attributes

    storage = 'customers.yml'
    yaml = load_yaml_storage(storage)
    yaml["customers"] ||= [] # Make sure the key customer is defined
    yaml["customers"] << { # Adds the new customer
      name: name,
      email: email
    }
    File.write(storage, yaml.to_yaml) # Saves the file with new data
  end
end

def load_yaml_storage(storage)
  yaml = YAML.load_file(storage) # Loads the yaml file
  yaml or {}
rescue Errno::ENOENT # If the file doesn't exists we return an empty hash
  {}
end
```

## Add input fields to our task

The example above is an working one so you can copy it and paste it into your
favorite editor and play along. To interact with the data before it's written to
the file we can use `STDIN` from ruby.

Lets try it by starting `irb` from the terminal and test the code below

```ruby
value = STDIN.gets
# Ruby will now wait for you to write something and press enter, try with Hello, World!

value.inspect
# => "Hello, World!\n"
```

As you can se the value will also include an `\n` which we don't want. This can
be removed with

```ruby
value.chomp
# => "Hello, World!"
```

So if we take our new experience from this and add it to the Rakefile it could
look something like

```ruby
# Rakefile
require "yaml"

namespace :customer do
  task :add do
    puts "What's the customer name?"
    name = STDIN.gets.chomp

    puts "And email?"
    email = STDIN.gets.chomp

    storage = 'customers.yml'
    yaml = load_yaml_storage(storage)
    yaml["customers"] ||= [] # Make sure the key customer is defined
    yaml["customers"] << { # Adds the new customer
      name: name,
      email: email
    }
    File.write(storage, yaml.to_yaml) # Saves the file with new data
  end
end

def load_yaml_storage(storage)
  yaml = YAML.load_file(storage) # Loads the yaml file
  yaml or {}
rescue Errno::ENOENT # If the file doesn't exists we return an empty hash
  {}
end
```

It should now be working. Lets try it out by running `rake customer:add` from
your terminal.

![/img/posts/2014-09-21/result.gif](/img/posts/2014-09-21/result.gif)

Remember, you can add as many customers as you want. `STDIN` is pure ruby so
this example could be applied to other ruby scripts as well.
