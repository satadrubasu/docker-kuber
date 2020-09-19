# docker-kuber


###  List of containers inside a pod:   
    > kubectl get pods [pod-name-here] -n [namespace] -o jsonpath='{.spec.containers[*].name}'
