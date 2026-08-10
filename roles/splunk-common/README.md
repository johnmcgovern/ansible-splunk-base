# splunk-common

Shared foundation for the other roles. Not run directly from a playbook.

Every role lists this one as a dependency in `meta/main.yml`, which puts its
defaults and handlers in scope and validates the shared variables once, before
any task runs. Its `tasks/main.yml` is intentionally empty - the helper task
files below are included by name, so a role that only needs the shared
variables does not run a dozen skipped tasks.

## Shared variables

Defined in `defaults/main.yml` and documented in `meta/argument_specs.yml`:
`os_user`, `os_group`, `splunk_home`, `splunk_uf_home`, `splunk_base_path`,
`install_method`, `uf_install_method`, `download_tgz_from_splunk_servers`,
`download_uf_tgz_from_splunk_servers`, `splunk_download_url`,
`splunk_uf_download_url`, `splunk_tgz_checksum`, `splunk_uf_tgz_checksum`.

Override any of them in `group_vars/all`.

## Helper task files

| File | Purpose |
| --- | --- |
| `resolve-version.yml` | Derive version, filename, and checksum from the download URL (or the legacy variables) |
| `fetch-tgz.yml` | Download the installer from Splunk, or upload it from `files/` |
| `boot-start-systemd.yml` | Run `splunk enable boot-start` with the arguments the target version accepts |
| `systemd-unit-fixes.yml` | Patch the generated unit file for Ubuntu, then reload systemd if it changed |
| `archive.yml` | Write a timestamped tgz into the backups directory |

Include one with, for example:

```yaml
- name: -TGZ- Fetch the Splunk installer
  ansible.builtin.include_role:
    name: splunk-common
    tasks_from: fetch-tgz
  vars:
    splunk_fetch_tgz: "{{ splunk_tgz }}"
    splunk_fetch_url: "{{ splunk_tgz_url }}"
    splunk_fetch_checksum: "{{ splunk_tgz_checksum_resolved | default('') }}"
    splunk_fetch_download: "{{ download_tgz_from_splunk_servers }}"
```

## Handlers

Notify `restart splunk` or `restart splunk uf` from any task whose change
needs Splunk to reload configuration. The handler picks systemd or the CLI
based on `install_method` / `uf_install_method`, and runs once at the end of
the play however many tasks notified it.
