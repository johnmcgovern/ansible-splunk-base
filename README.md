# Ansible-Splunk-Base

This is an Ansible project that installs or upgrades Splunk to a specific version. It can also perform basic OS config (ulimits, THP disabled, hostname, etc.), ./splunk/etc/ backups, and SSL cert installation.


### Setup

1. Install Ansible
 
		- sudo apt-get install ansible (Ubuntu) 
		- brew install ansible (macOS)
		- pipx install ansible-core (any platform, isolated)

2. git clone this project

		- git clone https://github.com/johnmcgovern/ansible-splunk-base.git
	
3. Navigate to project base directory

		- cd ./ansible-splunk-base		

3a. Install the required collections

	This project uses ansible.posix (firewalld) and community.general (archive)
	in addition to ansible.builtin. The full "ansible" package already bundles
	them; a minimal "ansible-core" install does not.

		- ansible-galaxy collection install -r requirements.yml

4. Copy hosts.sample to hosts

		- cp hosts.sample hosts

5. Edit hosts file to include desired hosts

		- vi hosts
	
6. Copy group_vars/all.sample to group_vars/all

		- cp group_vars/all.sample group_vars/all

7. Edit group_vars/all variables as appropriate for your environment

		- vi group_vars/all


### Selecting a Splunk Version

Version selection is URL-first: paste the full .tgz download URL for the
version you want into group_vars/all and everything else (version number,
filename, sha512 checksum) is derived automatically. Any past or future
Splunk version works without changes to this project.

1. Find your version on splunk.com and copy the direct download ("wget") link
   for the Linux .tgz (x86_64/amd64):

		- Splunk Enterprise:   https://www.splunk.com/en_us/download/splunk-enterprise.html
		- Universal Forwarder: https://www.splunk.com/en_us/download/universal-forwarder.html
		- Older releases:      https://www.splunk.com/en_us/download/previous-releases.html

2. Set it in group_vars/all:

		- splunk_download_url: https://download.splunk.com/products/splunk/releases/10.4.2/linux/splunk-10.4.2-33c3bf42cd73-linux-amd64.tgz
		- splunk_uf_download_url: https://download.splunk.com/products/universalforwarder/releases/10.4.2/linux/splunkforwarder-10.4.2-33c3bf42cd73-linux-amd64.tgz

The tgz checksum is verified against the sha512 published alongside it on
download.splunk.com. To pin a checksum explicitly (e.g. for air-gapped or
change-controlled environments), set splunk_tgz_checksum / splunk_uf_tgz_checksum
to `sha512:<hash>`, `sha256:<hash>`, or `md5:<hash>`.

The legacy three-variable form (splunk_version + splunk_tgz +
splunk_tgz_checksum with a bare md5 hash) from earlier releases of this
project is still fully supported - see the commented block at the bottom of
group_vars/all.sample.


### Usage
	
1. Navigate to playbook base directory

		- cd ./ansible-splunk-base

	The bundled ansible.cfg sets the default inventory to ./hosts, so "-i hosts"
	is optional from the project directory. It is shown below for clarity and
	for anyone running from elsewhere.
	
2. Run the Splunk install playbook

		- ansible-playbook -i hosts install.yml

3. -or- run the Splunk upgrade playbook

		- ansible-playbook -i hosts upgrade.yml

4. -or- run the Splunk OS initial configuration playbook (built to provide a simple configuration for lab hosts)

		- ansible-playbook -i hosts os-config.yml

5. -or- run a base OS config AND install Splunk.

		- ansible-playbook -i hosts combo.yml	

6. -or- configure an TLS/SSL key pair for the web UI (tcp/8000).

		- ansible-playbook -i hosts tls-config.yml						

7. -or- run the Splunk UF install playbook

		- ansible-playbook -i hosts uf-install.yml

8. -or- run the Splunk UF config playbook

		- ansible-playbook -i hosts uf-config.yml

9. -or- run the Splunk UF install AND config playbook

		- ansible-playbook -i hosts uf-combo.yml	

10. -or- run the Splunk UF upgrade playbook

		- ansible-playbook -i hosts uf-upgrade.yml					

11. -or- run the Splunk configuration only (./etc/) backup playbook

		- ansible-playbook -i hosts backup-etc.yml

12. -or- run the Splunk full backup (/opt/splunk/) playbook

		- ansible-playbook -i hosts backup-full.yml		

13. Run an Ansible playbook limited to certain hosts within the hosts list

		- ansible-playbook -i hosts --limit=host1 install.yml

14. Run multiple playbooks in one command

		- ansible-playbook -i hosts os-config.yml install.yml tls-config.yml


