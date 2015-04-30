# Resque

To create a job:

    Resque.enqueue(JobClass, arg)

## With Active Job

> From [https://quickleft.com/blog/getting-started-with-active-job/](https://quickleft.com/blog/getting-started-with-active-job/)

In `config/initializers/resque.rb`:

    Resque.redis = Redis.new(url: Rails.application.secrets.redis_url)
    # Should this be (https://gist.github.com/defunkt/238999#comment-581899)
    # Resque.after_fork = Proc.new { ActiveRecord::Base.verify_active_connections!  }
    # Resque.after_fork = Proc.new { ActiveRecord::Base.clear_active_connections! }
    Resque.after_fork = Proc.new { ActiveRecord::Base.establish_connection }
    
    # https://github.com/resque/resque/wiki/FAQ#how-do-you-make-a-resque-job-wait-for-an-activerecord-transaction-commit
    require 'ar_after_transaction'
    require 'resque'
    Resque.class_eval do
      class << self
        alias_method :enqueue_without_transaction, :enqueue
        def enqueue(*args)
          ActiveRecord::Base.after_transaction do
            enqueue_without_transaction(*args)
          end
        end
      end
    end

Now that you have your initializer you need some secrets; in `config/secrets.yml`:

    defaults: &defaults
      redis_url: <%= ENV["REDIS_URL"] || "redis://localhost:6379" %>

In `lib/tasks/resque.rake` you need to require the rake tasks:

    require 'resque/tasks'
    require 'resque/scheduler/tasks'

    namespace :resque do
      task :setup => :environment do
        require 'resque'
        require 'resque-scheduler'
      end
    end

In `config/application.rb` you need to require the correct railtie (unless you have already required `rails/all`:

   require "active_job/railtie"


In `config/initializers/active_job.rb`

   ActiveJob::Base.queue_adapter = :resque

## On Heroku

Heroku [waits 10 seconds after sending SIGTERM to send SIGKILL](https://devcenter.heroku.com/articles/error-codes#r12-exit-timeout), so be sure to use a RESQUE_TERM_TIMEOUT value less than 10.

> From [https://devcenter.heroku.com/articles/queuing-ruby-resque](https://devcenter.heroku.com/articles/queuing-ruby-resque)

In your `Procfile`:

    resque: env TERM_CHILD=1 RESQUE_TERM_TIMEOUT=7 QUEUE=* bundle exec rake resque:work

You'll need to add Redis to go for the above config values to work.

You'll want to make sure you have a resque worker:

    heroku scale web=1 resque=1
