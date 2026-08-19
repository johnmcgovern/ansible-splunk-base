# os-config

Base OS configuration for a Splunk host: hostname, the Splunk user, ulimits,
transparent hugepages, the web port, and optionally OS patching.

Used by `os-config.yml` and `combo.yml`.

## Variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `os_update_packages` | `false` | Update every OS package (apt dist-upgrade / dnf update) |

Plus the shared variables from `splunk-common`.

## Notes

- **OS patching is opt-in.** Releases before this one always ran a full
  dist-upgrade, which could pull in a new kernel as a side effect of a config
  run. Set `os_update_packages: true` to restore that.
- **ulimits install as a drop-in** at `/etc/security/limits.d/99-splunk.conf`
  rather than by replacing `/etc/security/limits.conf`. A host configured by an
  earlier release has the old entries removed from `limits.conf` on the next
  run, so the values are defined in one place.
  Under systemd, splunkd takes its limits from the `Splunkd.service` unit
  (`LimitNOFILE` and friends); these PAM limits apply to initd startup and to
  interactive sessions as the Splunk user.
- Transparent hugepages are disabled by a boot-time oneshot unit, and the
  script is run immediately only when THP is not already `never`.
- tcp/8000 is opened through firewalld where firewalld is present. On Ubuntu
  and Amazon Linux, 443 is redirected to 8000 by a boot-time oneshot unit
  (`splunk-web-redirect.service`), so the UI stays reachable at `https://host/`
  across reboots.
