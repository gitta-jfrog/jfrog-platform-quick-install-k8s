# JFrog Platform - Quick Install - K8S (unofficial)

## Prerequisites Details

* Google Cloud SDK - https://cloud.google.com/sdk/docs/install
* Kubectl https://kubernetes.io/docs/tasks/tools/install-kubectl/
* Recommand - K9s - https://k9scli.io/topics/install/
* Artifactory HA licenses 

## Create GKE Cluster
```bash
PROJECT=<project> 
CLUSTER_NAME=<cluster-name> ##limited to 16 charts
ZONE=<zone>

gcloud --project ${PROJECT} container clusters create ${CLUSTER_NAME}  \
    --enable-ip-alias \
    --no-enable-autoupgrade \
    --network default \
    --subnetwork default \
    --num-nodes 2 \
    --zone ${ZONE} \
    --machine-type e2-standard-4 \
    --disk-size 50 \
    --cluster-version 1.17.14-gke.400
```
## Install Artifactory
### Artifactory Licenses
Prepare a text file with the license(s) written in it. If writing multiple licenses (must be in the same file), it's important to put **two new lines between each license block**!
```bash
# Create the Kubernetes secret (assuming the local license file is 'art.lic')
kubectl create namespace artifactory-ha
kubectl -n artifactory-ha create secret generic artifactory-cluster-license --from-file=./art.lic
```
### Install Artifactory-HA
```bash
helm upgrade --install artifactory-ha --namespace artifactory-ha center/jfrog/artifactory-ha \
--set artifactory.joinKey="edf2019bc0d330a3f058b4662b16eb7a" \
--set artifactory.masterKey="c601841ee4a874161d9fc596a6a1974c99970771c6139eae20898eed1c61ace3" \
--set artifactory.node.replicaCount=1 \
--set artifactory.persistence.size=20Gi \
--set postgresql.postgresqlUsername="artifactory" \
--set postgresql.postgresqlPassword="1WcseEOS4s" \
--set postgresql.image.tag="12.3.0-debian-10-r71" \
--set postgresql.persistence.size=20Gi \
--set databaseUpgradeReady=true \
--set unifiedUpgradeAllowed=true \
--set artifactory.license.secret=artifactory-cluster-license,artifactory.license.dataKey=art.lic
```

## Install Mission-Control
```bash
helm upgrade --install mission-control --create-namespace --namespace mission-control center/jfrog/mission-control \
--set missionControl.joinKey="edf2019bc0d330a3f058b4662b16eb7a" \
--set missionControl.masterKey="c601841ee4a874161d9fc596a6a1974c99970771c6139eae20898eed1c61ace3" \
--set missionControl.jfrogUrl="http://artifactory-ha-nginx.artifactory-ha.svc.cluster.local" \
--set missionControl.persistence.size=20Gi \
--set elasticsearch.persistence.size=20Gi \
--set postgresql.postgresqlPassword="1WcseEOS4s" \
--set postgresql.image.tag="12.3.0-debian-10-r71" \
--set postgresql.persistence.size=20Gi \
--set databaseUpgradeReady=true \
--set unifiedUpgradeAllowed=true
```

## Install Distribution
```bash
helm upgrade --install distribution --create-namespace --namespace distribution center/jfrog/distribution \
--set distribution.joinKey="edf2019bc0d330a3f058b4662b16eb7a" \
--set distribution.masterKey="c601841ee4a874161d9fc596a6a1974c99970771c6139eae20898eed1c61ace3" \
--set distribution.jfrogUrl="http://artifactory-ha-nginx.artifactory-ha.svc.cluster.local" \
--set distribution.persistence.size=20Gi \
--set redis.password="1WcseEOS4s" \
--set postgresql.postgresqlPassword="1WcseEOS4s" \
--set postgresql.image.tag="12.3.0-debian-10-r71" \
--set postgresql.persistence.size=20Gi \
--set databaseUpgradeReady=true \
--set unifiedUpgradeAllowed=true 
```

