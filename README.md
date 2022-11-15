# Red Hat Enterprise Linux Standard Operating Environment automation

The Red Hat Enterprise Linux Standard Operating Environment (SOE) automation project is intended for effectively demonstrating the utilization of Red Hat Ansible Automation Platform (AAP) and Red Hat Satellite for the implementation of a Red Hat Enterprise Linux Standard Operating Environment (SOE).

## Access to AAP and Satellite golden images

The Red Hat Enterprise Linux SOE automation project utilizes golden images of a Red Hat Ansible Automation Platform system and a Red Hat Satellite system. Images shared from other AWS accounts can be utilized or golden images can be built.

**golden images shared from other accounts**

If using golden images shared from other accounts, record a) AWS account number that owns the EC2 AMI image shared and b) the EC2 AMI name of the shared image. Have this information ready for use later in the deploy instructions.

**golden images shared from other accounts**

The following GitHub repositories contain build automation for creating EC2 AMI images that will be located within the AWS account.

*Red Hat Ansible Automation Platform EC2 AMI build automation:*<br>
*- https://github.com/heatmiser/packer-ansible-ec2/tree/aap-2.2*<br>

*Red Hat Satellite EC2 AMI build automation:*<br>
*- https://github.com/heatmiser/packer-ansible-ec2/tree/satellite-6.11*<br>

## Use Ansible to Provision RHEL SOE on AWS

*The below instructions are steps that originated from:*<br>
*- https://github.com/ansible/workshops/tree/devel/provisioner*<br>

**Requirements:**
- System with git, python3(3.9.x), expect(unbuffer utility)
- Active public domain name (from AWS or other DNS registrar)
- AWS Account
- DNS setup (Hosted Zone) via AWS Route53

**Setup DNS:** 
- You will need to setup DNS via AWS route53
- Use AWS Public Hosted zone
- Example: subdomain.redhatpartnertech.net
- Reference: https://github.com/ansible/workshops/tree/devel/provisioner#dns

**Steps:**
 
OS Config -- software tools install & config (including Ansible)

> **NOTE**: The following instructions assume a RHEL family variant, however, all commands (other than package installation commands) *should* work on other Linux variants and Macs, assuming the same tools outlined as installed via dnf/yum are provisioned utilizing installation tools native to those operating systems, ie apt-get/Ubuntu, Homebrew/Mac, etc.

- use yum for RHEL7/CentOS7, dnf for RHEL8-9/Fedora
```
$ sudo dnf install vim git python39 expect
```
**RHEL family --> Set default python version via alternatives for provisioner to utilize correct python runtime**
- "alternatives" utility used when several programs fulfilling the same or similar functions are installed on a single system at the same time, in this case, `python`
```
$ which python
/usr/bin/which: no python in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
$ alternatives --list | grep -i python
python              	auto  	/usr/libexec/no-python
python3             	auto  	/usr/bin/python3.9
$ alternatives --set python /usr/bin/python3
$ alternatives --list | grep -i python
python              	manual	/usr/bin/python3
python3             	auto  	/usr/bin/python3.9
$ python --version
Python 3.9.6
```

