---
title: "Migrating from another software"
weight: 1200
toc: true
---

## From Misskey

Let's say you have a working Misskey, running as user `misskey` from
`/home/misskey/misskey`.

Migrating to Sharkey is the same as updating to a newer Misskey
version:

    sudo -u misskey -i
    cd misskey
    git remote rename origin misskey
    git remote add origin https://github.com/transfem-org/Sharkey.git
    git remote update -p
    git checkout -b stable --track origin/stable
    git pull --recurse-submodules
    pnpm install --frozen-lockfile
    pnpm run build
    pnpm run migrate

Then you can restart your service.

If you see weirdness like service not starting, or missing labels in
the web UI, you should first make sure the build worked (check all the
error messages!), then try building again from scratch:

    pnpm run clean-all
    pnpm install --frozen-lockfile
    pnpm run build
    pnpm run migrate

Also, clear your browser's cache and local storage (this will log your
browser out of Misskey/Sharkey).

## From Firefish

This guide was only tested on a Firefish 1.0.5-RC instance: other
versions and instances may differ quite a lot from the one we used,
esspecially if you have custom patches applied.

If any of the steps fails, especially SQL queries, seek help from us
[via Matrix](https://matrix.to/#/#sharkey-support:shourai.de) or
[Discord](https://discord.gg/8hF6pMVWja). Do *not* try fixing things
in the DB yourself unless you *really* know what you are doing.

Before you begin, please take a backup of your database. While you're
there, make sure you know how to restore from that backup! Using the
`plain` format for `pg_dump` is probably the simplest way.

It's a good idea to have a separate backup of your list of silenced
instances and of your `user` table.

### Using Docker

Stop / shut down the entire stack: PostgreSQL, Redis / KeyDB /
DragonflyDB, Sonic / ElasticSearch / MeiliSearch, Firefish
itself. Stop all of it.

Edit you docker compose and replace the Firefish image with
`ghcr.io/transfem-org/sharkey:stable`

If you use Sonic or ElasticSearch replace that section of the Docker
Compose with the following, as Sharkey currently only supports
meilisearch:

```yml
meilisearch:
    restart: always
    image: getmeili/meilisearch:v1.3.4
    environment:
      - MEILI_NO_ANALYTICS=true
      - MEILI_ENV=production
    env_file:
      - .config/meilisearch.env
    networks:
      - calcnet # <-- Use whatever network name is used in the docker compose here
    volumes:
      - ./meili_data:/meili_data # <--- make sure to replace the volume with one that fits your existing docker compose
```

If you use DragonflyDB replace it with Redis or KeyDB, as Sharkey
currently does not support DragonflyDB. To do this replace the section
in the docker compose with the following:

```yml
redis:
    restart: always
    image: redis:7-alpine
    networks:
      - calcnet # <-- Use whatever network name is used in the docker compose here
    volumes:
      - ./redis:/data # <-- Make sure to replace the volume with the one used in your firefish docker compose
    healthcheck:
      test: "redis-cli ping"
      interval: 5s
      retries: 20
```

Backup your your firefish config and replace it with the [default
sharkey
one](https://raw.githubusercontent.com/transfem-org/Sharkey/develop/.config/example.yml)

Edit the config inline with your instance settings. Make sure to use
the same `db` & `redis` settings as in your Firefish config.

Now is the time to *backup* your database and Redis volumes!

Firefish's docker-compose uses PostgreSQL version 12. We strongly
recommend upgrading to 15 or 16. You can do this by making a new db
volume, starting the newer PostgreSQL on it, and importing the data
from the backup you just made. Refer to [the PostgreSQL
documentation](https://www.postgresql.org/docs/current/backup-dump.html).


Make sure to update the *mount paths* for volumes of the Sharkey
container from `/firefish/` to `/sharkey/` (so your existing volumes
will show up at the new path, inside the container).

Now start *only* the database with `docker compose up -d db` (instead
of `db`, you may need to use whatever name is set for the service in
your docker compose config), and start a `psql` shell with `docker
exec -it db psql -U firefish -d firefish` (replace `db` as before, and
`-U firefish -d firefish` with the database user and database name,
respectively, if they're different from `firefish`).

You should now be connected to your Firefish database. We need to
massage it into shape so that Sharkey database migrations will
work. The following series of SQL queries / commands should do it, but
please read the comments and pay attention to the results after each
query!

```sql
-- start a transaction, so we won't leave the db in a halfway state if
-- things go wrong
BEGIN;

-- we need to add back some columns that Firefish removed, but that
-- Sharkey migrations expect
ALTER TABLE "user_profile" ADD "integrations" JSONB NOT NULL DEFAULT '{}';
ALTER TABLE "meta" ADD "twitterConsumerSecret" VARCHAR(128);
ALTER TABLE "meta" ADD "twitterConsumerKey" VARCHAR(128);
ALTER TABLE "meta" ADD "enableTwitterIntegration" BOOLEAN NOT NULL DEFAULT false;
ALTER TABLE "meta" ADD "enableGithubIntegration" BOOLEAN NOT NULL DEFAULT false;
ALTER TABLE "meta" ADD "githubClientId" VARCHAR(128);
ALTER TABLE "meta" ADD "githubClientSecret" VARCHAR(128);
ALTER TABLE "meta" ADD "enableDiscordIntegration" BOOLEAN NOT NULL DEFAULT false;
ALTER TABLE "meta" ADD "discordClientId" VARCHAR(128);
ALTER TABLE "meta" ADD "discordClientSecret" VARCHAR(128);

-- also an extra table, for the same reasons
CREATE TABLE antenna_note();

-- move aside some FireFish columns; Sharkey migrations will
-- re-create them; we don't `DROP` them because we want to keep the data
ALTER TABLE "user" RENAME COLUMN "movedToUri" TO "ff_movedToUri";
ALTER TABLE "user" RENAME COLUMN "alsoKnownAs" TO "ff_alsoKnownAs";
ALTER TABLE "user" RENAME COLUMN "isIndexable" TO "ff_isIndexable";
ALTER TABLE "user" RENAME COLUMN "speakAsCat" TO "ff_speakAsCat";
ALTER TABLE "user_profile" RENAME COLUMN "preventAiLearning" TO "ff_preventAiLearning";
ALTER TABLE "meta" RENAME COLUMN "silencedHosts" TO "ff_silencedHosts";

-- this column was added by both Firefish and Misskey, but with
-- different names, let's fix it
ALTER TABLE "meta" RENAME COLUMN "ToSUrl" TO "termsOfServiceUrl";

-- update antenna types, this is only needed on some instances but
-- recommend to run anyway
--
-- this *removes* any antennas of types not supported by Sharkey!
CREATE TYPE public.new_antenna_src_enum AS ENUM ('home', 'all', 'list');
ALTER TABLE antenna ADD COLUMN new_src public.new_antenna_src_enum;
DELETE FROM antenna WHERE src NOT IN ('home', 'all', 'list');
ALTER TABLE antenna DROP COLUMN src;
ALTER TABLE antenna RENAME COLUMN new_src TO src;
DROP TYPE public.antenna_src_enum;
ALTER TYPE new_antenna_src_enum RENAME TO antenna_src_enum;

-- optional but recommended: delete all empty moderation log entries
DELETE FROM moderation_log WHERE info = '{}';

-- only needed on some instances, run this if
-- `\dT+ user_profile_mutingnotificationtypes_enum`
-- does not show `note` in the "elements" section
ALTER TYPE "public"."user_profile_mutingnotificationtypes_enum" ADD VALUE 'note';
```

If everything worked and you saw no errors, you can run `COMMIT;` in
that same `psql` shell, to commit all the changes, then close that
shell. Again, if anything went wrong, come talk to us on
[Matrix](https://matrix.to/#/#sharkey-support:shourai.de) or
[Discord](https://discord.gg/8hF6pMVWja)!

Start Sharkey, and let it run all its migrations. Once that's done,
and Starkey says it's listening, stop Sharkey but keep the database
running.

Open another `psql` shell like before (`docker exec -it db psql -U
firefish -d firefish`, replacing things as before). We need another
small pass of massaging.

```sql
BEGIN;

-- all existing users are approved, because Firefish doesn't have a
-- concept of approvals
UPDATE "user" SET approved = true;

-- now we put back the data we moved aside
UPDATE "user" SET "movedToUri" = "ff_movedToUri" WHERE "ff_movedToUri" IS NOT NULL;
UPDATE "user" SET "alsoKnownAs" = "ff_alsoKnownAs" WHERE "ff_alsoKnownAs" IS NOT NULL;
UPDATE "user" SET "noindex" = NOT (COALESCE("ff_isIndexable", true));
UPDATE "user" SET "speakAsCat" = COALESCE("ff_speakAsCat", false);
UPDATE "user_profile" SET "preventAiLearning" = COALESCE("ff_preventAiLearning", true);
UPDATE "meta" SET "silencedHosts" = COALESCE("ff_silencedHosts",'{}');

ALTER TABLE "user" DROP COLUMN "ff_movedToUri";
ALTER TABLE "user" DROP COLUMN "ff_alsoKnownAs";
ALTER TABLE "user" DROP COLUMN "ff_isIndexable";
ALTER TABLE "user" DROP COLUMN "ff_speakAsCat";
ALTER TABLE "user_profile" DROP COLUMN "ff_preventAiLearning";
ALTER TABLE "meta" DROP COLUMN "ff_silencedHosts";

```

If everything worked and you saw no errors, you can run `COMMIT;` in
that same `psql` shell, to commit all the changes, then close that
shell. Again, if anything went wrong, come talk to us on
[Matrix](https://matrix.to/#/#sharkey-support:shourai.de) or
[Discord](https://discord.gg/8hF6pMVWja)!

Start everything up again, you should see no errors in the logs.

Log in as an administrator, and go to the control panel.
If you use an object store such as S3, double-check your settings
(it's possible, for example, that the URL now looks like
`https://https://yourdomain.com`, fix it). If you want your users to
be able to search notes, you must enable via the "roles" system.

Congratulations, you're now running Sharkey!
