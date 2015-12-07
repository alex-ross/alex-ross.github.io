---
layout: post
title:  "Configure RuboCop for your project"
date:   2015-12-07 19:24
categories: ruby rubocop
---

> RuboCop is a Ruby static code analyzer. Out of the box it will enforce many of the guidelines outlined in the community Ruby Style Guide.

-- From their [GitHub README.md](rubocop)

We use RuboCop for most of our projects at [Standout AB](standout) . It will
help the team to follow the style guide we all agreed on. If we change our
opinion it is also easy to just change the `.rubocop.yml` file to match it.

Installing and configure RuboCop is pretty simple if you are following [this
Ruby Style Guide](ruby_style_guide).

Just add it to your `Gemfile`

```ruby
gem "rubocop", require: false
```

And then run `bundle install` in your terminal to install it.

You will already be able to run `rubocop` in the terminal to analyze your code.

## Project specific configurations

If you're not completely satisfied with the Ruby Style Guide as is you can
create a file called `.rubocop.yml` and add your settings there. Take a look at
the gem's [default configuration files](default_config) to get an idea of what
you can change to match your or your's team coding style.

Our configuration file looks like this:

```yaml
# .rubocop.yml

AllCops:
  RunRailsCops: true
  DisplayCopNames: true
  DisplayStyleGuide: true
  Exclude:
    - "bin/*"
    - Capfile
    - config/boot.rb
    - config/environment.rb
    - config/initializers/version.rb
    - db/schema.rb
    - Gemfile
    - Guardfile
    - Rakefile

Metrics/LineLength:
  Exclude:
    - "config/**/*"

Style/ClassAndModuleChildren:
  Enabled: false

Style/Documentation:
  Enabled: false

Style/DoubleNegation:
  Enabled: false

Style/SpaceAroundEqualsInParameterDefault:
  EnforcedStyle: no_space

Style/StringLiterals:
  EnforcedStyle: double_quotes

Style/TrivialAccessors:
  AllowPredicates: true

Style/SingleSpaceBeforeFirstArg:
  Enabled: false
```

Feel free to copy and change. You can also get the up to date version with curl
at any time.

```bash
curl -o .rubocop.yml https://raw.githubusercontent.com/standout/Coding/master/ruby/rubocop.yml
```

## Ignore current errors for bigger projects

If you're not starting a new project you may find out that you already have a
lot of errors and warnings when you run `rubocop`. If you have a huge code base
and not the time to fix it all now, you can tell RuboCop to create you a todo
file. The point of this file is to ignore all current errors and you should try
to remove settings from this file over time until you can remove the hole file.

This file is called `.rubocop_todo.yml` and can be created using:

```bash
rubocop --auto-gen-config
```

You also need to add this at the top of your `.rubocop.yml` file to load it:

```yaml
inherit_from: .rubocop_todo.yml
```

[default_config]: https://github.com/bbatsov/rubocop/tree/master/config
[ruby_style_guide]: https://github.com/bbatsov/ruby-style-guide
[standout]: http://standout.se
[rubocop]: https://github.com/bbatsov/rubocop
