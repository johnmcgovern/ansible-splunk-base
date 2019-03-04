# Ansible-Splunk-Role

This is an Ansible role that installs a bare Splunk instance of a specified version.


### Setup

1. Install Ansible
 
		- sudo apt-get install ansible (Ubuntu) 
		- brew install ansible (macOS)

2. git clone this project

		- git clone https://github.com/johnmcgovern/ansible-splunk-role.git	
	
3. Navigate to project base directory

		- cd ./ansible-splunk-role		

4. Copy hosts.sample to hosts

		- cp hosts.sample hosts

5. Edit hosts file to include desired hosts

		- vi hosts
	
6. Copy group_vars/all.sample to group_vars/all

		- cp group_vars/all.sample group_vars/all

7. Edit group_vars/all variables as appropriate for your enviornment

		- vi group_vars/all


### Requirements

1. The Ubuntu target server must already have python installed (Ansible needs it to operate)

		- sudo apt-get install python


### Usage
	
1. Navigate to playbook base directory

		- cd ./ansible-splunk-role
	
2. Run the Ansible playbook

		- ansible-playbook -i hosts site.yml
	
3. Run the Ansible playbook limited to certain hosts

		- ansible-playbook -i hosts --limit=host1 site.yml  #limits to a subset of hosts


### Compatibility

This role is tested on:

- Ubuntu 18.04 Server (LTS)


### Notes

- The goal of this role is to get to execute a best-practices base Splunk install (including support for Workload Management which is a departure from the standard install).


### ToDo

- Upgrade in place support.
- Support for additional server settings.
- Support for roles such as indexer, standalone search head, search head cluster, cluster master, license master, deployment server, deployer, monitoring console, etc.
- Automated app install.
- Simplified version/file/hash dictionary.


### Contact

- john@johnmcgovern.com
- https://www.johnmcgovern.com
