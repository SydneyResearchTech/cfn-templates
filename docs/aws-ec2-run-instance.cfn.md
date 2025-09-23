# aws ec2 run-instance

## EKS cluster resources including primary EFS access

```bash
EKS_CLUSTER_NAME=<the EKS cluster name>
EKS_CLUSTER_NAME=ais-uat
KEY_NAME=<your ssh key pair name>
KEY_NAME=dean.taylor@sydney.edu.au

SECURITY_GROUP_ID=$( \
aws ec2 describe-security-groups \
--filters \
"Name=group-name,Values=eks-cluster-sg-$EKS_CLUSTER_NAME-*" \
"Name=tag:aws:eks:cluster-name,Values=$EKS_CLUSTER_NAME" \
"Name=tag:kubernetes.io/cluster/$EKS_CLUSTER_NAME,Values=owned" \
--query \
'SecurityGroups[0].GroupId' \
--no-paginate --output text \
)

# EFS file system ID for the EKS cluster
EFS_ID=$( \
aws cloudformation describe-stacks --stack-name eks-${EKS_CLUSTER_NAME}-efs-csi \
--query 'Stacks[0].Outputs[?OutputKey==`FileSystemId`].OutputValue' \
--no-paginate --output text \
)

# EFS network details
read -r SUBNET_ID AZ VPC_ID <<<$( \
aws efs describe-mount-targets --file-system-id=$EFS_ID \
--query 'MountTargets[0].[SubnetId,AvailabilityZoneName,VpcId]' \
--no-paginate --output text \
)

### OPTIONAL if you require specific image characteristics
INSTANCE_TYPE=$( \
aws ec2 describe-instance-types \
--filters \
'Name=bare-metal,Values=false' \
'Name=current-generation,Values=true' \
'Name=hypervisor,Values=nitro' \
'Name=instance-type,Values=m*' \
'Name=processor-info.supported-architecture,Values=x86_64' \
'Name=supported-root-device-type,Values=ebs' \
'Name=vcpu-info.default-vcpus,Values=4' \
--query \
'InstanceTypes[0].InstanceType' \
--no-paginate --output text \
)

aws cloudformation create-stack --stack-name=$STACK_NAME \
--parameters \
"ParameterKey=AvailabilityZone,ParameterValue=$AZ" \
"ParameterKey=EfsFileSystemId,ParameterValue=$EFS_ID" \
"ParameterKey=KeyName,ParameterValue=$KEY_NAME" \
"ParameterKey=SecurityGroupId,ParameterValue=$SECURITY_GROUP_ID" \
"ParameterKey=SubnetId,ParameterValue=$SUBNET_ID" \
"ParameterKey=VpcId,ParameterValue=$VPC_ID" \
--template-body=file://./aws-ec2-run-instance.cfn.yml
```
