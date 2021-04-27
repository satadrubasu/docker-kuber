

### Cluster DNS

 DNS available as a Service in a cluster.
   2 pod - loadbalancing

### Sample way 

 > kubectl create deployment hello-clusterip --image=gcr.io/google-samples/hello-app:1.0  

 > kubectl expose deployment hello-clusterip --port=80 --target-port=8080 --type ClusterIP  

 > kubectl get service -A  

> SERVICEIP=$(kubectl get service hello-clusterip -o jsonpath='{ .spec.clusterIP}')  

Access the service inside the cluster  
   > curl http://$SERVICEIP

Get Listing of the endpoints for  service  
 > kubectl get endpoints hello-clusterip  
