# dockerize-rails-projects
How to dockerize basic rails project

## Generate project without Ruby/Rails installed local

$ `docker run --rm --user "$(id -u):$(id -g)" -v $(pwd):/usr/src -w /usr/src -ti ruby:2.7.1 bash`

- docker run: basic comand to run/up any docker container.
- --rm: auto remove the container after use.
- --user "$(id -u):$(id -g)": map current local user id with container user (to allow edit files generate on container).
- -v $(pwd):/usr/src: map current path with '/usr/src' in container.
- -w /usr/src: set '/usr/src' as workdir (main path).
- -ti ruby:2.7.1: specify a docker image (from dockerhub for example)
- bash: command to execute in container to open a terminal instead ruby irb (as default in ruby image)

Done. You can now install any version of Rails you want. So...

$ `gem install rails`
(latest version)

Generate a rails project in ruby container:

$ `rails new my-rails-project --database=postgresql --skip-bundle`
(You can use any rails new options)

Done. You can now run `exit` in container and it's will finished and removed/down because the command --rm

## Create Dockerfile

Now, create a file with name Dockerfile in root of project.

And use this configuration:
(pay attention to the project name, node version, etc ... this is an example, you can customize it according to your needs)

```
FROM ruby:2.7.1

# add nodejs and yarn dependencies for the frontend
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - && \
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

# Install dependencies
RUN apt-get update && apt-get install -qq -y --no-install-recommends \
nodejs yarn build-essential libpq-dev imagemagick git-all nano

# Installl bundler
RUN gem install bundler

# Set enviroment INSTALL_PATH
ENV INSTALL_PATH /my-rails-project

# Create directory with INSTALL_PATH name
RUN mkdir -p $INSTALL_PATH

# Set WORKDIR (main directory) with a INSTALL_PATH
WORKDIR $INSTALL_PATH

# Copy Gemfile to current container folder.
COPY Gemfile ./

# Set enviroment BUNDLE_PATH
ENV BUNDLE_PATH /gems

# COPY all files in current folder to container.
COPY . .
```

The Dockerfile will determine a new "container" based on a container (ruby: 2.7.1 in this case) and customize it to your specifications.

## Docker-Compose

The docker-compose.yml file determine all the services you will use in your project (without install any in your local machine). In this example i use my Dockerfile to generate an "app" and a postgres as "db" direct from dockerhub.

Create a file docker-compose.yml in root of project and use this example (customize with any service):

```
version: "3.8"

services:
  db:
    image: "postgres:12.2"
    environment:
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres:/var/lib/postgresql/data

  app:
    build: .
    command: bash start.sh
    ports:
      - "3000:3000"
    environment:
      - DB_PASSWORD=postgres
    volumes:
      - .:/my-rails-project
      - gems:/gems
    depends_on:
      - db

volumes:
  postgres:
  gems:
```

## Start.sh

Create a file start.sh in root of project. This file will determine the commands when the container "app" is up:

```
# Install gems
bundle check || bundle install

# Run puma server
bundle exec puma -C config/puma.rb
```

## Build

Run `docker-compose build` to download, install, make everything to get up your rails app.

## Up

$ `docker-compose up` 
(will start your rails app).

## Observations

### config/database

Pay attention in your config/database.yml file if is set to connect with a docker db container. If you follow the steps in this document, your file should look like this:

```
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  user: postgres
  password: <%= ENV['DB_PASSWORD'] %>

development:
  <<: *default
  database: my-rails-project_development

test:
  <<: *default
  database: my-rails-project_test

production:
  <<: *default
  database: my-rails-project_production
  username: my-rails-project
  
### To run any command in your app container

$ `docker-compose run --rm app bundle exec rails db:create db:migrate`
(docker-compose run --rm app #AND YOUR COMMAND#)

other example:

$ `docker-compose run --rm app bundle exec rails webpacker:install`

### Permission problem ?

If you create any file from your container like `docker-compose run --rm app bundle exec rails g todos controller`, you will need a permission to edit the files in your local machine, run this:

$ `sudo chown -R $USER:$USER .`
