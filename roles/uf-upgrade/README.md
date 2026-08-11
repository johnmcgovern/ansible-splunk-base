# uf-upgrade

Upgrades the Splunk Universal Forwarder.

Used by `uf-upgrade.yml`.

## Safe to re-run

The role reads `$SPLUNK_UF_HOME/etc/splunk.version` first and does nothing when
the installed version already matches the target.

## Variables

Takes the shared variables from `splunk-common`, in particular
`splunk_uf_download_url` and `uf_install_method`.

`uf_install_method` defaults to `systemd`; set it to `initd` for a real init
script.
