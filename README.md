# Project versions: 
`current - October-10-2019`

Kubernetes: **v1.16.1** `[current v1.16.1]`

Node exporter: **v0.18.1** `[current v0.18.1]`

Kube-state-metrics: **v1.8.0** `[current v1.8.0]`

Prometheus: **v2.13.0** `[current v2.13.0]`

Grafana: **v5.0.0-5.0.4** `[current v6.4.2] *explained below`


# All private 

Namespace: **monprivate**

Folder: **manifests_monprivate**

![all-private](https://raw.githubusercontent.com/andre-lx/grafana-prometheus-exporters/master/docs/img/all_private.png)

# All exposed

Namespace: **monexposed**

Folder: **manifests_monexposed**

![all-exposed](https://raw.githubusercontent.com/andre-lx/grafana-prometheus-exporters/master/docs/img/all_exposed.png)

# Running:

```
git clone https://github.com/andre-lx/grafana-prometheus-exporters.git
cd grafana-prometheus-exporters/manifests_monXXXXX/

kubectl apply -f .
kubectl apply -f prometheus/kube-state-metrics/
kubectl apply -f prometheus/node-exporter/
kubectl apply -f prometheus/
kubectl apply -f grafana/
```

### Grafana Kubernetes APP

#### Install 

This manifests will also install the official grafana kubernetes plugin (not updated for a while). In order to use my fork (with the updated dashboards and metrics to match the new versions of the exporters), install git and git clone my fork ([pending pull request](https://github.com/grafana/kubernetes-app/pull/83)):

```
kubectl --namespace=monXXXXX exec -it pod/grafana-core-xxxxxx -- /bin/bash
cd /var/lib/grafana/plugins/
mv grafana-kubernetes-app/ grafana-kubernetes-app.old

apt-get update 
apt-get install git

git clone https://github.com/andre-lx/kubernetes-app.git grafana-kubernetes-app

Open grafana -> Plugin config -> Dashboards -> Re-import all
```

The official plugin works in the most recent grafana versions, but the custom pannels created for version 5.0.0 starts to fail in some functionalities for versions 5.0.4+.

# Config services

#### Prometheus:

![prometheus-config](https://raw.githubusercontent.com/andre-lx/grafana-prometheus-exporters/master/docs/img/config_prometheus.png)

#### Cluster config:

**Do not use the deploy button present in the plugin, since you already have all the needed exporters on the system**

![k8s-cluster-config](https://raw.githubusercontent.com/andre-lx/grafana-prometheus-exporters/master/docs/img/config_k8s.png)

# Accessing

#### monprivate:
```
Grafana:
kubectl port-forward --address=0.0.0.0 pod/grafana-core-xxxx 5000:3000 --namespace=monprivate

Prometheus:
kubectl port-forward --address=0.0.0.0 pod/prometheus-core-xxxx 5001:9090 --namespace=monprivate
```

Access using localhost:5000 and localhost:5001

#### monexposed:

For node exporters:

```
http://node_ip or master_node_ip:check port on get
```

For grafana, prometheus and kube-state-metrics exporter:

```
http://any_node_ip:check port on get
```

# Implementation result:

```
[root@k8s-master ~]# kubectl get all --namespace=monprivate
NAME                                                           
pod/grafana-core-6754fb48f8-zdgtg         
pod/kube-state-metrics-55fdc45f47-btwrx  
pod/prometheus-core-bf84cb6bb-5hfnk       
pod/prometheus-node-exporter-ntqm6      
pod/prometheus-node-exporter-asw22 
pod/prometheus-node-exporter-asd2g 
pod/prometheus-node-exporter-ertzs 

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    
service/grafana                    ClusterIP   10.101.192.15    <none>        3000/TCP   
service/kube-state-metrics         ClusterIP   10.110.250.218   <none>        8080/TCP   
service/prometheus                 ClusterIP   10.99.61.63      <none>        9090/TCP   
service/prometheus-node-exporter   ClusterIP   10.106.215.249   <none>        9100/TCP   

NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   
daemonset.apps/prometheus-node-exporter   4         4         4       4            4           <none>          

NAME                                 READY   UP-TO-DATE   AVAILABLE   
deployment.apps/grafana-core         1/1     1            1           
deployment.apps/kube-state-metrics   1/1     1            1           
deployment.apps/prometheus-core      1/1     1            1           

NAME                                            DESIRED   CURRENT   READY  
replicaset.apps/grafana-core-6754fb48f8         1         1         1       
replicaset.apps/kube-state-metrics-55fdc45f47   1         1         1       
replicaset.apps/prometheus-core-bf84cb6bb       1         1         1       
```

```
[root@k8s-master ~]# kubectl get all --namespace=monexposed
NAME                                      
pod/grafana-core-6754fb48f8-5cjbk         
pod/kube-state-metrics-55fdc45f47-zmskt   
pod/prometheus-core-bf84cb6bb-pj2k6       
pod/prometheus-node-exporter-24mjd        
pod/prometheus-node-exporter-bcgsz 
pod/prometheus-node-exporter-bfdsz 
pod/prometheus-node-exporter-dajqa

NAME                               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          
service/grafana                    NodePort   10.102.38.125   <none>        3000:32348/TCP   
service/kube-state-metrics         NodePort   10.109.35.231   <none>        8080:32137/TCP   
service/prometheus                 NodePort   10.105.164.24   <none>        9090:30975/TCP   
service/prometheus-node-exporter   NodePort   10.105.120.38   <none>        9100:30958/TCP   

NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   
daemonset.apps/prometheus-node-exporter   4         4         4       4            4           <none>          

NAME                                 READY   UP-TO-DATE   AVAILABLE   
deployment.apps/grafana-core         1/1     1            1           
deployment.apps/kube-state-metrics   1/1     1            1           
deployment.apps/prometheus-core      1/1     1            1           

NAME                                            DESIRED   CURRENT   READY   
replicaset.apps/grafana-core-6754fb48f8         1         1         1       
replicaset.apps/kube-state-metrics-55fdc45f47   1         1         1       
replicaset.apps/prometheus-core-bf84cb6bb       1         1         1       

```

***
***
***

## Credits:

This is an original fork from: [giantswarm/prometheus](https://github.com/giantswarm/prometheus)
