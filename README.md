# aci-demos
Demos with ACI


The imagesRecognition folder has a Dockerfile to deploy python code that does facial recognition on multiple pictures and counts up the amount of throughput for each node - meant to be used across a kubernetes cluster

The other folder has a Dockerfile to deploy an html UI that has a dashboard of the metrics from the cluster's throughput of the above Dockerfile.

Contact ria.bhatia@microsoft.com if you need help!

prereq: have helm installed

steps to deploy dis demo

create resource group
`az group create --name myResourceGroup --location westeurope`

deploy acs cluster
`az acs create --orchestrator-type kubernetes --resource-group myResourceGroup --name myK8sCluster --generate-ssh-keys`

get creds
`az acs kubernetes get-credentials --resource-group=myResourceGroup --name=myK8sCluster`

make sure you're connected
`kubectl get nodes`

go into the charts folder in this repo and deploy the webserver helm chart first
`helm inspect values charts/webserver > customWeb.yaml`
`helm install -n webserver -f customWeb.yaml charts/webserver`

wait to get an external ip for the webserver
`kubectl get svc -w aci-demo-aci-demo-webserver`

once you get the external ip copy the ip and paste it into the aci-demo and aci-ui values.yaml files
<webserver ip>

deploy the aci-ui chart
`helm inspect values charts/aci-ui > customUI.yaml`
`helm install -n aci-ui -f customUI.yaml charts/aci-ui`

now let's deploy aci
use the same rg for aci or a new one
`az group create -n aci-test -l westus`

grab your sub id
`az account list -o table`

`az ad sp create-for-rbac --role=Contributor --scopes /subscriptions/<subscriptionId>/`

clone the aci-connector repo
`git clone https://github.com/Azure/aci-connector-k8s.git`

edit your yaml file from within the aci-connector folder
`vim examples/aci-connector.yaml`

deploy the aci-connector
`kubectl create -f examples/aci-connector.yaml`

deploy the aci-demo chart
`helm inspect values charts/aci-demo > custom.yaml`
`helm install -n aci-demo -f custom.yaml charts/aci-demo`

checkout https://<aci-ui externalIP>:80

cleanup
helm del --purge webserver
helm del --purge aci-demo
helm del --purge aci-ui
