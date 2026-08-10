# backup-full

Archives all of `$SPLUNK_HOME`, including index data, into
`/home/<os_user>/backups/`.

Used by `backup-full.yml`.

The archive is named `splunk-full-<inventory_hostname>-<date>--<time>.tgz`,
timestamped from the remote host's clock, so each run produces a new file
rather than overwriting the last one. Old archives are never pruned, and a
full backup can be large - budget the disk.

> **Consistency:** this archives `$SPLUNK_HOME` while splunkd is running, so
> hot buckets and the KV store are not guaranteed to be in a consistent state.
> Stop Splunk first if you need a restore-grade backup.

The work is done by `splunk-common`'s `archive.yml`, which this role calls with
`backup_source` and `backup_label`.