## Install Xray
```bash
helm upgrade --install xray --create-namespace --namespace xray center/jfrog/xray \
--set xray.joinKey="edf2019bc0d330a3f058b4662b16eb7a" \
--set xray.masterKey="c601841ee4a874161d9fc596a6a1974c99970771c6139eae20898eed1c61ace3" \
--set xray.jfrogUrl="http://artifactory-ha-nginx.artifactory-ha.svc.cluster.local" \
--set common.persistence.size=20Gi \
--set rabbitmq.rabbitmqPassword="1WcseEOS4s" \
--set postgresql.postgresqlPassword="1WcseEOS4s" \
--set postgresql.image.tag="12.3.0-debian-10-r71" \
--set postgresql.persistence.size=50Gi \
--set databaseUpgradeReady=true \
--set unifiedUpgradeAllowed=true
```

## Install Artifactory Edge node
```yaml
helm upgrade --install artifactory-edge --create-namespace --namespace artifactory-edge center/jfrog/artifactory \
--set artifactory.joinKey="edf2019bc0d330a3f058b4662b16eb7a" \
--set artifactory.masterKey="c601841ee4a874161d9fc596a6a1974c99970771c6139eae20898eed1c61ace3" \
--set artifactory.persistence.size=20Gi \
--set postgresql.postgresqlPassword="1WcseEOS4s" \
--set databaseUpgradeReady=true \
--set unifiedUpgradeAllowed=true \
--set postgresql.image.tag="12.3.0-debian-10-r71"
```

## Install Pipelines
```bash
helm upgrade --install pipelines --create-namespace --namespace pipelines center/jfrog/pipelines \
--set pipelines.jfrogUrl="http://artifactory-ha-nginx.artifactory-ha.svc.cluster.local" \
--set pipelines.jfrogUrlUI="http://artifactory-ha-nginx.artifactory-ha.svc.cluster.local" \
--set pipelines.joinKey="edf2019bc0d330a3f058b4662b16eb7a" \
--set pipelines.masterKey="c601841ee4a874161d9fc596a6a1974c99970771c6139eae20898eed1c61ace3" \
--set pipelines.msg.uiUserPassword="somepassword" \
--set rabbitmq.auth.password="password" \
--set existingSecret="" \
--set postgresql.postgresqlPassword="1WcseEOS4s" \
--set postgresql.image.tag="12.3.0-debian-10-r71" \
--set postgresql.persistence.size=20Gi \
--set databaseUpgradeReady=true \
--set unifiedUpgradeAllowed=true \
--set pipelines.api.externalUrl="http://pipelines-pipelines-api.pipelines.svc.cluster.local:30000" \
--set pipelines.www.externalUrl="http://pipelines-pipelines-www.pipelines.svc.cluster.local:30001" \
--set rabbitmq.externalUrl="amqp://pipelines-rabbitmq.pipelines.svc.cluster.local" \
--set pipelines.rbac.clusterRole="true"
```

### Create kubeconfig file and add it in AdminPanel->Pipelines->Kubernetes Intergration 


Extract Token

```bash
kubectl -n pipelines describe secret $(kubectl -n pipelines get secret | grep pipelines-token- | awk '{print $1}')| grep token
```

Copy ~/.kube/config and edit the file:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: 
    server: 
  name: 
contexts:
- context:
    cluster: 
    user: pipelines  ### change exist user to  pipelines
  name:
current-context: 
kind: Config
preferences: {}
users:  ### Delete exist section under users and add  name and Token
- name: pipelines
  user:
    token: TOKEN
  
```

## Export JFrog Platform URL
```bash
export SERVICE_IP=$(kubectl get svc --namespace artifactory-ha artifactory-ha-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo http://$SERVICE_IP/
```
OR
```bash
export SERVICE_IP=$(kubectl get svc --namespace artifactory-ha artifactory-ha-nginx -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo http://$SERVICE_IP/
```