### Which Hosts a Playbook Targets

Splunk Enterprise and Universal Forwarder playbooks target separate inventory
groups, so a UF playbook cannot install a forwarder onto an indexer or search
head:

| Playbooks | Default group |
| --- | --- |
| install, upgrade, os-config, combo, tls-config, backup-etc, backup-full | `splunkhosts` |
| uf-install, uf-config, uf-combo, uf-upgrade | `ufhosts` |

Define both groups in your hosts file - see hosts.sample.

To target something else, pass splunk_hosts or splunk_uf_hosts as an extra
variable. Any inventory pattern works: a group, a host, or a comma-separated
list.

		- ansible-playbook -i hosts install.yml -e splunk_hosts=all
		- ansible-playbook -i hosts install.yml -e splunk_hosts=indexers
		- ansible-playbook -i hosts uf-install.yml -e splunk_uf_hosts=my-forwarder.splk.me

These two must be passed with -e. Setting them in group_vars has no effect,
because Ansible resolves which hosts a play targets before it loads the
variables belonging to those hosts.

Earlier releases targeted "all" from every playbook. If you relied on that,
"-e splunk_hosts=all" reproduces it, or you can rename your inventory group to
splunkhosts.

"--limit" still works as always, and narrows whatever the playbook targets.


### Protecting the Splunk Credentials

group_vars/all holds splunk_pass and splunk_uf_pass in plain text. To encrypt
them, turn group_vars/all into a directory and split the secrets into their own
encrypted file. Ansible reads every file in group_vars/all/, so both are picked
up automatically.

Note that an extra file such as group_vars/all_secrets.yml is NOT read -
Ansible would treat that as variables for a group called "all_secrets", which
does not exist, and your settings would be silently ignored.

1. Turn the file into a directory

		- mkdir group_vars/all.tmp
		- mv group_vars/all group_vars/all.tmp/vars.yml
		- mv group_vars/all.tmp group_vars/all

2. Create the encrypted file beside it

		- ansible-vault create group_vars/all/vault.yml

3. Put the credentials in it, and delete them from group_vars/all/vars.yml

		splunk_pass: yourRealPassword
		splunk_uf_pass: yourRealPassword

4. Run any playbook with the vault password

		- ansible-playbook -i hosts install.yml --ask-vault-pass

	Or point at a password file instead of typing it:

		- ansible-playbook -i hosts install.yml --vault-password-file ~/.splunk-vault-pass

Edit it later with "ansible-vault edit group_vars/all/vault.yml". To confirm a
file really is encrypted, check that it starts with "$ANSIBLE_VAULT":

		- head -1 group_vars/all/vault.yml

.gitignore excludes group_vars/all, which covers the directory and everything
in it, so neither file is committed.


### Role Documentation

Each role has a README describing what it does and the variables it takes, and
a meta/argument_specs.yml that Ansible validates before the role runs:

- roles/splunk-common - shared variables, handlers, and helper task files
- roles/core-install, roles/core-upgrade - Splunk Enterprise
- roles/uf-install, roles/uf-config, roles/uf-upgrade - Universal Forwarder
- roles/os-config, roles/tls-config - host and TLS configuration
- roles/backup-etc, roles/backup-full - backups
- roles/prereqs - python3 bootstrap

prereqs is the one exception with no argument_specs.yml: it runs before python
exists (that is what it installs, using the raw module), and argument
validation is itself a Python module, so a spec there could not run. It takes
no variables of its own.


### Compatibility

This role has been tested on:

- Ubuntu 26.04, 22.04, 20.04, & 18.04 Server (LTS)
- Rocky Linux 9.8 (the primary RHEL-family target)
- RHEL 9.7 & 8
- CentOS 7 1810
- Amazon Linux 2 2022.06 & 2020.04

Most recently verified against Splunk 10.4.2:

- Rocky Linux 9.8 - Splunk Enterprise, the primary RHEL-family target. On a
  minimal image (no `tar`), os-config, a full install of 10.4.1, and an
  in-place upgrade to 10.4.2, through systemd boot-start with polkit rules and
  firewalld active. The upgrade re-runs at `changed=0`.
- Ubuntu 26.04 - Splunk Enterprise. os-config, install, combo, upgrade
  (9.4.2 to 10.4.2, including stale file removal), tls-config, backup-etc.
- Ubuntu 22.04 - Universal Forwarder. uf-install, uf-config, uf-upgrade
  (9.4.2 to 10.4.2), forwarding to a 10.4.2 indexer and confirmed searchable
  there.
