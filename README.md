# Portworx Armory Demo

This repository is designed to be cloned per environment. We are working to automate the setup steps.

## Prerequites

Ensure that you have the following utilities installed and up to date:
- eksctl
- aws - should have access to create clusters
- armory - should be logged in
- kubectl
- storkctl
- pxctl (best to use an alias for this one)

Details for utility installation coming soon.

It is important for the installation of portworx that the IAM role assigned to the EKS cluster has permissions to add disks. Details on creating an IAM policy can be found here: https://docs.portworx.com/portworx-enterprise/install-portworx/kubernetes/aws/aws-eks

Note that I have already created this policy and assigned it to the cluster using the eksctl file

## Manual Setup

### Deploy EKS clusters

`eksctl create cluster --kubeconfig ~/git/newstack/labv2/configs/eks03 -f setup/uswest2-eks03-prod.yaml`

`eksctl create cluster --kubeconfig ~/git/newstack/labv2/configs/eks04 -f setup/uswest2-eks04-stage.yaml`

### Install the Armory Agents

NOTE: in order to avoid modifying config files, it is important that agents are installed within tenants. There is a demo1 and demo2 tenant environment in armory. The source (prod) cluster is called cluster01 and the destination (stage) cluster is called cluster02


### Generate Keys

Generate credential for the CLI by creating a new Client Credential in the Armory portal under the demo1 tenant. Create a new alias in your .bashrc file using the form:

`alias a1='armory -c ******** -s *********`

Install the agents using the demo01 tenant to the demo01 environments.

`a1 agent create --context-name ccrow@ccrow-uswest2-eks01-prod.us-west-2.eksctl.io --namespace armory-rna --name cluster01`

`a1 agent create --context-name ccrow@ccrow-uswest2-eks01-prod.us-west-2.eksctl.io --namespace armory-rna --name cluster01`

**The above doesn't seem to work, so I manually added the agents**

### Populate Armory Secrets in Github

Generate a new set of client credentials in armory and populate the client and secret IDs in the following environment variables

`CDAAS_CLIENT_ID`
`CDAAS_CLIENT_SECRET`

### Populate Github Secrets in Armory
Create a new github token that has access to repos, workflows and packages. Populate that as an armory secret called:

`Github_token`

Sorry for the errant capital letter, too late to change it now!


### Setup Github Actions

### Deploy the Armory Utilities

This can be done by modifying anything inside the utilities directory, or you can simply deploy the manifests manually as specified in the utilities.yaml deployment file in the root of the repo


### Deploy PX-operator and portworx

I have had the best result deploying the operator manually with:

`kubectl apply -f portworx/px-operator.yaml`

Be sure to run the above on both clusters.

Deploying Portworx can be done by modifying the storagecluster01.yaml (this will deploy to both clusters)

You can manually deploy by applying the appropriate manifest to the appropriate cluster.

### Change the default storage class

`kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'`

`kubectl patch storageclass px-db -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'`

### Build service accounts

Apply the serviceaccount,secret,clusterrole binding to both clusters.

`kubectl apply -f setup/svcAccount.yaml`

edit the create-migration-config-esk01.sh and create-migration-config-eks02.sh and update the server variable.

Export the configuration files:

NOTE: Be sure you are in the correct context!

`./setup/create-migration-config-esk03.sh > ~/eks03.config`

`./setup/create-migration-config-esk03.sh > ~/eks03.config`

### Pair the cluster

Run the following (substituting your access and secret key):

`storkctl create clusterpair stage \
--dest-kube-file ~/eks03.config \
--src-kube-file ~/eks04.config \
--dest-ep <PortworxAPI>:9001 \
--src-ep <PortworxAPI>:9001 \
--namespace portworx \
--provider s3 \
--s3-endpoint s3.amazonaws.com \
--s3-access-key **************** \
--s3-secret-key **************** \
--s3-region us-west-2 \
--unidirectional`

### Set up the initial PXBBQ application

Run the following against cluster01

`kubectl apply -f pxbbq-demo/namespace.yml`
`kubectl apply -f pxbbq-demo/mongodb-pvc.yaml`
`kubectl apply -f pxbbq-demo/mongodb.yaml`
`kubectl apply -f pxbbq-demo/pxbbq.yaml`

I then applied the migration script to get the initial pvc over:

`kubectl apply -f pxbbq-demo/migrate.yaml`

Next, ensure that PXBBQ is running by connecting to the service IP

### Setting up the integration tests

The integration tests contained in the pxbbq.yaml deployment file reference the github repository. Update it (should be line 126)

### Set up prometheus integration:

Config in armory:

`prometheus config:
base url: http://prometheus-server.prometheus:9090/
remote network agent: the one for the prod account
name: prod-prometheus
type: prometheus
authentication type: none`

## Running the Demo

### Prerequisites

- Ensure you can connect to both clusters (Chris should have generated service accounts and configurations)
- Bring up the Armory Portworx Deck
- Log in to Github.com/ccrow42/armory-px-demo01
- Log in to Armory Cloud console at https://console.cloud.armory.io/

### Running the Demo

The demo is fairly simple to run.

- First, run through the slide deck to talk about what we are doing
- Upgrade Mongo to 6.0.2
- Explain why it failed
- Upgrade Mongo to 6.0.3
- Explain the Process by showing the pxbbq.yaml Armory deployment file


### Todo

- Add a title slide (and booth locations)
- Get QR Code
- Fix Prometheus
- Add labels to deck
- Schedule one more check-in
- Post recording