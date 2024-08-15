# AWS CloudFormation alter an existing stack

```bash
STACK_NAME="restek-vrd-lab-v1-0-0"
STACK_NAME="restek-vrd-lab-ccf512e294dd"    # 7 Aug 2024
CAPACITY="16"

aws cloudformation get-template --stack-name $STACK_NAME |jq -r '.TemplateBody'

aws autoscaling describe-auto-scaling-groups --output=text \
--filters "Name=tag:aws:cloudformation:stack-name,Values=$STACK_NAME" \
--query 'AutoScalingGroups[].DesiredCapacity'

aws cloudformation update-stack --stack-name $STACK_NAME \
--use-previous-template \
--parameters \
ParameterKey="ASGDesiredCapacity",ParameterValue=$CAPACITY \
ParameterKey="AvailabilityZones",UsePreviousValue=true \
ParameterKey="LaunchTemplateName",UsePreviousValue=true \
ParameterKey="LaunchTemplateVersion",UsePreviousValue=true
```

```bash
aws cloudformation get-template --stack-name restek-vrd-lab-v1-0-0 |jq -r '.TemplateBody'

aws cloudformation create-change-set --stack-name restek-vrd-lab-v1-0-0 \
--change-set-name CapacityChange --use-previous-template --parameters \
ParameterKey="AvailabilityZones",UsePreviousValue=true \
ParameterKey="ASGDesiredCapacity",ParameterValue=1 \
ParameterKey="LaunchTemplateName",UsePreviousValue=true \
ParameterKey="LaunchTemplateVersion",UsePreviousValue=true

aws cloudformation list-change-sets --stack-name restek-vrd-lab-v1-0-0

aws cloudformation describe-change-sets 
```

```bash
aws service-quotas list-services

aws service-quotas
"QuotaCode": "L-CAE24619",
"QuotaName": "Running Dedicated g4dn Hosts",


aws service-quotas get-service-quota \
--service-code ec2 \
--quota-code L-DB2E81BA

aws ec2 describe-instances --filters \
'Name=instance-type,Values=g4dn.*' \
--query 'Reservations[].Instances[].CpuOptions.CoreCount'

4 vCPU x22 = 88

Current 40
Lab     88
Contant VMs = 28

aws service-quotas request-service-quota-increase \
--desired-value 116 \
--quota-code L-DB2E81BA \
--service-code ec2
```
