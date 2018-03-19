# Development Guide

## If you alredy have Rails project.

### Add raildock to your project by submodule

```bash
cd project/
git submodule add https://github.com/anneau/raildock.git
```

Your folder structure is here.

```
project
   |- app/
   |- bin/
  ...
   |- raildock/
   |- Gemfile
   |- package.json
  ...
```

### Copy docker env file

```bash
cp .env.example .env
```

If you want to change version, edit this file.

### Copy MariaDB table sql file

```
cp mariadb/docker-entrypoint-initdb.d/createdb.sql.example mariadb/docker-entrypoint-initdb.d/createdb.sql
```

If you want to change table name, update this.

By default, RailDock create dev_db_1 ~ dev_db_3 tables.

### Build and Run RailDock

```bash
docker-compose up -d
```

Build and Run Workspace container, Nginx container, MariaDB container

### Edit puma config

By default, RailDock Nginx container watch tmp/sockets/puma.sock file.
So, You should edit puma.rb file in rails config/ dir.

```ruby
# frozen_string_literal: true

threads_count = ENV.fetch('RAILS_MAX_THREADS') { 5 }
threads threads_count, threads_count

environment ENV.fetch('RAILS_ENV') { 'development' }

app_root = File.expand_path('../..', __FILE__)
bind "unix://#{app_root}/tmp/sockets/puma.sock"
```

### Run Rails Application

```bash
docker-compose exec --user=raildock workspace bash
> bundle install
> yarn install
> mkdir -p tmp/sockets && bundle exec puma -d
> bundle exec rails db:migrate
> bundle exec rails db:seed
```

### Access

Access to http://localhost