- RHEL 9.7 - Splunk Enterprise. os-config and install, with SELinux enforcing
  and firewalld active. os-config opens tcp/8000 through firewalld; install
  runs under SELinux with no denials.

The remaining platforms were tested against earlier Splunk releases and have
not been re-checked since.


### Testing

Every push and pull request runs two GitHub Actions workflows:

- **Lint** - `yamllint`, `ansible-lint` (production profile), and a syntax check
  of every playbook.
- **Molecule** - converges `os-config` in a container on Ubuntu 22.04, Ubuntu
  24.04, Rocky Linux 9 and Amazon Linux 2023, re-runs it to confirm the second
  run reports no changes, and verifies the files it installs are present and
  that polkit is installed. This is the automated form of the manual "run it
  twice, check for `changed=0`" check, run across distributions so every
  OS-specific branch is exercised on every change:

  | Image | Path it covers |
  | --- | --- |
  | ubuntu2204 | Debian family, `policykit-1` (before 24.04) |
  | ubuntu2404 | Debian family, `polkitd` (24.04 and later) |
  | rockylinux9 | RedHat family, dnf/polkit, no iptables redirect |
  | amazonlinux2023 | RedHat family *and* the iptables 443-to-8000 redirect |

  These four map onto the Linux families Splunk supports (Ubuntu, the
  RHEL/CentOS/Rocky/Alma/Oracle family, and Amazon Linux) and between them
  reach every conditional branch in `os-config`. SUSE (SLES) is on Splunk's
  supported list but is deliberately out of scope: this project has no
  zypper/SUSE code path, so it is not claimed and not tested.

Molecule covers the roles that need no live Splunk download. The install and
upgrade roles pull multi-gigabyte tarballs and manage a real splunkd, so they
stay on live verification against a real host (see Compatibility above).

To run Molecule locally you need Docker:

		- pip install ansible-core molecule "molecule-plugins[docker]" docker
		- ansible-galaxy collection install community.docker
		- molecule test                        # default target (Ubuntu 22.04)
		- MOLECULE_DISTRO=rockylinux9 molecule test


### Notes

- The goal of this role is to quickly execute a best-practices base Splunk install/upgrade (including support for Workload Management, which is a departure from the previous install method).
- There are more complex/full-featured projects out there for various deployment topologies. The goal here is simplicity, speed, and utility.
- 8.1.1 introduced PolicyKit (polkit) management of systemd processes which allows for splunk to be restarted (for example) as the splunk user or super user using the commands "splunk restart", "systemctl restart Splunkd", and "sudo systemctl restart Splunkd" for maximum flexibility.
- Both "systemd" and "initd" methods of Linux process management are supported. systemd is ONLY available in Splunk Enterprise version 7.2.2 and later. 
- Splunk versions 7.2.2 - 7.2.x implement "enable boot-start" differently than 7.3.0 and later. This is now accounted for.
- Assuming a semi-default install (such as you would find if you installed with this playbook), upgrade.yml will convert from initd process management to systemd process management if you flag "systemd" on install_method.
- A number of config items are set which disable pop-ups and modal dialogues which would normally be shown to the Splunk admin and/or users such as new version available notifications, UI tours, and python 2.7 deprecation notifications. The goal here is to generally avoid UI annoyances that would crop up in automatic distributed Splunk deployments.
- This Ansible playbook does not currently handle OS-level firewall allowances for splunkd TCP ports.
- os-config.yml no longer patches the OS unless you ask it to. Earlier releases always ran a full dist-upgrade, which could pull in a new kernel as a side effect of a config run. Set os_update_packages to true to restore that behaviour.
- ulimits are installed as /etc/security/limits.d/99-splunk.conf rather than by replacing /etc/security/limits.conf. Hosts configured by an earlier release have the old entries removed from limits.conf on the next os-config run. Note that under systemd, splunkd takes its limits from the Splunkd.service unit; the PAM limits apply to initd startup and to interactive sessions as the splunk user.
- The Universal Forwarder has its own uf_install_method variable, separate from install_method, and it defaults to initd. Unlike the Splunk Enterprise path, UF systemd management has not yet been exercised against a real forwarder - test it before relying on it.
- We bias towards being non-destructive. For example, if we see an existing/previous Splunk install we will fail out rather than damage the current install. 

### To-Do

- Support for additional server settings.


### Warranty

This project is provided WITHOUT any form of warranty and should be tested thoroughly before using it in your environment. Development is best-effort only. This project is provided as-is with no guarantee as to fitness for a specific purpose. Please use it at your own risk.


### Contact

- john@johnmcgovern.com or https://www.johnmcgovern.com
