# php-mysql-monitoring

## Goal

## Architecture

## Steps
### Setup Kubernetes environment
### Deploy Elastic Cloud or other Elasticsearch / Kibana instance
### Deploy a service in Kubernetes (Wordpress in this demo)
### Deploy shippers (open source Elastic Beats)

Raw keystrokes
```gcloud container clusters create mysql-php-elastic –num-nodes=3 –zone=us-central1-a
mkdir wordpress
cd wordpress/
git clone https://github.com/GoogleCloudPlatform/kubernetes-engine-samples
cd kubernetes-engine-samples/wordpress-persistent-disks
gcloud compute disks create –size 200GB mysql-disk
gcloud compute disks create –size 200GB wordpress-disk
kubectl create secret generic mysql -–from-literal=password=<oneofmypasswords>
kubectl create -f mysql.yaml
kubectl get pod
kubectl create -f mysql-service.yaml
kubectl get service mysql
kubectl create -f wordpress.yaml
view wordpress.yaml
kubectl get po
kubectl get po -w
kubectl create -f wordpress-service.yaml
kubectl get svc -l app=wordpress
kubectl get svc -l app=wordpress -w
vi PASSWORD
```

And for the beats:

```cd ${HOME}
# Note: in the metricbeat file I am not using these secrets, need to update that config to use them!
kubectl create secret generic dynamic-logging \
--from-file=./ELASTIC_PASSWORD \
--from-file=./CLOUD_ID --namespace=kube-system

#same password as used for mysql in the default namespace, but in kube-system this time.  
# need to see about accessing a diff namespace.
kubectl create secret generic mysql-pass --from-file=./MYSQL_PASSWORD --namespace=kube-system

# need to enable kube-state-metrics
go get k8s.io/kube-state-metrics
cd /home/dan_roscigno/gopath/src/k8s.io/kube-state-metrics
make container
kubectl apply -f kubernetes
kubectl create -f kubernetes
kubectl get service kube-state-metrics  --namespace=kube-system

# I like to verify things, if you want to verify that kube-state-metrics is collecting and publishing metrics then:
0) Get the Cluster IP of the kube-state-metrics service: kubectl get service kube-state-metrics  --namespace=kube-system
1) Find the wordpress pod: kubectl get po
2) Run a shell within the wordpress pod: kubectl exec -ti <wordpress pod name> -- /bin/bash
3) curl from the Cluster IP of the kube-state-metrics service: curl <cluster IP>:8080/metrics
4) exit from the bash shell

mkdir elastic
cd elastic/


kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=<your Google account@gmail.com>

curl -L -O https://raw.githubusercontent.com/elastic/beats/6.2/deploy/kubernetes/metricbeat-kubernetes.yaml

# edit the yaml to set the CLOUD_ID and ELASTIC_PASSWORD and create the index template and dashboards
vi metricbeat-kubernetes.yaml

Here are my diffs:
dan_roscigno@wordpress-198401:~/elastic/php-mysql-monitoring$ diff -y --suppress-common-lines metricbeat-kubernetes.yaml working.yaml
                                                              >     setup.dashboards.enabled: true
                                                              >     setup.template.enabled: true
                                                              >
                                                              >     setup.template.settings:
                                                              >       index.number_of_shards: 1
                                                              >
      hosts: ['${ELASTICSEARCH_HOST:elasticsearch}:${ELASTICS |             #hosts: ['${ELASTICSEARCH_HOST:elasticsearch}:${E
        - name: ELASTICSEARCH_HOST                            <
          value: elasticsearch                                <
        - name: ELASTICSEARCH_PORT                            <
          value: "9200"                                       <
          value: changeme                                     |           value: DQ8rxxxxxxxxxMWsnEc
          value:                                              |           value: PHPMySQLDemo:dXMtY2VudHJhbDEuZ2NwLmNsb3VkLmV
        - name: ELASTICSEARCH_HOST                            <
          value: elasticsearch                                <
        - name: ELASTICSEARCH_PORT                            <
          value: "9200"                                       <
          value: changeme                                     |           value: DQ8rlxxxxxxxxMWsnEc
          value:                                              |           value: PHPMySQLDemo:dXMtY2VxxxxxxxwLmNsb3VkLmV

kubectl create -f metricbeat-kubernetes.yaml

kubectl get pods --namespace kube-system | egrep "beat|metric"
```
