# deploying
What to do when you want to deploy your application to heroku -- file changes

1 #//--- gem files
replace Sqlite3 to Development test group

    group :development, :test do
      gem 'sqlite3'
      gem 'byebug'
    end

2 #//--- create production group and add two gems |pg|rails_12factor

    group :production do
      gem 'pg'
      gem 'rails_12factor'
    end

3 #//--- add Gem Puma to the top list to replace sqlite3

    gem 'puma'

4 #//--- Terminal run bundle
    bundle

5 #//--- create Procfile
    Procfile
    fyi This file must be named **Procfile** exactly.
    #inser this line of code
    web: bundle exec puma -t 5:5 -p ${PORT:-3000} -e ${RACK_ENV:-development}

6 #//--- create Configuration file for puma

     create new file Puma.rb
  --**paste this into the puma file
    
     workers Integer(ENV['WEB_CONCURRENCY'] || 2)
    threads_count = Integer(ENV['RAILS_MAX_THREADS'] || 5)
    threads threads_count, threads_count

    preload_app!

    rackup      DefaultRackup
    port        ENV['PORT']     || 3000
    environment ENV['RACK_ENV'] || 'development'

    on_worker_boot do
      # Worker specific setup for Rails 4.1+
      # See: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot
      ActiveRecord::Base.establish_connection
    end
    
 7 #//--- Set Rack enviroments
    
     echo "RACK_ENV=development" >>.env
     echo "PORT=3000" >.env
  
  You’ll also want to add .env to your .gitignore since this is for local environment setup.
  
      echo ".env" >.gitignore
      git add .gitignore
      git commit -m "add .env to .gitignore"
     
    Test if it works with heroku local  -- to exit type control C 
  
  8 #//--- config development and production files in enviroments
        
      cut this
      *
      config.action_mailer.default_url_options = { host: 'localhost' }   you can remove port 3000
      *
      and paste in production file
      *
      localhost will be changed with your url after deploy.. like https//:www.kungsamu.com or something..
      
   9 #//--- Heroku run rake db:migrate + figaro set enviroment
   
    Heroku run rake db:migrate --app *name of app*
      
   10 #//--- AWS bucket Heroku config
         
         -add gem 'aws-sdk', '~> 2.3'
         
  # config/environments/production.rb

        config.paperclip_defaults = {
        storage: :s3,
        s3_credentials: {
        bucket: ENV.fetch('S3_BUCKET_NAME'),
        access_key_id: ENV.fetch('AWS_ACCESS_KEY_ID'),
        secret_access_key: ENV.fetch('AWS_SECRET_ACCESS_KEY'),
        s3_region: ENV.fetch('AWS_REGION'),
        }
        }
          
     Additionally, we’ll need to the set the AWS configuration variables on the Heroku application.
        paste into command line via heroku
        heroku config:set S3_BUCKET_NAME=your_bucket_name --app *nameofapp
        heroku config:set AWS_ACCESS_KEY_ID=your_access_key_id --app *nameofapp
        heroku config:set AWS_SECRET_ACCESS_KEY=your_secret_access_key --app *nameofapp
        heroku config:set AWS_REGION=your_aws_region --app *nameofapp
To check type heroku config
