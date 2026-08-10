# core-install

Installs Splunk Enterprise from the tarball, applies a base configuration, and
configures boot-start.

Used by `install.yml` and `combo.yml`.

## Non-destructive by design

The tarball is extracted with `creates: {{ splunk_home }}/bin/splunk`, so an
existing installation is never overwritten and re-running the playbook against
an installed host is a no-op. `enable boot-start` is skipped the same way once
the unit file exists.

Two files are written with `force: no` - `web.conf` and `server.conf` - so a
re-run cannot revert settings written by `tls-config` or by Splunk itself.
`user-seed.conf` is written only on a fresh extract, so a re-run can never
reset the admin password of a running instance.

## Variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `splunk_user` | *(required)* | Admin username, seeded on first install |
| `splunk_pass` | *(required)* | Admin password, seeded on first install |
| `check_for_updates` | `true` | Let Splunk check for new releases |
| `splunk_email_allowed_domains` | `[]` | Domains alert email may be sent to. Empty leaves the setting unmanaged, which is what Splunk warns about in the UI. A single domain may be given as a bare string. |

Plus the shared variables from `splunk-common`.

## Notes

- Configuration changes notify the `restart splunk` handler, so Splunk is
  restarted only when something actually changed.
- `alert_actions.conf` is edited a key at a time rather than templated, so
  email settings configured in Splunk Web are left alone.
