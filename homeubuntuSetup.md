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


## Working on labs.play-with-docker.com
  
  1. Login with docker credentials.Comes installed with __git__ and __docker__
  2. Install maven:
       ``apk add maven``
  3. Install JDK :
       ``apk add --no-cache openjdk8``
  
  Continue practicing by pulling images etc and use maven need be.
  
  **Reference commands:**
  setupNode.sh
```
#!/bin/bash
alias l='ls -lrt'
alias c='clear'
alias di='docker images'
alias dp='docker ps'
apk add maven
apk add --no-cache openjdk8
mkdir gitrepo
cd gitrepo
git clone https://github.com/satadrubasu/boot_docker_PropService.git
```
  
Other references for runtime
```
 docker pull satadrubasu/dockerplayground:latest
 docker run -d --name propService -p 8091:8091 satadrubasu/dockerplayground:latest
 docker inspect propService
 curl http://node1:8091
```
To execute commands inside the container use __exec__ on the container name
```
 docker exec propService <commmand>
 docker exec propService ls /usr
```



  
   
  
  
  

