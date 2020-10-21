This is a Demo for Airwallex DevOps Tests
-----
The original repo with examples and codes could be found in below [section](#tutorial-cli-ansible)

# Intro
## Targets
1. Creating and provision an AWS EC2 t2.micro or Aliyun ECS t5-lc1m1.small instance, with an OS of your own choice.
2. Change the security group of the instance to ensure it’s security level.
3. Change the OS/Firewall settings of the started instance to further enhance it’s security level.
4. Install Docker CE.
5. Deploy and start an nginx container in docker.
6. Run a command to test the healthiness of the nginx container.
7. Fetch the output of the nginx container’s default http page.
8. Strip all the html tags, grep and count all the words in the fetched html, then print the result in the following manner, sort the result in alphabet order.
9. Logs the resource usage of the container every 10 seconds.

## Design
- using Aliyun to implement the test
- using Ubuntu 18.04 LTS with better OS hardening support
- using roles and plays in order to organize the tasks and achieve separation of concerns
- version freeze for most of the key components involved to increase the success rate of the automation
- put os security and sec group at last to simpify the prepartion tasks

## Steps
----
solution tested with:
- Python 3.8.6
- all the command ran in the root folder of this folder
----
1. pip packages installation
* virtualenv is strongly suggested to do this
```shell
pip install -r pip_requirement.txt
```
2. ansible-galaxy collections installation
```shell
ansible-galaxy install -r galaxy_requirement.yml
```
3. initial ALICLOUD access
* change value of ALICLOUD_ACCESS_KEY and ALICLOUD_SECRET_KEY in access_init_sample.sh and 
```shell
source access_init_sample.sh 
```

4. create cloud resources
* **one key pair used here, named adt in the var**
* change value of "key_name" in ansible/group_vars/all if needed
```shell
ansible-playbook ansible/create.yml
```
5. deploy business components, nginx in this case, and some validation steps
* after this step, nginx can only be accessed within the VM, need step 7 to reach to it from internet
```shell
ansible-playbook -i ansible/alicloud.py ansible/deploy_nginx.yml
```
* ssh login to check monitoring data
```shell
tail -f /var/log/docker_stats.log
```
6. os harderning with multiple methods/aspects
* for this step to work, **assumption is under control nodes folder**: ~/.ssh/ there is id_rsa and id_rsa.pub pair
* after this step, only ubuntu user could login, and need step 7 finished to get firewall accept the remote ssh connection
```shell
ansible-playbook -i ansible/alicloud.py ansible/os_security_hardening.yml
```
7. cloud VM security group adoption with os hardening changes
```shell
ansible-playbook ansible/sec_group_hardening.yml
```
* new port for ssh is 37, user is ubuntu, private key is ~/.ssh/id_rsa
* nginx access url is http://public_ip_allocated:8787


------
------
------
# tutorial-cli-ansible
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 

**This is the original repo demo just for reference**

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

A tutorial for how to use ansible.

Click the open in Cloud Shell button below to run a sample tutorial yourself.

<a href="https://shell.aliyun.com/?action=git_open&git_repo=https://code.aliyun.com/labs/tutorial-cli-ansible.git&tutorial=tutorial-zh.md" target="cloudshell_tutorial_cli_ansible">
  <img src="https://img.alicdn.com/tfs/TB1wt1zq9zqK1RjSZFpXXakSXXa-1066-166.png" width="180" />
</a>

