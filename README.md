# cfn-templates

Amazon CloudFormation templates

## Desktop

```bash
STACK_NAME='##########'

aws cloudformation create-stack --stack-name ${STACK_NAME} \
--template-body file://./cvl-desktop-lab.cfn.yml \
--parameters file://./tmp/desktop-development.param.cfn.json
```

```bash
aws ec2 describe-subnets \
  --filters 'Name=map-public-ip-on-launch,Values=true' \
  --query 'reverse(sort_by(Subnets,&AvailableIpAddressCount))[0].{SubnetId: SubnetId, VpcId: VpcId, AvailabilityZone: AvailabilityZone}'
```

```json
[
  {"ParameterKey": "DefaultUserPassword", "ParameterValue": "##########"},
  {"ParameterKey": "ImageId", "ParameterValue": "ami-0b4387dd850b3a1ef"},
  {"ParameterKey": "KeyName", "ParameterValue": "##########"},
  {"ParameterKey": "AvailabilityZones", "ParameterValue": "##########"},
  {"ParameterKey": "SubnetId", "ParameterValue": "##########"},
  {"ParameterKey": "VpcId", "ParameterValue": "##########"}
]
```

```bash
STACK_NAME='cvl-desktop-lab'
VM_CNT=1

# Get the latest CVL desktop image available
IMAGE_ID=$(aws ec2 describe-images --owners 381427642830 \
 --filters 'Name=name,Values=ubuntu-jammy-22.04-amd64-server-cvl-desktop-*' \
 --query 'sort_by(Images,&CreationDate)[-1].{id:ImageId}' --output text)

# Create the CVL destop laboratory
# Alter any of the parameters required
aws cloudformation create-stack --stack-name $STACK_NAME \
 --template-url https://raw.githubusercontent.com/SydneyResearchTech/cfn-templates/main/cvl-desktop-lab.cfn.yml \
 --parameters <<EOT
[
 {"ParameterKey": "ASGDesiredCapacity", "ParameterValue": $VM_CNT},
 {"ParameterKey": "ImageId", "ParameterValue": "$IMAGE_ID"},
 {"ParameterKey": "KeyName", "ParameterValue": "dean.taylor@sydney.edu.au"}
]
EOT

# 
aws ec2 describe-instances --filters \
 'Name=tag:aws:cloudformation:stack-name,Values='$STACK_NAME \
 'Name=instance-state-name,Values=running' \
 --query 'sort_by(Reservations[0].Instances, &AmiLaunchIndex)[].{AmiLaunchIndex: AmiLaunchIndex, PublicDnsName: PublicDnsName}'
```
