# Privacy Policy - Linkding for TerraMaster TOS 7

**Application**: Linkding (`app_id: linkding`)
**Publisher**: waskevin
**Policy version**: 1.0
**Effective date**: 2026-11-09
**Contact**: https://github.com/waskevin/linkding/issues

This policy covers the TOS 7 application package distributed by waskevin
(the `linkding.tar.gz` archive and the `config.ini` / `linkding.lang` / `linkding.svg` /
`docker-compose.yml` files inside it) and the container it starts on your
device.  It does not cover the Linkding project's own website or any service you
connect the application to.

## What the package itself does

- It runs one Docker container from the public image `sissbruecker/linkding:1.47.0`.
- It does not contain analytics, telemetry, advertising or tracking code.
- It does not send any data to waskevin or to any third party.
- It does not open network connections on its own; the only traffic is what
  the application and your browser make.

## Data the application stores

All application data (one SQLite database plus the favicon assets, in
`data/`) is written to
`/Volume<N>/DockerAppData/linkding/`.  Nothing is
uploaded anywhere by the packaging.

## Data the application may send

- When you save a bookmark, Linkding requests that web page once to read its
  title, description and favicon. Nothing else leaves your device.
- Page snapshots, preview images and automatic tagging are disabled in this
  package, so no background job contacts any third party.
- Flat list of outbound destinations: the sites you bookmark.

## Removing your data

Uninstalling the application from the TOS App Center stops and removes the
container.  Deleting the folder `DockerAppData/linkding/` on your device removes
all remaining data; nothing is left elsewhere.

## Changes

Any change to this policy is published in this repository together with the
version of the package it applies to.
