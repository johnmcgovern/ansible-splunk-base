# uf-install

Installs the Splunk Universal Forwarder from the tarball and starts it.

Used by `uf-install.yml` and `uf-combo.yml`.

## Non-destructive by design

The tarball is extracted with `creates: {{ splunk_uf_home }}/bin/splunk`, so an
existing forwarder is never overwritten and re-running is a no-op.
`user-seed.conf` is written only on a fresh extract, so a re-run cannot reset
the admin password of a running forwarder.

## Variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `splunk_uf_user` | *(required)* | Forwarder admin username, seeded on first install |
| `splunk_uf_pass` | *(required)* | Forwarder admin password, seeded on first install |
| `uf_install_method` | `initd` | `initd` or `systemd` (see below) |

Plus the shared variables from `splunk-common`.

## Process management

`uf_install_method` is deliberately separate from `install_method`, and
defaults to `initd` because that is what this role has always done. Setting it
to `systemd` manages the forwarder through `SplunkForwarder.service`, with the
same Ubuntu unit-file fixes the Enterprise path uses.

> **Untested:** unlike the Enterprise path, UF systemd management has not yet
> been exercised against a real forwarder. Test it before relying on it.
