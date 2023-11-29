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

(someone please write this)