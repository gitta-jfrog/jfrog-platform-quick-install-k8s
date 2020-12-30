# jfrog-platform-quick-install-k8s


## Install Artifactory-HA
### Artifactory Licenses
Prepare a text file with the license(s) written in it. If writing multiple licenses (must be in the same file), it's important to put **two new lines between each license block**!
```bash
# Create the Kubernetes secret (assuming the local license file is 'art.lic')
kubectl create secret generic artifactory-cluster-license --from-file=./art.lic
```
### Install Artifactory-HA
```bash
helm upgrade --install artifactory-ha --create-namespace --namespace artifactory-ha center/jfrog/artifactory-ha \
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
--set postgresql.persistence.size=20Gi \
--set databaseUpgradeReady=true \
--set unifiedUpgradeAllowed=true
```

## Install Pipelines
```bash
helm upgrade --install pipelines --namespace pipelines center/jfrog/pipelines \
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
--set rabbitmq.externalUrl="amqps://pipelines-rabbitmq.pipelines.svc.cluster.local"
```

### Create kubeconfig and add it in AdminPanel->Pipelines->Kubernetes Intergration 
Extract Token

```yaml
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep nodepool-sa- | awk '{print $1}')| grep token
```

Copy ~/.kube/config and edit (changes in bold)

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01URXpNREUyTURZeE5Wb1hEVE13TVRFeU9ERTJNRFl4TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTkZ6CnBVTklCNFlzTURjZjdPR3lIRmM1UHRyZDgwcTFLeGIyVTd0TWtDMFRVRjdjVEUvbzgxdldaRXU3cHl1cE1KUkwKM2RMWDVnNnVGZFBGRnUxRWlvSmFLTTVMa0JsdjlwRnlvT050QWxCVGJ4SlhPYnN4NVIrWXEwZ29UdkFjVzV3Qgp1aXQrTHNMUmJmRlJYSjVKRFJURC9UNkFOU3NqWUlOcHBXS1FISjRkUWpjNFlZZzVBZjBGSVhra0h0MHJUcm0vCnlPWFU5SWNYUk5XeFpLM2JIVUVwd1RnVms1c2hWK3Y4UDlVR1Y4RzFwMUgxSzBFOVhZdFZSeGI5ZWRnaWR3ZzkKMjNvTEZqaENkd21YTm1qU0RMSTdDR3BrcllMcC9PN1loYTR5bHdMemRZaTh1akR1cjE2K25nc0ozWit5SllnagpCMHVWOXRIN2dPZGowVUJ6ellVQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHNmdFYldpS1BKN2lXY1FOYkt4aDIwNmlKNWIKWlpEUGlEZ2lxY3dBemJ3STJXbVQ3dk9hRmVRcXgyanBOdFdDZTVSUHZ5djl3Z3lQNFU5eUNvbm5Td0NUWE5tbwpMcG1Ua3d3YjNVaFZBbkFraVVoWk82bGpwbnFjZnhuQ3R3TzFORE91R0t0QmFDRi9TT2dOVXN4N2didGVRT01HCkM5UGZ1OStmeEtzSVJsUWNtWEsvSUxoV3N4alB4Tk1oVkpLZmVUSkN0cmpWVy9adTNrdWdSMmNMemkrci91WVcKUnlLUkFMVVA4M3JOeU5qVnlyZHBFdURyTjFCTno3TE1zdG1vbFVhVjZ3SnlZcVMraklKNzQyZWIvY01ueW84YQpmQlJnaUw4OWZ1VVQxQms0elBGNjFyQUw5clBDMVh1YjRvQjlFQWhkSXZMcXoyY2lQSG42SXpIRXJNdz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://89E82F9E808C64CD221CCE3211ACDF34.gr7.us-west-2.eks.amazonaws.com
  name: yardeng-cluster2.us-west-2.eksctl.io
contexts:
- context:
    cluster: yardeng-cluster2.us-west-2.eksctl.io
    **user: nodepool-sa**
  name: yardeng@yardeng-cluster2.us-west-2.eksctl.io
current-context: yardeng@yardeng-cluster2.us-west-2.eksctl.io
kind: Config
preferences: {}
users:
**- name: nodepool-sa
  user:
    token:** **TOKEN**
  
```

## Export JFrog Platform URL
```bash
export SERVICE_IP=$(kubectl get svc --namespace artifactory-ha artifactory-ha-nginx -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo http://$SERVICE_IP/
```
OR
```bash
export SERVICE_IP=$(kubectl get svc --namespace artifactory-ha artifactory-ha-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo http://$SERVICE_IP/
```
