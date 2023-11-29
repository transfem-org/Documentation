---
title: "Fresh Installation"
weight: 1100
toc: true
---

Before you start, you really need:

* an always-on machine that can receive connections from the Internet
* a domain name to dedicate to your Sharkey instance, resolving to the
  address(es) of that machine
* a web server running on that machine, with valid TLS certificates
  for the domain name

## With Docker

Prerequisites:

* Docker
* Compose Plugin

Create multiple directories:

    mkdir Sharkey && mkdir Sharkey/.config

Fetch all required examples and enter directory:

    wget -O Sharkey/docker-compose.yml https://raw.githubusercontent.com/transfem-org/Sharkey/stable/docker-compose.yml.example
    wget -O Sharkey/.config/default.yml https://raw.githubusercontent.com/transfem-org/Sharkey/stable/.config/example.yml
    wget -O Sharkey/.config/docker.env https://raw.githubusercontent.com/transfem-org/Sharkey/stable/.config/docker_example.env
    cd Sharkey

Edit `.config/default.yml`, there are comments explaining what each
option means. In particular, we're going to assume you have:

    url: https://{YOUR DOMAIN NAME}/

(replace `{YOUR DOMAIN NAME}` with the domain name we talked about
at the start).

Edit `docker-compose.yml`, there are multiple comments there as
well. If you want to set up note search with meilisearch, uncomment
all of meilisearch options, otherwise proceed to do the following
changes in the `services:` / `web:` section:

* uncomment the line that starts with `image:`
* remove the line `build: .`

Starting:

    docker compose up -d

## Manually

[Same as
Misskey](https://misskey-hub.net/en/docs/install/manual.html).

Prerequisites:

* NodeJS version 20.4 or later, with *both* `npm` and `pnpm`
  installed (`corepack enable` should suffice, make sure you run it
  as `root` if you're using your system NodeJS)
* PostgreSQL version 15 or later
* Redis
* FFmpeg
* all the various packages to compile and build C code, and Python (on
  Debian-style systems, that's `build-essential` & `python`)

Create a `sharkey` user:

    adduser --disabled-password --disabled-login sharkey

start a shell as that user:

    sudo -u sharkey -i

(or something like that), then:

    git clone --recurse-submodules -b stable https://github.com/transfem-org/Sharkey.git
    cd Sharkey
    pnpm install --frozen-lockfile
    cp .config/example.yml .config/default.yml

Edit `.config/default.yml`, there are comments explaining what each
option means. In particular, we're going to assume you have:

    url: https://{YOUR DOMAIN NAME}/
    db:
      host: localhost
      port: 5432
      db: sharkey
      user: sharkey
      pass: {YOUR PASSWORD}

(replace `{YOUR PASSWORD}` with an actual password you make up for
this, and `{YOUR DOMAIN NAME}` with the domain name we talked about
at the start)

Building:

    pnpm run build

Create the PostgreSQL user and database, either with `createuser`
and `createdb` or with `sudo -u postgres psql` and then:

    create database sharkey with encoding = 'UTF8';
    create user sharkey with encrypted password '{YOUR_PASSWORD}';
    grant all privileges on database sharkey to sharkey;
    alter database sharkey owner to sharkey;
    \q

(replace `{YOUR PASSWORD}` with the same password as before)

Then create the schema:

    pnpm run init

And start it:

    pnpm start

you should see a series of colourful lines, ending with something
like:

    Now listening on port 3000 on https://example.tld

but with a different URL at the end (you *did* change the `url`
setting in the config file, right?). Stop that process (control-C is
enough), and set up a system service for Sharkey.

### With systemd

Create a file `/etc/systemd/system/sharkey.service` containing:

    [Unit]
    Description=Sharkey daemon

    [Service]
    Type=simple
    User=sharkey
    ExecStart=/usr/bin/pnpm start
    WorkingDirectory=/home/sharkey/Sharkey
    Environment="NODE_OPTIONS=--max-old-space-size=8192"
    Environment="NODE_ENV=production"
    TimeoutSec=60
    StandardOutput=journal
    StandardError=journal
    SyslogIdentifier=sharkey
    Restart=always

    [Install]
    WantedBy=multi-user.target

(you may need to change that `/usr/bin/pnpm` if you're not using
your system NodeJS).

Then:

    sudo systemctl daemon-reload
    sudo systemctl enable sharkey
    sudo systemctl start sharkey

After that, `systemctl status sharkey` should show that it's
running.

### With OpenRC

Create a file `/etc/init.d/sharkey` containing:

    #!/sbin/openrc-run

    name=sharkey
    description="Sharkey daemon"

    command="/usr/bin/pnpm"
    command_args="start"
    command_user="sharkey"

    supervisor="supervise-daemon"
    supervise_daemon_args=" -d /home/sharkey/Sharkey -e NODE_ENV=production -e \"NODE_OPTIONS=--max-old-space-size=8192\"

    pidfile="/run/${RC_SVCNAME}.pid"

    depend() {
      need net
      use logger nginx
    }

(you may need to change that `/usr/bin/pnpm` if you're not using
your system NodeJS).

Then:

    sudo rc-update add sharkey
    sudo rc-service sharkey start

After that, `rc-service sharkey status` should show that it's
running.

## Configure the web server

### NGINX

See [Misskey's
instructions](https://misskey-hub.net/en/docs/admin/nginx.html)

# Update Sharkey

Very similar to the installation process:

    sudo -u sharkey -i
    cd Sharkey
    git checkout stable
    git pull --recurse-submodules
    pnpm install --frozen-lockfile
    pnpm run build
    pnpm run migrate

Then restart the service (`sudo systemctl restart sharkey` or
`rc-service sharkey restart`).

If there's problems with updating, you can run `pnpm run clean`
and/or `pnpm run clean-all` which will remove all the effects of a
previous build, then you can install+build+migrate+restart again.