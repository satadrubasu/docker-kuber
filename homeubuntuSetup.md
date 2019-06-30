Post install of kubeadm / kubernetes + minikube + virtualbox on my local :

1. Disable swap memory (if running)
 need to disable swap memory as Kubernetes does not perform properly on a system that is using swap memory.
 
 ``sudo swapoff -a``

2. https://kubernetes.io/docs/setup/learning-environment/minikube/
  
 
=========

Other installations 
1. minikube : Single Node kubernetes cluster
  https://kubernetes.io/docs/setup/learning-environment/minikube/
  ```
  minikube start
  minikube delete
  ```

2. Virtualbox - 6 : https://tecadmin.net/install-virtualbox-on-ubuntu-18-04/

