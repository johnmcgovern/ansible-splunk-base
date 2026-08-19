# Changelog

All notable changes to this project are documented here.

This project had no releases before 2.0.0. Everything prior to it is referred
to as 1.x for convenience.

## [Unreleased]

## [2.0.1] - 2026-08-18

### Added

- Molecule + Docker CI that converges `os-config` in a container, asserts
  idempotence, and verifies the files it installs are present and that polkit
  is installed on both OS families. It runs on every push and pull request
  across a four-distro matrix - Ubuntu 22.04 and 24.04 (both polkit branches),
  Rocky Linux 9 and Amazon Linux 2023 - and is green on all four.
- meta/argument_specs.yml for the four roles that still lacked one -
  tls-config, uf-upgrade, backup-etc and backup-full - so every role except
  prereqs now validates its arguments before it runs. prereqs is the exception
  by necessity: it bootstraps python3 (with the raw module) before argument
  validation, itself a Python module, could run.

### Fixed

- The 443-to-8000 redirect for Splunk Web (Ubuntu and Amazon Linux) did not
  survive a reboot: it was a bare iptables rule added at play time with nothing
  to restore it, so after a reboot `https://host/` stopped working and the UI
  was reachable only at `https://host:8000/`. os-config now installs it as a
  boot-time oneshot unit (`splunk-web-redirect.service`), the same way it
  handles the THP tweak, so the redirect persists. Found when a rebooted lab
  host lost the rule.
- The installer could not be extracted on a host without `tar`. Ansible's
  `unarchive` shells out to `tar` to unpack the `.tgz`, and minimal RHEL-family
  images do not ship it, so install and upgrade failed at extraction with
  "Failed to find handler". `tar` is now installed before extraction, guarded
  behind a package-facts check so it is added only when missing (no dnf
  metadata refresh on a host that already has it). Found during live Rocky
  Linux 9.8 testing.
- RedHat-family tasks were gated on `ansible_distribution == 'Red Hat
  Enterprise Linux'`, but that fact reports `RedHat`, so the OS package update
  and the polkit installation never ran on RHEL, CentOS, or Amazon Linux - and
  would not have run on Rocky or Alma either. They are now gated on
  `ansible_os_family == 'RedHat'`, which covers all of them. (Latent since long
  before 2.0.0; found during live RHEL 9.7 testing.)
- The polkit install is now skipped when polkit is already present. It ships by
  default on the RedHat family, and `package: state=present` still refreshes
  dnf metadata, which failed on a host with no repo access even though nothing
  needed installing.

### Verified

- Splunk Universal Forwarder on Ubuntu 22.04 with systemd process management -
  the forwarder path the notes previously flagged as unexercised. uf-install,
  uf-config and uf-upgrade (10.4.1 to 10.4.2); the forwarder connects to a
  10.4.2 indexer on the receiving (9997) and deployment-server (8089) ports
  (splunkd logs "Connected to idx"), and uf-upgrade re-runs at `changed=0`.
- Splunk Enterprise on Rocky Linux 9.8 (a minimal image, no `tar`) with
  firewalld active: os-config, a full install of 10.4.1, and an in-place
  upgrade to 10.4.2, all through systemd boot-start with polkit rules. The
  upgrade re-runs at `changed=0`, and Splunk Web answers on tcp/8000.
- Splunk Enterprise on RHEL 9.7 with SELinux enforcing and firewalld active:
  os-config opens tcp/8000 through firewalld, and install runs with no SELinux
  denials. Both converge to `changed=0` on a repeat run.

## [2.0.0] - 2026-08-10

A modernization release. Splunk version selection no longer depends on a list
maintained by hand, the playbooks are safe to re-run, and the project works on
current Ansible and current Ubuntu. Several defaults changed - read
"Breaking changes" before upgrading.

Verified against Splunk 10.4.2 on Ubuntu 26.04 (Splunk Enterprise) and Ubuntu
22.04 (Universal Forwarder).

### Breaking changes

- **Playbooks target inventory groups instead of `all`.** Splunk Enterprise
  playbooks target `splunkhosts` and Universal Forwarder playbooks target
  `ufhosts`, so a UF playbook can no longer install a forwarder onto an indexer
  or search head. Define both groups (see `hosts.sample`), or pass
  `-e splunk_hosts=all` / `-e splunk_uf_hosts=all` for the old behaviour. These
  two must be given with `-e`; setting them in `group_vars` has no effect,
  because Ansible resolves a play's host pattern before loading those hosts'
  variables.

- **`os-config` no longer patches the OS.** 1.x always ran a full
  `apt dist-upgrade` / `yum update`, which could pull in a new kernel as a side
  effect of a configuration run. Set `os_update_packages: true` to restore it.

- **Splunk credentials are now required.** `splunk_user`, `splunk_pass`,
  `splunk_uf_user` and `splunk_uf_pass` have no defaults, and a run without them
  fails immediately at role entry rather than seeding a password nobody chose.