**AWS Keys/Credentials - configured via credentials file *or* environment variables**
- [Walkthrough Steps](https://github.com/ansible/workshops/blob/devel/docs/aws-directions/AWSHELP.md)
- Reference: https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html

**via credentials file**
```
$ cd ~
$ mkdir .aws
$ cd .aws
$ vim credentials 
$ cat credentials
[default]
aws_access_key_id = AKIAIDA7I6XT....
aws_secret_access_key = GP7nC7Oq+tP8akFWqO....
$
```

- or if you don't want keys/creds in a file...

**via environment variables**
```
$ export AWS_ACCESS_KEY_ID="AKIAIDA7I6XT...."
$ export AWS_SECRET_ACCESS_KEY="GP7nC7Oq+tP8akFWqO...."
$
```

Create directory named "soe01" which will hold the python virtual env, in addition to log and var directories

```
$ cd ~
$ export PYVENV_PROJDIR="soe01"
$ mkdir -p ~/$PYVENV_PROJDIR/deploy_logs
$ mkdir ~/$PYVENV_PROJDIR/deploy_vars
```

Upgrade user python libraries pip and setuptools, create python virtual env, and deploy pip libraries to project python virtual env

```
$ python3 -m pip install --user --upgrade pip setuptools
$ python3 -m venv $PYVENV_PROJDIR
$ source ~/$PYVENV_PROJDIR/bin/activate
(soe01) $ python3 -m pip install --upgrade pip setuptools
(soe01) $ python3 -m pip install wheel
(soe01) $ python3 -m pip install ansible-core==2.11.7 \
    paramiko==2.8.1 \
    requests==2.27.1 \
    yamllint==1.26.3 \
    boto3==1.20.49 \
    boto==2.49.0 \
    awscli==1.22.49 \
    netaddr==0.8.0 \
    passlib==1.7.4 \
    tox==3.22.0 \
    pywinrm==0.4.2 \
    requests-credssp==1.2.0
```

 clone provisioner

```
(soe01) $ mkdir -p ~/github/project
(soe01) $ cd ~/github/project
(soe01) $ git clone https://github.com/heatmiser/rhel-soe-firstlight.git
(soe01) $ vim ~/$PYVENV_PROJDIR/deploy_vars/soe01_vars.yml
```
Please modify the following variables below to match your unique configuration.
- ec2_region: (a single ec2 region, ex. us-east-1; us-east-2)
- ec2_name_prefix: (a single unique name for your workshop, ex. soe01)
- provision_mode: `workshop` or `playground` (`workshop` completes Satellite IAC roll out; products, repos, CV, LE, Act Keys, manifest publishing, etc. takes longer to deploy, however, saves major time for workshop attendees because it's already completed prior to beginning exercises)
- offline_token: token to authenticate calls to Red Hat's APIs, generate token at https://access.redhat.com/management/api
- redhat_username / redhat_password: Red Hat account name and password, used for authorized access to registry.redhat.io
- admin_password: (a strong password 15+ characters, ex. Ansible=2021!!!)
- workshop_dns_zone: (setup in your route53 configuration, ex. subdom.redhatpartnertech.net)


All remaining variables LEAVE AS IS!!!
```
---
# region where the nodes will live
ec2_region: us-east-1

# name prefix for all the VMs
ec2_name_prefix: soe01

# creates student_total of workbenches for the workshop
student_total: 1

# Set the right workshop type, like network, rhel or f5...or smart_mgmt :)
#workshop_type: rhel
workshop_type: smart_mgmt
#workshop_type: windows

# workshop or playground
provision_mode: workshop

# Generate offline token to authenticate the calls to Red Hat's APIs
# 
offline_token: "eyJhbGci...9Kx7DU"

# Required for podman authentication to registry.redhat.io
redhat_username: myredhatacct@redhat.com
redhat_password: Xtr4Sup3rS3cr3tP4ssw0rd?!

#####OPTIONAL VARIABLES

# turn DNS on for control nodes, and set to type in valid_dns_type
dns_type: aws

# Sets the Route53 DNS zone to use for Amazon Web Services
workshop_dns_zone: subdom.redhatpartnertech.net

# password for Ansible control node
admin_password: St4nd4rdiz3!!!

# install control node
controllerinstall: true

# IBM Community Grid - defaults to true if you don't tell the provisioner
ibm_community_grid: false

# select rhel7 or rhel8 client nodes
rhel: rhel7
# choice of centos7 nodes
# refer to https://wiki.centos.org/Cloud/AWS for available minor releases
# select centos7 client node version
centos7: centos78

# deploy HA Ansible Tower cluster
#create_cluster: false

# Install automation hub node?
automation_hub: false

# this will install VS Code web on all control nodes
code_server: true

# this will change the location of lab guide
# ansible_workshops_url: https://github.com/ansible/workshops
# ansible_workshops_version: smart_mgmt

# Enable AWS IAM integration with control node and workshop nodes
tower_node_aws_api_access: true

# default vars for ec2 AMIs (ec2_info) are located in:
# provisioner/roles/manage_ec2_instances/defaults/main/main.yml
# select ec2_info AMI vars can be overwritten via ec2_xtra vars, e.g.:
#ec2_xtra:
#  satellite:
#    owners: 012345678910
#    filter: Satellite*
#    username: ec2-user
#    os_type: linux
#    size: r5a.xlarge
#    disk_volume_type: gp3
#    disk_space: 500
#    disk_iops: 3000
#    disk_throughput: 125
#    architecture: x86_64
```

**Manifest**
- Login to https://access.redhat.com --> Subscriptions --> Subscription Allocations, then [New Subscription Allocation]
- Name it, then select type: Satellite 6.11, then click [Create]
- Once created, select the Subscriptions tab, then click [Add Subscriptions] to add # of Red Hat Ansible Automation subs
- Click [Export Manifest] to download .zip file
- Move zip file to "workshops/provisioner" folder and rename<br>
`$ mv ~/manifest_sm-mgmt-wkshop_20210128T182529Z.zip ~/github/ansible/workshops/provisioner/manifest.zip`

**Test AWS Credentials**
```
(soe01) $ aws sts get-caller-identity
```
or
```
(soe01) $ aws ec2 describe-regions --region us-east-1
```

**Build Ansible workshops provisioner collection**
```
(soe01) $ cd ~/github/project/rhel-soe-firstlight
(soe01) $ git checkout main
(soe01) $ export ANSIBLE_CONFIG=provisioner/ansible.cfg
(soe01) $ ansible-galaxy install --force -r collections/requirements.yml
(soe01) $ ansible-galaxy collection build --verbose --output-path build/
(soe01) $ ansible-galaxy collection install --verbose build/*.tar.gz
```

**Finally, set some related environment variables and run the provisioner via ansible-playbook**
```
(soe01) $ export AWS_MAX_ATTEMPTS=10
(soe01) $ export AWS_RETRY_MODE=standard
```
> **NOTE**: Utilizing `unbuffer` utility to handle `ansible-playbook` is not required, however, provides a convenient method to monitor console while simultaneously writing to log file
```
(soe01) $ unbuffer ansible-playbook ./provisioner/provision_soe.yml -e @~/$PYVENV_PROJDIR/deploy_vars/soe01_vars.yml | tee ~/$PYVENV_PROJDIR/deploy_logs/soe-deploy-$(date +%Y-%m-%d.%H%M).log
```

If `unbuffer` is utilized to handle the ansible-playbook command, the log output can be cleaned up post deployment with the following command:
```
sed -i "s,\x1B\[[0-9;]*[a-zA-Z],,g" ~/$PYVENV_PROJDIR/deploy_vars/soe01-deploy-<date_timestamp>.log
```

Unfamiliar with Python virtual environments (venv)?  Remember, the Python venv was "activated" earlier in the process via this command:
* `$ source ~/$PYVENV_PROJDIR/bin/activate`

"Step out" of the venv by deactivating it:
```
(soe01) $ deactivate
$
```

**Teardown**
> **NOTE**: If you exited the Python venv for other tasks and need to return later to teardown the lab environment, activate the project venv and proceed.
```
$ export PYVENV_PROJDIR="soe01"
$ source ~/$PYVENV_PROJDIR/bin/activate
(soe01) $ ~/github/project/rhel-soe-firstlight
(soe01) $ export ANSIBLE_CONFIG=provisioner/ansible.cfg
(soe01) $ export AWS_MAX_ATTEMPTS=10
(soe01) $ export AWS_RETRY_MODE=standard
(soe01) $ unbuffer ansible-playbook teardown_soe.yml -e @~/$PYVENV_PROJDIR/deploy_vars/soe01_vars.yml -e debug_teardown=true | tee ~/$PYVENV_PROJDIR/deploy_logs/soe01-teardown-$(date +%Y-%m-%d.%H%M).log
(soe01) $ deactivate
$
```

## Self Paced Exercises

- [Ansible Automation Platform Self-Paced Labs
](https://www.redhat.com/en/engage/redhat-ansible-automation-202108061218) - These interactive learning scenarios provide you with a pre-configured Ansible Automation Platform environment to experiment, learn, and see how the platform can help you solve real-world problems. The environment runs entirely in your browser, enabling you to learn more about Red Hat automation technology at your pace when convenient.

## Product Demos

- [Demos](https://github.com/ansible/product-demos) - These demos are intended for effectively demonstrating Ansible capabilities with prescriptive guides on the Ansible Automation Workshop infrastructure.

## Additional Content

- [Get a Trial Subscription for Red Hat Ansible Automation Platform](http://red.ht/try_ansible)
- [Ansible Blog - The Inside Playbook](https://www.ansible.com/blog)
- [Ansible YouTube](https://youtube.com/ansibleautomation)
- [Ansible Getting Started Guide](https://docs.ansible.com/ansible/latest/user_guide/index.html#get)
- [Ansible Network Automation - Getting Started](https://docs.ansible.com/ansible/latest/network/getting_started/index.html)
- [Red Hat Training and Certification for Red Hat Ansible Automation Platform](https://red.ht/aap_training)

## E-Books

- [Red Hat Ansible Automation Platform: A beginner’s guide](https://www.redhat.com/en/engage/redhat-ansible-automation-20220412)
- [The Automated Enterprise](https://www.redhat.com/en/engage/automated-enterprise-ebook-20171107?intcmp=7013a000002DXg8AAG)

### E-Books for Ansible Security Automation

  - [Simplify your security operations center](https://www.redhat.com/en/resources/security-automation-ebook?extIdCarryOver=true&sc_cid=7013a000002gyQ2AAI)

---
![Red Hat Ansible Automation](https://github.com/ansible/workshops/raw/devel/images/rh-ansible-automation-platform.png)