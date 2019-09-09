# Deploying Elixir Applications

[.header: #FDDF26, text-scale(3.0)]
[.text: #FFFFFF]

![original](title-slide.png)

Eric Oestrich
SmartLogic

---

![original](plain-slide.png)

# Agenda

- Distillery
- Docker
- Server Setup
- Deploying

---

![original](plain-slide.png)

# Disclaimer

- This is one way of deploying
- You may want more complex or less complex
  - More: Kubernetes
  - Less: Heroku

---

![original](plain-slide.png)

# Grapevine

[https://github.com/oestrich/grapevine](https://github.com/oestrich/grapevine)
[http://grapevine.haus](http://grapevine.haus)

---

![original](section-div.png)

# Distillery

---

![original](plain-slide.png)

# Distillery

Generates Elixir OTP Releases

[https://github.com/bitwalker/distillery](https://github.com/bitwalker/distillery)

---

![original](plain-slide.png)

## rel/config.exs

[.code-highlight: all]
[.code-highlight: 2-3]
[.code-highlight: 4]
[.code-highlight: 5]
[.code-highlight: 7-12]

```elixir
environment :prod do
  set include_erts: true
  set include_src: false
  set cookie: cookie
  set vm_args: "rel/vm.args.eex"

  set config_providers: [
    {
      Mix.Releases.Config.Providers.Elixir,
      ["/etc/grapevine.config.exs"]
    }
  ]
end
```

---

![original](plain-slide.png)

## rel/config.exs

[.code-highlight: all]
[.code-highlight: 2]
[.code-highlight: 3-5]
[.code-highlight: 7-10]

```elixir
release :grapevine do
  set version: current_version(:grapevine)
  set applications: [
    grapevine_telnet: :none
  ]

  set commands: [
    migrate: "rel/commands/migrate.sh",
    restarting: "rel/commands/restarting.sh",
  ]
end
```

---

![original](plain-slide.png)

## /etc/grapevine.config.exs

```elixir
use Mix.Config

# Config that only contains values
```

---

![original](plain-slide.png)

## rel/commands/restarting.sh

```bash
#!/bin/sh
bin/grapevine rpc "Grapevine.restart()"
```

---

![original](section-div.png)

# Generating a Release

---

![original](plain-slide.png)

## release.sh

```bash
#!/bin/bash
set -e

UUID=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)

rm -r deploy/tmp
mkdir -p deploy/tmp/build
git archive master | tar x -C deploy/tmp/build/
cd deploy/tmp/build

docker build -f Dockerfile.releaser -t knctrr:releaser .

docker run --name knctrr_releaser_${DOCKER_UUID} knctrr:releaser /bin/true
docker cp knctrr_releaser_${DOCKER_UUID}:/app/_build/prod/rel/knctrr/releases/0.1.0/knctrr.tar.gz ../
docker rm knctrr_releaser_${DOCKER_UUID}
```

---

![original](plain-slide.png)

## release.sh

[.code-highlight: all]
[.code-highlight: 1]
[.code-highlight: 2]
[.code-highlight: 3]
[.code-highlight: 4]

```bash
rm -r deploy/tmp
mkdir -p deploy/tmp/build
git archive master | tar x -C deploy/tmp/build/
cd deploy/tmp/build
```

---

![original](plain-slide.png)

## release.sh

[.code-highlight: all]
[.code-highlight: 1]
[.code-highlight: 3]
[.code-highlight: 4-5]
[.code-highlight: 6]

```bash
docker build -f Dockerfile.releaser -t app:releaser .

docker run --name releaser_${UUID} app:releaser /bin/true
path=/app/_build/prod/rel/app/releases/0.1.0/app.tar.gz
docker cp releaser_${UUID}:${path} ../
docker rm releaser_${UUID}
```

---

![original](section-div.png)

# Docker

---

![original](plain-slide.png)

# Docker

- Uses a multi-part docker image

---

![original](plain-slide.png)

## Dockerfile.releaser

[.code-highlight: all]
[.code-highlight: 1]
[.code-highlight: 3-4]
[.code-highlight: 8-9]
[.code-highlight: 11]

```docker
FROM elixir:1.8 as builder

RUN mix local.rebar --force && \
    mix local.hex --force

WORKDIR /app
ENV MIX_ENV=prod
COPY mix.* /app/
RUN mix deps.get --only prod

RUN mix deps.compile
```

---

![original](plain-slide.png)

## Dockerfile.releaser

[.code-highlight: all]
[.code-highlight: 1]
[.code-highlight: 4-6]
[.code-highlight: 8]
[.code-highlight: 10-11]

```docker
FROM node:11.2 as frontend

WORKDIR /app
COPY assets/package.json assets/yarn.lock /app/
COPY --from=builder /app/deps/phoenix /deps/phoenix
COPY --from=builder /app/deps/phoenix_html /deps/phoenix_html

RUN npm install -g yarn && yarn install

COPY assets /app
RUN npm run deploy
```

---

![original](plain-slide.png)

## Dockerfile.releaser

[.code-highlight: all]
[.code-highlight: 1]
[.code-highlight: 2-3]
[.code-highlight: 4]
[.code-highlight: 5]

```docker
FROM builder as releaser
COPY --from=frontend /priv/static /app/priv/static
COPY . /app/
RUN mix phx.digest
RUN mix release --env=prod
```

---

![original](section-div.png)

# Ansible

---

![original](plain-slide.png)

# Ansible

- Tool for automating server configuration
- Mostly outside of the scope of the talk

---

![original](plain-slide.png)

## Ansible

- Installs very little
  - nginx, traefik
- Note that the server will *not* have erlang installed

---

![original](section-div.png)

# Deploying

---

![original](plain-slide.png)

## Simple script

```bash
set -e
host=grapevine.haus

scp tmp/grapevine.tar.gz deploy@${host}:

ssh deploy@${host} './grapevine/bin/grapevine restarting'
ssh deploy@${host} 'sudo systemctl stop grapevine'
ssh deploy@${host} 'tar xzf grapevine.tar.gz  -C grapevine'
ssh deploy@${host} './grapevine/bin/grapevine migrate'
ssh deploy@${host} 'sudo systemctl start grapevine'
```

---

![original](section-div.png)

# Live Deploy
---

![original](section-div.png)

# Questions?

---

![original](closer-slide.png)

## <br><br><br><br><br><br><br><br><br><br><br><br><br><br>We build custom web <br>and mobile applications.
#### We've built 150+ applications since 2005. <br>Need a custom web/mobile app? We can help.<br><br><br>888-544-SMRT  /  contact@smartlogic.io

[.header:#ffffff]
