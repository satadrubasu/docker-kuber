
Lab1:
-----
 
List all the pods within all the namespaces.
    kubectl get pods --all-namespaces
List all the namespaces.
    kubectl get namespaces
See if there are any pods running in the default namespace.
    kubectl get pods
Find the IP address of the API server running on the master node.
    kubectl get pods --all-namespaces -o wide
See if there are any deployments.
    kubectl get deployments --all-namespaces
Find the label applied to the etcd pod on the master node.
    kubectl get pods --all-namespaces --show-labels -o wide
