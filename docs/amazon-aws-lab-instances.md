# Amazon AWS Lab management

```bash
STACK_NAME='restek-vrd-lab-v1-0-0'
STACK_NAME='restek-vrd-lab-ccf512e294dd'

ASG_NAME=$( \
aws autoscaling describe-auto-scaling-groups --filters Name=tag:aws:cloudformation:stack-name,Values=$STACK_NAME \
--query 'AutoScalingGroups[].AutoScalingGroupName' --output text)

aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names $ASG_NAME \
|jq -r '.AutoScalingGroups[].Instances[].InstanceId' \
|xargs aws ec2 describe-instances --instance-ids \
|jq -r '.Reservations[].Instances[]|"\(.PublicDnsName) \(.PublicIpAddress) \(.PrivateDnsName) \(.PrivateIpAddress)"'

aws ec2 describe-instances --filters "Name=tag:aws:cloudformation:stack-name,Values=${STACK_NAME}" \
|jq -r '.Reservations[].Instances[]|"\(.PublicDnsName) \(Tags|filter(Key==sydney.edu.au/resTek/defaultUserPwd).Value)"'

aws ec2 describe-instances --filters "Name=tag:aws:cloudformation:stack-name,Values=${STACK_NAME}" \
|jq -r '.Reservations[].Instances[]|"\(.PublicDnsName)"'
```

```bash
ansible stack_${STACK_NAME//-/_} -m ping
```

## VRD Lab setup instructions

```bash
ansible stack_${STACK_NAME//-/_} -a 'cloud-init status --wait'

ansible-playbook -e "target=stack_${STACK_NAME//-/_}" restek.core.apt_upgrade_reboot

ansible-playbook -e "target=stack_${STACK_NAME//-/_}" restek.core.nvidia_toolkit_install

ansible-playbook -e "target=stack_${STACK_NAME//-/_}" restek.core.microk8s_dev_env
```

### Add SSH public keys as required. Ensure you have all instructors and helpers at a minimum.

```bash
ansible stack_${STACK_NAME//-/_} -m ansible.posix.authorized_key -a '{
"key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmAqXRkkXxWzixAa7c7VFXqWim0dUUjZWaLtEl8MR8gEZU1hFjJaArQxw4Dgq6zg1GY1ZvSgTga9JsNg08jQWpvxZzd3lrN7gqc3jYH0uPWR2qD03kdGzFqMUq6vzMoMDeI+y9H36LPEt5HAws2eMOVw5W/0nIbxBVqRG9uBLdJNf1zPqY6H3XGSMEiC1vwqnq+OhrA4dARHEJ2U0meHyA+cRVLzaG549aPkmdnUJ2tRcq4GxEltXt0nZftOVKLknQ0SuE1x01BrPZsp4W5+/3ipVkDcWWcytiyIR2PgtbFma0bdFFO+KL05SHJ05x06F8fDiNZkdEuzx1UofTOOxn thomas.close@sydney.edu.au",
"state": "present",
"user": "ubuntu"
}'
```

### Downgrade / fix versions of any resources as required

```bash
ansible stack_${STACK_NAME//-/_} -m ansible.builtin.apt -b -a '{
"allow_downgrade": true,
"name": "docker-ce=5:24.0.9-1~ubuntu.22.04~jammy",
"state": "present",
"update_cache": true
}'

ansible stack_${STACK_NAME//-/_} -m ansible.builtin.dpkg_selections -b -a '{
"name": "docker-ce",
"selection": "hold"
}'
```

### Enable password access for the lab

A complex unique password will be set for each user.
This will become part of the workshop setup report generated later.

The report generated bellow will provide the DNS name and password generated for each instance.
Please note, that you will need some way to disseminate this information to workshop participants. Something
as simple as a printout on each table or slip of paper with the details at each chair.

```bash
ansible-playbook -e "target=stack_${STACK_NAME//-/_}" \
-e '{active: true}' \
restek.core.aws_sshd_pwd_enable
```

### OR set a common password for every instance

This allows the connection details to become part of the presentation.
Consider:
1. The password needs to be reasonably complex keeping in mind the entire lab is short lived.
2. The password needs to be easily typed or there is a risk of initial setup delays during the workshop.

```bash
PASSWD="SET_YOUR_OWN_OR_GENERATE_ONE_BELLOW"
PASSWD="${$(uuidgen)##*-}"

ansible-playbook -e "target=stack_${STACK_NAME//-/_}" \
-e '{active: true}' \
-e "password=${PASSWD}" \
restek.core.aws_sshd_pwd_enable
```

### Generate access report

```bash
aws ec2 describe-instances --filters "Name=tag:aws:cloudformation:stack-name,Values=${STACK_NAME}" \
|jq -r '
.Reservations[].Instances[]|
"\(.PublicDnsName) \(.Tags[]|select(.Key=="sydney.edu.au/resTek/defaultUserPwd").Value)"
'
```

### Disable password access

```bash
ansible-playbook -e "target=stack_${STACK_NAME//-/_}" \
-e '{active: false}' \
restek.core.aws_sshd_pwd_enable
```
