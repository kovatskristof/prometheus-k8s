kubectl --namespace=prometheus port-forward deploy/prometheus-server 9090

export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus-mysql-exporter,release=mysqlexporter-release" -o jsonpath="{.items[0].metadata.name}")

kubectl port-forward $POD_NAME 8080:9104

kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword