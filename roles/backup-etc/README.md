# backup-etc

Archives `$SPLUNK_HOME/etc` - the Splunk configuration - into
`/home/<os_user>/backups/`.

Used by `backup-etc.yml`.

The archive is named `splunk-etc-<inventory_hostname>-<date>--<time>.tgz`,
timestamped from the remote host's clock, so each run produces a new file
rather than overwriting the last one. Old archives are never pruned.

The work is done by `splunk-common`'s `archive.yml`, which this role calls with
`backup_source` and `backup_label`.
