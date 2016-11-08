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
- config.yml
	Main configuration file for installation process
- inventory
	Ansible inventory file. It doesn't exist by default (will be created automatically)
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
├── config.yml
├── inventory  (it doesn't exist by default)
├── playbooks
│   ├── step_1_cloudformation.yml
│   ├── step_2_pre_configure_systems.yml
│   └── step_3_openshift_install.yml
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
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           ├── hosts
│           ├── ifcfg-eth0.j2
│           ├── sysconfig.docker
│           ├── sysconfig.docker-storage
│           └── sysconfig.docker-storage-setup
└── templates
    ├── ansible.hosts.j2
    ├── inventory.j2
    ├── iptables.j2
    └── oauth.config.block

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
| localdomain     | example.li9.local      | Local DNS domain                                                               |
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
## Tasks outputs
 As part of preparation activities all nodes will be configured to host OpenShift services:
 - docker is up and running
 - docker storage is configured
 - all required packages are installed
 - hostsname are configured
 - local DNS server is up and running

# OpenShit installation

This project automates all installation procedures. To run the installation process it is enough to run the following:
```
ansible-playbook playbooks/step_3_openshift_install.yml
```
It will last ~ 30 mins
Once it is finished it will be possible to use openshift.
As a part of installation process ansible configures AllowAll Authentication provider. This means that OpenShift will allow all users at the first login.

## Verify installation

 - Connect to the master node
 - Check that all nodes exist and registry and router conteiners are up and running (please be aw
	```
	oc get nodes
	oc get pods
	systemctl status atomic-openshift-master
        systemctl status atomic-openshift-node
	```
 - Access the OpenShift service using  https://PublicIP:8443
 - Use Any username and any password (since AllowAll provider was configured as a part of installation)


