# EFS EC2 Instance

* Create EFS access point
* IAM instance profile
  * Detach existing profile
  * Create
* Create and attach instance group via instance profile
* Create EFS
* Create EFS AP
* Put file system policy

```bash
# aws efs describe-file-systems
# aws ec2 describe-instances
EFS_ID='fs-019b87186b4284827'
EC2_INSTANCE_ID='i-0f610a619c61a5670'
#EC2_INSTANCE_IDs=()

TAGS='[
{"Key": "sydney.edu.au/resTek/createdBy", "Value": "dean.taylor@sydney.edu.au"},
{"Key": "sydney.edu.au/resTek/name", "Value": "cryosparc"},
{"Key": "sydney.edu.au/resTek/environment", "Value": "development"},
]'

for EC2_INSTANCE_ID in "${EC2_INSTANCE_IDs[@]}"; do

# Create EFS Access point
aws efs create-access-point \
--file-system-id ${EFS_ID} \
--root-directory "Path=/${EC2_INSTANCE_ID}, CreationInfo={OwnerUid=0,OwnerGid=0,Permissions=0755}" \
--tags "'${TAGS}'"

done

aws iam create-instance-profile \
--instance-profile-name \
--profile

ASSOCIATION_ID=$(aws ec2 describe-iam-instance-profile-associations \
--filters 'Name=instance-id,Values=i-0048dfa08dcc0610f' 'Name=state,Values=associated' \
--query 'IamInstanceProfileAssociations[0].AssociationId' --output text)

aws ec2 replace-iam-instance-profile-association \
--association-id $ASSOCIATION_ID \
--iam-instance-profile Name=restek-vrd-an-zhao-InstanceProfile-97ws2gYV1IxK


# Create EFS access policy
# https://docs.aws.amazon.com/efs/latest/ug/iam-access-control-nfs-efs.html
aws efs put-file-system-policy \
--file-system-id ${EFS_ID} \
--policy '{
  "Id": "1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["elasticfilesystem:Client*"],
      "Principal": {"AWS": "${ROLE_ARN}"},
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "true",
          "elasticfilesystem:AccessedViaMountTarget": "true"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": ["elasticfilesystem:Client*"],
      "Principal": {"AWS": "${ROLE_ARN}"},
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "true",
          "StringEquals": {
            "elasticfilesystem:AccessPointArn": "${EFS_AP_ARN}"
          }
        }
      }
    }
  ]
}'

% aws efs create-access-point \
--file-system-id ${EFS_ID} \
--root-directory "Path=/${EC2_INSTANCE_ID}, CreationInfo={OwnerUid=0,OwnerGid=0,Permissions=0755}"
{
    "ClientToken": "aff7ba34-c800-4a6f-bf4d-9e1e3010bb2a",
    "Tags": [],
    "AccessPointId": "fsap-00b893a01a4f724ff",
    "AccessPointArn": "arn:aws:elasticfilesystem:ap-southeast-2:381427642830:access-point/fsap-00b893a01a4f724ff",
    "FileSystemId": "fs-019b87186b4284827",
    "RootDirectory": {
        "Path": "/i-0f610a619c61a5670",
        "CreationInfo": {
            "OwnerUid": 0,
            "OwnerGid": 0,
            "Permissions": "0755"
        }
    },
    "OwnerId": "381427642830",
    "LifeCycleState": "creating"
}
```
