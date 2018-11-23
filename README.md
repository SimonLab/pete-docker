- Create new repository on Github and clone it

- create Dockerfile to create the app container with elixir node and elm

```
FROM elixir:1.7.3

  RUN mix local.hex --force \
    && mix archive.install hex phx_new 1.4.0 --force \
    && apt-get update \
    && curl -sL https://deb.nodesource.com/setup_8.x | bash \
    && apt-get install -y apt-utils \
    && apt-get install -y nodejs \
    && apt-get install -y build-essential \
    && apt-get install -y inotify-tools \
    && mix local.rebar --force \
    && wget "https://github.com/elm/compiler/releases/download/0.19.0/binaries-for-linux.tar.gz" \
    && tar xzf binaries-for-linux.tar.gz \
    && mv elm /usr/local/bin/

  ENV APP_HOME /app
  WORKDIR $APP_HOME


CMD ["mix", "phx.server"]
```

- create docker-compose.yml file to attach app and Postgres containers

```
version: '3'
services:
  app:
    build: .
    ports:
      - "4000:4000"
    volumes:
      - .:/app
    depends_on:
      - db
    env_file:
      - ./.env
  db:
    image: postgres:10.5
    ports:
        - "5432:5432"
```

- create .env file as docker-compose is looking for this file for environment variables

- create the new Phoenix application with
`docker-compose run --rm app mix phx.new . --app name_of_the_app`

- Update permissions of files to allow saving (files are created via the Docker so they don't have the correct permissions): `sudo chown -R $USER:$USER .`

- add `.env` to gitignore

- update config for dev and test to change `localhost` as host for Postgres to db (the name of the postgres service define in docker-compose.yml)

- create the database: `docker-compose run --rm app mix ecto.create`

- run the application: `docker-compose up`

- visit http://localhost:4000/
