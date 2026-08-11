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
| `uf_install_method` | `systemd` | `systemd` or `initd` (see below) |

Plus the shared variables from `splunk-common`.

## Process management

`uf_install_method` is separate from `install_method`, so a forwarder and an
indexer can be managed differently.

It defaults to `systemd`, which is what this role has effectively produced for
years: earlier releases ran `enable boot-start -user <user>` with no
`-systemd-managed` flag, and modern Splunk defaults that to systemd. The tasks
were labelled "initd" but created `SplunkForwarder.service`.

Setting it to `initd` passes `-systemd-managed 0`, which is required to get a
real `/etc/init.d/splunk` script.
