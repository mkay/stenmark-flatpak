# Stenmark Flatpak repository

A Flatpak remote for Stenmark, a GTK4 Markdown reader, organizer and editor.

- Project page: <https://singular.de/apps/stenmark/>
- Source and issues: <https://github.com/mkay/stenmark>

Stenmark is not on Flathub: their policy doesn't allow AI-assisted apps, and
Stenmark is. This repository is how you install the Flatpak and keep it updated
instead.

## Install

```sh
flatpak remote-add --user stenmark https://mkay.github.io/stenmark-flatpak/stenmark.flatpakrepo
flatpak install stenmark de.singular.stenmark
```

Updates then arrive with `flatpak update` like any other app.

The GNOME runtime Stenmark builds against is **not** hosted here — it comes from
Flathub, so you need that remote too:

```sh
flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

## What it is allowed to touch

Stenmark browses folders rather than single files — the sidebar walks a
directory tree — so a document-portal grant, which covers only the one file a
user picked, is not enough. The sandbox therefore gets `--filesystem=home`,
plus `/media`, `/run/media` and `/mnt` so notes on a USB stick or a mounted
share are reachable. Not `--filesystem=host`: `/etc`, `/usr` and other users'
home directories stay invisible.

It gets **no network access**. Files are read and rendered locally, and a link
you click is handed to the portal, which opens it in your browser outside the
sandbox.

Settings live in `~/.var/app/de.singular.stenmark/config/stenmark/`, separate
from a native install's `~/.config/stenmark/`. The two do not share state.

## What is in here

`repo/` is a static [OSTree](https://ostreedev.github.io/ostree/) repository in
`archive-z2` mode: every file is stored once under the hash of its contents, so
successive releases share everything that did not change, and a client fetches
only what it is missing. Nothing but a plain HTTP server is required to serve it.

Commits are GPG-signed; the public key is embedded in `stenmark.flatpakrepo`, so
`flatpak` verifies every update against it.

Debug symbols are not published — they roughly double the repository for
something no user of a binary remote consumes. Build from source if you need
them.

## Publishing

`build-aux/flatpak/publish-repo.sh` in the main Stenmark repository writes into
this checkout, but only these two paths — everything else here is hand-written
and survives a publish untouched:

| Path | |
| --- | --- |
| `repo/` | deleted and recopied on every publish |
| `stenmark.flatpakrepo` | regenerated on every publish, so the embedded key cannot drift from the key the repo was signed with |

`index.html`, this README and the images are safe to edit by hand.
