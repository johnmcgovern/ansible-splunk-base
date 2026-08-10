# core-upgrade

Upgrades Splunk Enterprise: extracts the target release over `$SPLUNK_HOME`,
rebuilds boot-start, and restarts Splunk.

Used by `upgrade.yml`.

## Safe to re-run

The role reads `$SPLUNK_HOME/etc/splunk.version` first and does nothing when
the installed version already matches the target, so running the playbook
twice costs one fact-gathering pass and nothing else.

## Variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `splunk_remove_stale_files` | `false` | Remove files a previous release shipped that the current one does not |

Plus the shared variables from `splunk-common`.

## Removing stale files

Extracting a tarball over an installation replaces what the new release ships
but cannot remove what it dropped, so old python, mongod and library versions
accumulate and Splunk's file integrity check reports them as
`present_but_shouldnt_be`. Splunk provides no cleanup for this: `splunk clean`
only touches data, and `splunk validate files` is report-only.

Set `splunk_remove_stale_files: true` to have this role tidy them. It runs
outside the version guard, so it also cleans a host already at the target
version.

**The deletion list is derived from the previous release's manifest, never
from the contents of the filesystem.** A file is removed only when a previous
manifest listed it, the current manifest does not, and it is still on disk -
so indexes, `var/`, the KV store and `etc/*/local` can never be selected.
Removal is scoped to the top level of `bin/` and `lib/`, matching what Splunk
reports, and afterwards `splunk validate files` must still report the install
intact or the play fails.

Run with `--check` first to list exactly what would go:

```bash
ansible-playbook -i hosts upgrade.yml --check -e splunk_remove_stale_files=true
```
