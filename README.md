# cfn-templates
Amazon CloudFormation templates

## CVL desktop

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