- **ulimits moved to a drop-in.** They install as
  `/etc/security/limits.d/99-splunk.conf` instead of replacing
  `/etc/security/limits.conf` wholesale. Entries written by 1.x are removed from
  `limits.conf` on the next `os-config` run, so the values stay defined in one
  place. Effective limits are unchanged.

- **`uf_install_method` controls forwarder process management**, separate from
  `install_method`, and defaults to `systemd`. This documents what 1.x actually
  did: it ran `enable boot-start` with no `-systemd-managed` flag, which modern
  Splunk defaults to systemd, so tasks labelled "initd" were creating
  `SplunkForwarder.service`. Selecting `initd` now passes `-systemd-managed 0`
  and produces a real init script.

- **Two collections are required** on a minimal `ansible-core` install:
  `ansible.posix` and `community.general`. Run
  `ansible-galaxy collection install -r requirements.yml`. The full `ansible`
  package already bundles them. In 1.x the `archive` and `firewalld` tasks could
  not resolve at all without them.

### Added

- **URL-first version selection.** Paste the `.tgz` URL from splunk.com into
  `splunk_download_url` / `splunk_uf_download_url`; the version, filename and
  sha512 checksum are derived from it, the checksum coming from the file Splunk
  publishes beside the tarball. Any past or future release works with no change
  to this project. The 1.x three-variable form still works.
- `splunk_email_allowed_domains`, which sets `allowedDomainList` in
  `alert_actions.conf` and clears the security warning Splunk shows when alert
  email may go to any domain.
- `splunk_remove_stale_files`, which removes files a previous Splunk release
  shipped that the current one does not. The list is derived from the previous
  release's manifest, never from the filesystem, so unmanifested files - indexes,
  `var/`, the KV store, `etc/*/local` - can never be selected. Run with
  `--check` first to see what would go.
- Role defaults and `meta/argument_specs.yml` for every role that takes input,
  validated before the role runs. A typo in `install_method` used to skip every
  boot-start and start task silently, leaving an installed but never-started
  Splunk behind a successful play.
- A README for each role, an `ansible.cfg`, a `requirements.yml`, and an
  ansible-vault walkthrough for the credentials.
- CI running `ansible-lint`, `yamllint` and a syntax check of every playbook.

### Changed

- **Playbooks are idempotent.** Re-running reports no changes when nothing
  needs doing. Restarts are handlers, so Splunk restarts only when configuration
  actually changed.
- **`install` is non-destructive**: an existing installation is never extracted
  over, and `user-seed.conf` is written only on a fresh install, so a re-run
  cannot reset the admin password of a running instance.
- **`upgrade` is version-guarded**: it does nothing when the installed version
  already matches the target, and it ensures the service is running even when
  the upgrade itself is skipped.
- **`uf-config` writes `deploymentclient.conf` directly** instead of running
  `splunk set deploy-poll`, which passed admin credentials on the command line
  where they were visible in the process list and in Ansible output.
- All actions use fully qualified module names, and facts are read from
  `ansible_facts` rather than the injected top-level variables that
  ansible-core 2.24 stops providing.
- Duplicated logic - the installer fetch, the boot-start ladder, the unit-file
  fixes, the backup archiving - now lives in one place.

### Fixed

- The cloud-init disable task never ran: it compared a registered result object
  to `True`, so the hostname was not preserved across reboots on cloud images.
- `[settings]` in several `lineinfile` patterns was an unescaped character
  class, matching any line starting with `s`, `e`, `t`, `i`, `n` or `g`.
- Splunk 7.2.0 and 7.2.1 matched two overlapping initd conditions and ran both
  `enable boot-start` variants.
- `policykit-1` does not exist on Ubuntu 24.04 and later; `polkitd` is installed
  there instead.
- The 443-to-8000 redirect appended a duplicate iptables rule on every run.
- The `yum` module was removed from ansible-core, so the RedHat-family tasks
  could not even parse on a current controller. They use `package` now.
- `tls-config` failed when run standalone, and could not enable
  `enableSplunkWebSSL` when the line was absent. It also forced `web.conf` to a
  mode Splunk immediately changed back, restarting splunkd on every run.
- The installer fetch computed an md5 of the multi-gigabyte tarball on every
  run to answer a question it never asked.
- Universal Forwarder: `uf-install` guarded boot-start on an init script that is
  never created, so a re-run failed; `uf-upgrade` did not clear the previous
  boot-start configuration and died after extraction; and a failed upgrade left
  the forwarder stopped while the version guard made every later run a no-op.

### Upgrading from 1.x

1. Install the collections: `ansible-galaxy collection install -r requirements.yml`
2. Add a `ufhosts` group to your inventory if you manage forwarders, and check
   your Splunk hosts are in `splunkhosts` - otherwise pass
   `-e splunk_hosts=<your group>`.
3. If you relied on `os-config` patching the OS, set `os_update_packages: true`.
4. Confirm the credential variables are set in your `group_vars`.
5. Optionally switch to URL-first version selection by replacing
   `splunk_version` / `splunk_tgz` / `splunk_tgz_checksum` with
   `splunk_download_url`.

Run any playbook with `--check` first. The first `os-config` run after
upgrading will move the ulimits into the drop-in file.
