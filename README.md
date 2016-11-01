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

# Directory structure
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
```
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
```

# Usage - create instances
1. **Pull source code**

	```
	cd ~
	git clone  https://github.com/Li9onGitHub/li9demoenvironment.git
	cd li9demoenvironment
	```
2. **Configure AWS access**

	Automation requires access to AWS using ACCESS and SECRET_ACCESS keys. This parametres can be found on AWS portal.
	You should configure shell variables before using automation.
	```
	export AWS_ACCESS_KEY='...'
	export AWS_SECRET_ACCESS_KEY='+...'
	```

3. **Configure installation parameters**
	```
	vi config.yml
	```
Here you are able to specify the following:

| Name            | Default Value          | Description                                                                    |
|-----------------|------------------------|--------------------------------------------------------------------------------|
| publicdomain    | example.li9.com        | Public DNS domain                                                              |
| localdomain     | example.li9.local      | Local DNS domain. Will be applied to create local hosts file and set hostnames |
| KeyName         | openshift_aws          | Key Pari name (for AWS EC2)                                                    |
| StackName       | ansibleopenshift       | StackName for CloudFormation                                                   |
| root_public_key | ~/openshift_public.key |                                                                                |
| docker_disk     | /dev/xvdh              | Path to block device                                                           |


4. **Deploy AWS OpenShift Infrastructure**
	
	To deploy required instances in AWS run step_1_cloudformation.yaml  playbook
	```
	ansible-playbook playbooks/step_1_cloudformation.yml
	```
Note! it is possible to define StackName as an ansible variable (ansibleopenshift will be used by default if it is not specified)
	```
	ansible-playbook -e "StackName=MyOpenShift" playbooks/step_1_cloudformation.yml
	```

## tasks outputs
step_1_cloudformation.yml playbook does the following:
- creates keypair openshift_aws at AWS EC2
	This keypair will be used for all instances
- downloads private key as ~/openshift_aws.pem
- applies cloudformation template cloudformation/openshift.json

Once it is finshed the following resources should be available:
- VPC (10.0.0.0/16)
- Subnet (10.0.0.0/24)
- EC2 instances:
	- OpenShiftMaster
	- OpenShiftNode01
	- OpenShiftNode02
- EC2 Volumes (9G additional storage(as /dev/xvdh) for each instance)
- Several Security Groups
- openshift_aws.pem which is available in user home directory
- "inventory" file located in root project directory

## verify
Since inventory file is available ansible should be able to reach all nodes (1xMaster, 2xNodes)
	```
	ansible -m ping all
	``` 

# Usage - configure instances
1. **Get Red Hat OpenShift subscription**
 Automation uses Red Hat subscription to install OpenShift packages. As part for the subscription the following information must be available:
 - username
 - password

2. **Run pre configuration playbook**
	```
	ansible-playbook -e 'redhat_user=<REDHATUSER>  redhat_password=<REDHATPASSWORD>' playbooks/step_2_pre_configure_systems.yml		 
	```
	Example:
	```
	ansible-playbook -e 'redhat_user=artemi.kropachev@li9.com redhat_password=MyPassword' playbooks/step_2_pre_configure_systems.yml
	```

