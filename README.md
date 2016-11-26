# openshiftdemo
OpenShift demo deployment automation


This project automates initial configuration of OpenShift 3.x nodes (masters and nodes) at AWS environment
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
- roles
	Directory which contains required ansible roles
- playbooks
	Directory which contains ansible playbooks used for automation purposes
- templates
	File templates used by ansible template module
- cloudformation
	This directory contains cloudformation templates
```

.
├── ansible.cfg
├── cloudformation
│   └── openshift.json
├── config.yml
├── playbooks
│   ├── deploy_openshift.yml
│   ├── step_0_send_startservice_email.yml
│   ├── step_1_cloudformation.retry
│   ├── step_1_cloudformation.yml
│   ├── step_2_register_on_redhat.yml
│   ├── step_3_apply_openshiftcommon_role.yml
│   ├── step_4_pre_configure_systems.yml
│   ├── step_5_openshift_install.yml
│   └── step_6_send_servicedeployed_email.yml
├── README.md
├── roles
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
    ├── iptables.j2
    └── oauth.config.block



```

# Usage
1. **Pull source code**

	```
	cd ~
	git clone  https://github.com/Li9onGitHub/li9demoenvironment.git
	cd li9demoenvironment
	```

2. **Get AWS credentials**

	Automation requires access to AWS using ACCESS and SECRET_ACCESS keys. This parametres can be found on AWS portal.

3. **Configure installation parameters**

	All installation parameters are stored in config.yml but usually there are no needs to change them.
	```
	vi config.yml
	```

4. **Deploy AWS OpenShift**
	
	To deploy required instances in AWS run  openshift_install.yml playbook as shown below
	```
	ansible-playbook -e 'email=<USEREMAIL> domaintag=<DOMAINTAG> redhat_user=<REDHATUSER>  redhat_password=<REDHATPASSWORD> aws_access_key=<AWS_ACCESS_KEY> aws_secret_key=<AWS_SECRET_KEY>' playbooks/step_1_cloudformation.yml
	```
NOTE! The following parameters are mandatory and must be specified:

- **redhat_username** (Red Hat user who has OpenShift subscription)
- **redhat_password** (Red Hat password)
- **aws_access_key** (AWS ACCESS KEY)
- **aws_secret_key** (AWS SECRET ACCESS KEY)
- **domaintag**  - domaintag is 3 letters uniqie ID for newly created domain (this means that <domaintag>.demo.li9.com zone will be created)

Optional parameters:
- **email** - email to send notifications (if it is not send - email will not be sent)

	Example:
	```
	ansible-playbook -e 'email=myuser@example.com domaintag=yfs redhat_user=someredhatlogin redhat_password=StrongPwd123 aws_access_key=PPAJ2CVAKPCXFOUTHHA aws_secret_key=+QCXt3nHQbR9QUioGg1taLOIGy46V9CtbaRsjimM' playbooks/step_1_cloudformation.yml	
	```

It will need ~ 25 .. 30  minutes to be completed.  Once it is finished it will be possible to use openshift.  As a part of installation process ansible configures AllowAll Authentication provider. This means that OpenShift will allow all users at the first login.


# Actions and outputs

These automation templates deploys OpenShift with all required AWS infrastrucure items. The templates do the following (TAG is domaintag):

- send user an email notification that the requests is registred
- create AWS keypair with name idrsa-TAG
- download private key locally as ~/idrsa-TAG.pem
- apply cloudformation template cloudformation/openshift.json with the following outputs:
	- VPC (10.0.0.0/16)
	- Subnet (10.0.0.0/24)
	- EC2 instances:
		- OpenShiftMaster
		- OpenShiftNode01
		- OpenShiftNode02
	- EC2 Volumes (as /dev/xvdh) for each instance)
	- Several Security Groups
	- DNS records (type "A") which points to Master PublicIP
		- TAG.demo.li9.com 
		- master01.TAG.demo.li9.com
		- *.TAG.demo.li9.com

- send user an email notification that CloudFormation step is done
- register each system on Red Hat portal (+apply proper OpenShift-related repositories)
- do general node configuration:
	- docker installation and configuration
	- installation of additional software
- configure OpenShift installer (create ansible inventory file on master node)
- run OpenShift installer
- send user an email notification that demo environment is ready to use


# Installation verification


If everything is completed successfully user will receive an email which contains all required information to use the environment

 - Connect to the master node
 - Check that all nodes exist and registry and router conteiners are up and running

	```
	oc get nodes
	oc get pods
	systemctl status atomic-openshift-master
	systemctl status atomic-openshift-node
	
	```

 - Access the OpenShift service using  https://PublicIP:8443 or https://TAG.demo.li9.com
 - Use Any username and any password (since AllowAll provider was configured as a part of installation)

