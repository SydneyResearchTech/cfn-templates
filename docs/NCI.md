# National Computational Infrastructure (NCI)

Doc path: cfn-templates/docs/NCI.md

Deploy a single compute node on the projects default network.

```bash
CFN="$HOME/cfn-templates"
KEY_NAME=$(openstack keypair list --format=json |jq -r '.[0].Name')
PROJECT_NAME=$(openstack project list --format=json |jq -r '.[0].Name')
NETWORK=$PROJECT_NAME
SECURITY_GROUP="$PROJECT_NAME ssh"

openstack stack create -t $CFN/server.hot.yaml \
--environment $CFN/env-nci.hot.yaml \
--parameter key_name=$KEY_NAME \
--parameter os_project=$PROJECT_NAME \
--parameter network=$NETWORK \
--parameter security_groups='["'$SECURITY_GROUP'"]' \
<stack_name_here>
```
