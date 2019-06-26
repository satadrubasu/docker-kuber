Post install of kubeadm and kubernetes on my local :


1. Disable swap memory (if running)
 need to disable swap memory as Kubernetes does not perform properly on a system that is using swap memory.
 ```   
 sudo swapoff -a
 ```
