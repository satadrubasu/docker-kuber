## References - CKA: 

https://kubernetes.io/docs/reference/kubectl/conventions/

Generate a pod.yaml without submitting for creation  
 > kubectl run nginx --image=nginx --dry-run=client -o yaml  
 
Generate deployment yaml  
 > kubectl create deployment --image=nginx nginx --dry-run=client -o yaml  

For k8s 1.19+ , pass the replica parameter
 > kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
 
