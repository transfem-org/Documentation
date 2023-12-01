---
title: "Sharkey vs Misskey"
weight: 2100
toc: true
---

## A (probably comprehensive) list of differences

### Big ones

* fully federated note editing, you can also see previous versions of
  edited notes
* Mastodon-compatible API, including OAuth2
* can import your exported posts from Mastodon and most of its forks,
  Pleroma / Akkoma, Misskey / Firefish and forks, Twitter, Instagram,
  Facebook, including attachments (threading may not work perfectly,
  and other people's replies to your posts may not get imported)
* admins can require approval for new users' signups
* admins can silence users
* admins can mark all of a user's media as NSFW
* GDPR-style Data Subject Access Requests (users can export all data
  related to themselves)

### Fun ones

* can play module / tracker music files
* (federated) listenbrainz integration
* (federated) background image on user profiles
* "speak as cat" separate from "is a cat" (both setting are federated
  with compatible software)

### UI/UX
  
* option to open a note's detailed view by clicking on the note (most
  useful on mobile)
* images lacking alt text are marked as such
* UI elements can be round (as in Misskey) or square-ish
* "sign out" button in user menu
* user profile page has "notes" / "all" / "including files" tabs
* attachments can be collapsed by default
* buttons to show/hide all notes with CWs in a conversation
* one-button "like" (plus custom reactions on a separate button)
* animated MFM can be enabled/disabled on each note
* supports longer alt text
* pop-up user profiles show if follow requests to the user require
  approval, have a "open remote profile" option, and show custom
  fields (e.g. the user's website address)
* MFM cheatsheet when composing notes
* emoji auto-complete is case-insensitive
* it's always clear if a note has a poll (misskey sometimes hides
  that)
* multiple-choice polls are clearly marked as such
* boosts and quote-boost are accounted separately
* only 1 boost per note per user is allowed
* admins can remove bots from "trending"
* users can hide bots from their timelines
* translatable notes are shown translated regardless of where they're
  shown (e.g. when quoted, or when looking at their replies)
* when searching, users can restrict results to notes with attachments
* CSS class names are human-readable, to simplify browser-side
  customisation
* users can disable indexing of their notes (the setting is federated)
* "likes" and "reactions" federate correctly to Mastodon / Pleroma /
  Akkoma (Misskey sends them all as reactions)
* different error icons
* users can disable the "disconnected" warning (connection is usually
  re-established automatically, so the warning is rarely useful)
* users can set a default emoji for their likes/reactions
* when the instance is using meilisearch to index notes, Sharkey will
  use it in more cases (e.g. when limiting results to notes containing
  images)
* when showing a reply containing many mentions, they are shortened
* there's a search widget

### Ones of interest to admins

* quote-boosts federate correctly from/to Mastodon forks
* the not-very-functional "automatically mark attachments as NSFW" has
  been removed (smaller installation, faster image/video uploads)
* argon2 instead of bcrypt for hashing users' secrets
* admins can delete remote emojis
* admins can disable achievements
* admins can refresh remote user details
* admins can set a default emoji for likes/reactions
* the PWA icon matches the instance icon
* deleted custom emoji are automatically removed from Drive
