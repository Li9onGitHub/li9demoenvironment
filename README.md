# li9demoenvironment
Li9 Customer Demo Environment


This project automates initial configuration of OpenShift 3.x nodes (masters and nodes).
# Prerequisites (Automation host)
- Linux
- Ansible
- git

# Preparations (RHEL, CentOS, Fedora)
- Install ansible and git
```
yum -y install ansible git
```

# Directory strucutre
- ansible.cfg
	Local Ansible configiration file
- inventory
	Ansible inventory file. Doesn't exist by default (will be created automatically)
- roles
	Directory which contains required ansible roles
- playbooks
	Directory which contains ansible playbooks used for automation purposes
- templates
	File templates used by ansible template module

├── ansible.cfg
├── cloudformation
│   └── openshift.json
├── playbooks
│   ├── step_1_cloudformation.yml
│   └── step_2_pre_configure_systems.yml
├── README.md
├── roles
│   ├── dnsserver
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       ├── 0.0.10.in-addr.arpa
│   │       ├── local.zone
│   │       ├── named.conf
│   │       └── public.zone
│   └── openshiftcommon
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   └── ifcfg-eth0
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           ├── hosts
│           ├── sysconfig.docker
│           ├── sysconfig.docker-storage
│           └── sysconfig.docker-storage-setup
└── templates
    └── inventory.j2

# Usage
1. Pull source code
	```
	cd ~
	git clone  https://github.com/Li9onGitHub/li9demoenvironment.git
	cd li9demoenvironment
	```
2. Configure AWS access
	Automation requires access to AWS using ACCESS and SECRET_ACCESS keys. This parametres can be found on AWS portal.
	You should configure shell variables before using automation.
	```
	export AWS_ACCESS_KEY='...'
	export AWS_SECRET_ACCESS_KEY='+...'
	```

3. Deploy AWS OpenShift Infrastructure
	
	```
	ansible-playbook aws_run_cloudformation.yml 
	```
Note! it is possible to define StackName as an asible variale
```
ansible-playbook -e "StackName=MyOpenShift" aws_run_cloudformation.yml
```

4. Check Stack Outputs (external IP addresses of nodes)

5. Modify inventory (IP addresses of servers)

```
vim inventory
```
Example:
```
[master]
52.201.241.253 hostname=master01.li9.local

[node01]
54.147.194.213  hostname=node01.li9.local

[node02]
54.158.136.50 hostname=node02.li9.local
...
```

