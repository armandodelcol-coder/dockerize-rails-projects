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

$ `rails new name-of-project --database=postgresql --skip-bundle`
(You can use any rails new options)

Done. You can now run `exit` in container and it's will finished and removed/down because the command --rm

## Create Dockerfile

Now, create a file with name Dockerfile in root of project.

And use this configuration:

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
ENV INSTALL_PATH /rails-exchange

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
