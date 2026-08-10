# uf-upgrade

Upgrades the Splunk Universal Forwarder.

Used by `uf-upgrade.yml`.

## Safe to re-run

The role reads `$SPLUNK_UF_HOME/etc/splunk.version` first and does nothing when
the installed version already matches the target.

## Variables

Takes the shared variables from `splunk-common`, in particular
`splunk_uf_download_url` and `uf_install_method`.

> **Untested:** the UF systemd path (`uf_install_method: systemd`) has not yet
> been exercised against a real forwarder. Test it before relying on it.
