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

ansible stack_${STACK_NAME//-/_} -m ansible.posix.authorized_key -a '{
"key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmAqXRkkXxWzixAa7c7VFXqWim0dUUjZWaLtEl8MR8gEZU1hFjJaArQxw4Dgq6zg1GY1ZvSgTga9JsNg08jQWpvxZzd3lrN7gqc3jYH0uPWR2qD03kdGzFqMUq6vzMoMDeI+y9H36LPEt5HAws2eMOVw5W/0nIbxBVqRG9uBLdJNf1zPqY6H3XGSMEiC1vwqnq+OhrA4dARHEJ2U0meHyA+cRVLzaG549aPkmdnUJ2tRcq4GxEltXt0nZftOVKLknQ0SuE1x01BrPZsp4W5+/3ipVkDcWWcytiyIR2PgtbFma0bdFFO+KL05SHJ05x06F8fDiNZkdEuzx1UofTOOxn thomas.close@sydney.edu.au",
"state": "present",
"user": "ubuntu"
}'

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

ansible-playbook -e "target=stack_${STACK_NAME//-/_}" \
-e '{active: true}' \
-e 'password=AIStalk@NIF2024Pa$$word' \
restek.core.aws_sshd_pwd_enable

aws ec2 describe-instances --filters "Name=tag:aws:cloudformation:stack-name,Values=${STACK_NAME}" \
|jq -r '
.Reservations[].Instances[]|
"\(.PublicDnsName) \(.Tags[]|select(.Key=="sydney.edu.au/resTek/defaultUserPwd").Value)"
'
```
