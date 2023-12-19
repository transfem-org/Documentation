---
title: "FAQs"
weight: 4000
toc: false
---

# How do I enable note search?

You use the "roles" system. Log in as administrator, go to the
"control panel", select the "roles" section (under "management"). Then
either expand the "role template" (if you want to give every user
access to search) or create a new role (if you want to give access to
only some users), then change the "**Usage of note search**" setting.

# How do I give my users more Drive space?

You use the "roles" system. Log in as administrator, go to the
"control panel", select the "roles" section (under "management"). Then
either expand the "role template" (if you want to give every user the
same amount space) or create a new role (if you want to give different
amounts of space to different users), then change the "**Drive
capacity**" setting.

# How do I enable push notifications for the web interface?

First of all, you need to generate a pair of so-called "VAPID" keys.

One way to do that is, from your Sharkey directory (git clone, or
inside the Docker image):

    ./packages/backend/node_modules/.bin/web-push generate-vapid-keys

Alternatively, you can use [an online
generator](https://www.stephane-quantin.com/en/tools/generators/vapid-keys).

Once you have that public and private keys, log in as administrator,
go to the "control panel", select the "general" section (under
"settings"), scroll to the "ServiceWorker" bit, enter both keys, and
enable the "Enable Push-Notifications for your Browser" toggle.
