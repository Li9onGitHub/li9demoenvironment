# li9demoenvironment
Li9 Customer Demo Environment


This project automates initial configuration of OpenShift 3.x nodes (masters and nodes).
# Prerequisites
- OS: Linux
- Ansible installed
- git

# Usage
1. Pull source code
```
cd ~
git clone  https://github.com/Li9onGitHub/li9demoenvironment.git
cd li9demoenvironment
```

2. Configure AWS access shell variables
Note! This parametres can be found on AWS portal
```
export AWS_ACCESS_KEY='...'
export AWS_SECRET_ACCESS_KEY='+...'
```

3. Run AWS infrastrucure provisioning
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

