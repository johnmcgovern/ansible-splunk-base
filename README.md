# Ansible-Splunk-Base

This is an Ansible project that installs or upgrades Splunk to a specific version. It can also perform basic OS config (ulimits, THP disabled, hostname, etc.) and ./splunk/etc/ backups.


### Setup

1. Install Ansible
 
		- sudo apt-get install ansible (Ubuntu) 
		- brew install ansible (macOS)

2. git clone this project

		- git clone https://github.com/johnmcgovern/ansible-splunk-base.git
	
3. Navigate to project base directory

		- cd ./ansible-splunk-base		

4. Copy hosts.sample to hosts

		- cp hosts.sample hosts

5. Edit hosts file to include desired hosts

		- vi hosts
	
6. Copy group_vars/all.sample to group_vars/all

		- cp group_vars/all.sample group_vars/all

7. Edit group_vars/all variables as appropriate for your enviornment

		- vi group_vars/all


### Usage
	
1. Navigate to playbook base directory

		- cd ./ansible-splunk-base
	
2. Run the Splunk install playbook

		- ansible-playbook -i hosts install.yml

3. -or- run the Splunk upgrade playbook

		- ansible-playbook -i hosts upgrade.yml

4. -or- run the Splunk configuration backup playbook

		- ansible-playbook -i hosts backup.yml

5. -or- run the Splunk OS initial configuration playbook

		- ansible-playbook -i hosts os-config.yml

6. Run the Ansible playbook limited to certain hosts within the hosts list

		- ansible-playbook -i hosts --limit=host1 install.yml


### Compatibility

This role is tested on:

- Ubuntu 20.04 Server (LTS)
- Ubuntu 18.04 Server (LTS)
- RHEL 8
- CentOS 7 1810
- Amazon Linux 2 2020.04


### Notes

- The goal of this role is to quickly execute a best-practices base Splunk install/upgrade (including support for Workload Management which is a departure from the previous install method).
- Both "systemd" and "initd" methods of Linux process managemenent are supported. systemd is ONLY available in Splunk Enteprise version 7.2.2 and later. 
- Splunk versions 7.2.2 - 7.2.x implement "enable boot-start" differently than 7.3.0 and later. This is now accounted for.
- Assuming a semi-default install (such as you would find if you installed with this playbook), upgrade.yml will convert from initd process management to systemd process management if you flag "systemd" on install_method.
- A number of config items are set which disable pop-ups and modal dialogues which would normally be shown to the Splunk admin and/or users such as new version available notifications, UI tours, and python 2.7 deprication notifications. The goal here is to generally avoid UI annoyances that would crop up in automatic distributed Splunk deployments.
- This Ansible playbook does not currently handle OS level firewall allowances for splunkd TCP ports.

### To-Do

- Optional firewall allowances for splunkd.
- Support for additional server settings.
- Support for roles such as indexer, standalone search head, search head cluster, cluster master, license master, deployment server, deployer, monitoring console, etc.
- Automated app install.
- Simplified version/file/hash dictionary.


### Contact

- john@johnmcgovern.com or https://www.johnmcgovern.com
