1. Create a Dockerfile

```
FROM ruby:2.3.3
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
ADD Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
ADD . /myapp
```

2. Create a Gemfile

```
source 'https://rubygems.org'
gem 'rails', '5.0.0.1'
```

3. Create a Gemfile.lock

`touch Gemfile.lock`

4. Create a docker-compose.yml file

```
version: '3'
services:
  db:
    image: postgres
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

5. Generate the project

`docker-compose run web rails new . --force --database=postgresql`

6. Build the containers

`docker-compose build`

7. Update the database config/database.yml

```
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password:
  pool: 5

development:
  <<: *default
  database: myapp_development


test:
  <<: *default
  database: myapp_test
```

8. Create the database

`docker-compose run web rake db:create`

9. Add sidekiq gem

`gem 'sidekiq'`

10. Update Gemfile.lock

`docker-compose run web bundle install`

11. Add redis container

```
redis:
  image: redis
```

12. Add active job queue config

```
config.active_job.queue_adapter = :sidekiq
```

13. Add sidekiq config

```
Sidekiq.configure_server do |config|
  config.redis = { url: 'redis://redis.example.com:7372/12' }
end

Sidekiq.configure_client do |config|
  config.redis = { url: 'redis://redis.example.com:7372/12' }
end
```

14. Generate a job

`docker-compose run web rails generate job Example`
