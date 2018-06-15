---
layout:     post
title:      "Start Delayed Job on boot and restart on deploy"
date:       2017-10-14
tags:
- delayed job
- ubuntu
- upstart
---

If you have an rails application that is running delayed job for its background processes you probably would like it to start automatically whenever the server restarts. But you will also need it to restart if your rails applications are deployed so you can be sure that it is running the latest version of it.

The process below should work great on Ubuntu 14.04 and newer versions.

## Create an Upstart script

Lets say that we have a rails application that are running as the user myapp and that it needs to initialise rbenv. Then this is the script you would like to add in `/etc/init/delayed_job.conf`

```bash
# /etc/init/delayed_job.conf
description "Delayed Job Background Worker"

# Set user that will run the service
setuid myapp
setgid myapp
env HOME=/home/myapp

# Start when system enters runlevel 2 (multi-user mode).
start on runlevel 2

# Restart the process if it dies with a signal
# or exit code not given by the 'normal exit' stanza.
respawn

# Give up if restart occurs 10 times in 90 seconds.
respawn limit 10 90

script
# this script runs in /bin/sh by default
# respawn as bash so we can source in rbenv
exec /bin/bash <<'EOT'
        # Set home path
        export HOME=/home/myapp
        # Add rbenv to path
        export PATH="$HOME/.rbenv/bin:$PATH"
        # Initialize rbenv
        eval "$(rbenv init -)"
        # Add rbenv plugins to path
        export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"
        # Set rails environment
        export RAILS_ENV=production
        # Go to the application directory
        cd /var/www/myapp/current
        # Run delayed job
        exec bin/delayed_job run
EOT
end script
```

Now, allow root to read and write but the rest should only be able to read the file.

```bash
$ sudo chmod 644 /etc/init/delayed_job.conf
```

Now delayed job will be started when your server starts.

You can start it now by running this command:

```bash
$ sudo initctl start delayed_job
```

## Allow user to start and stop delayed job without password

This step is kind of a preparation for the next step when we need to restart on deploy. The delayed job service that the user of the application should be able to control. We will now make the user able to start and stop the process using sudo but without password. That will enable us to create a script for capistrano later on which can execute the task.

Create your new sudo override file using this command

```bash
$ sudo visudo -f /etc/sudoers.d/delayed_job
```

Add this content

```bash
# /etc/sudoers.d/delayed_job
myapp ALL=NOPASSWD:/sbin/initctl start delayed_job, /sbin/initctl status delayed_job, /sbin/initctl stop delayed_job
```

## Restart Delayed Job on deploy

When you make a new deployment of your source code you don't want the delayed job process to still be running on the old version of the code. To fix that we should restart delayed job on deployment.

If you had delayed job in your application before you may have this line in your `Capfile`.

```ruby
require 'capistrano/delayed-job'
```

You must remove that one now if you have it. It basically does the same thing that we will add now but without using upstart.

We will now create a new file that will host our start, stop and restart tasks. Create a file at `lib/capistrano/tasks/delayed_job.rake` and add the content below.

```ruby
# lib/capistrano/tasks/delayed_job.rake
namespace :delayed_job do
  # Fetches the role for delayed job. Using :app as default
  # if delayed_job_server_role was not set.
  def delayed_job_roles
    fetch(:delayed_job_server_role, :app)
  end

  desc 'Stop the delayed_job process'
  task :stop do
    on roles(delayed_job_roles) do
      execute :sudo, :initctl, :stop, :delayed_job
    end
  end

  desc 'Start the delayed_job process'
  task :start do
    on roles(delayed_job_roles) do
      execute :sudo, :initctl, :start, :delayed_job
    end
  end

  desc 'Restart the delayed_job process'
  task :restart do
    on roles(delayed_job_roles) do
      execute :sudo, :initctl, :stop, :delayed_job
      execute :sudo, :initctl, :start, :delayed_job
    end
  end
end
```

To make the restart command runs when you deploy, add this to your `config/deploy.rb` file.

```ruby
after 'deploy:publishing', 'delayed_job:restart'
```

Now whenever someone deploys a new version using `cap production deploy` for example it will also restart the delayed job process. It will also respawn on failure and start it when the server reboots.
