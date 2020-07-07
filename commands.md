Once AWS EKS has been set up for hosting the kubernetes cluster, it had to be reached from the AWS CLI:

```shell
  aws eks update-kubeconfig --name prometheus_payever
```

Created nodes are shown:

```shell
  kubectl get nodes
```

Deploying Grafana and assign it to elb:

```shell
  kubectl create deployment grafana --image=docker.io/grafana/grafana:5.4.3
```

```shell
    kubectl expose deployment grafana --type=LoadBalancer --port=80 --target-port=3000 --protocol=TCP
```

```shell
    kubectl get service grafana
```

```shell
    export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

Creating namespace for Prometheus and deploying it:


```shell
    kubectl apply -f monitoring-namespace.yaml
```

```shell
    kubectl apply -f prometheus-config.yaml
```

```shell
    helm install stable/prometheus \ 
                 --name prometheus \
                 --namespace monitoring \
                 --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2",server.service.type=LoadBalancer
```

```shell
    kubectl --namespace=prometheus port-forward deploy/prometheus-server 9090
```

```shell
    export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus-mysql-exporter,release=mysqlexporter-release" -o jsonpath="{.items[0].metadata.name}")
```

MySQL eyporter will be available on localhost:8080 with port forwarding:

```shell
    kubectl port-forward $POD_NAME 8080:9104
```

Deploying MySQL instance to the cluster:

```shell
    kubectl apply -f https://k8s.io/examples/application/mysql/mysql-pv.yaml
```

```shell
    kubectl apply -f https://k8s.io/examples/application/mysql/mysql-deployment.yaml
```

Verifying the deployment:

```shell
    kubectl get pods -l app=mysql
```

Accessing MySQL instance:

```shell
    kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
```

I've enabled Slow query logging:

```shell
Kovatss-MacBook-Pro:~ kovatskristof$ kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword

If you don't see a command prompt, try pressing enter.

mysql> SET GLOBAL slow_query_log = 'ON';
Query OK, 0 rows affected (0.00 sec)

mysql> 
```
