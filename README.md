# jfrog-platform-quick-install-k8s


```
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

```
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

```
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


```
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

```
export SERVICE_IP=$(kubectl get svc --namespace artifactory-ha artifactory-ha-nginx -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo http://$SERVICE_IP/
```
   

```
export SERVICE_IP=$(kubectl get svc --namespace artifactory-ha artifactory-ha-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo http://$SERVICE_IP/
```
