
# Building the first cluster

1) Install k8s from packages  
2) Create Master Node  
3) Create Pod Networking  
4) Join additional Nodes as workers  


### Required packages  to be installed on all NODES
- kubelet  
- kubeadm  
- kubectl  
- Container Runtime ( Docker)  
  ** show current installed package version **   
   >  apt list kubelet

## Getting and installing K8S on ubuntu VMs ( DO on All Nodes )


### [Step-0] Setup all nodes with hosts file entry
   

### [Step-1] Master Node Setup with packages
   - Diable Swap  
      > swapoff -a
   - edit the fstab removing any entry for swap partitions and ReBOOT
      > vi /etc/fstab
     
   - Add Googles apt repo gpg key  
      > curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
     
   - Add the K8s apt repo into local repo  
     ``` 
     sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
     ```

   - Refresh the system repo with details updated and GPG key configured  
       ```
       sudo apt-get update
       apt-cache policy kubelet | head -n 20  # see which versions available
       apt-cache policy docker.io | head -n 20  
       ```
   - Install the required packages and __HOLD__ to prevent autoupdate
     
     | Pkg1 |   
     |---|  
     |kubelet| 
     |kubeadm|
     |kubectl|
     |docker.io|
     
     ```
     sudo apt-get install -y docker.io kubelet kubeadm kubectl  
     sudo apt-get hold -y docker.io kubelet kubeadm kubectl    
       OR    
     apt-mark hold docker.io kubelet kubeadm kubectl    
     apt-mark showhold    
     ```
     
   - In case of Ubuntu 20.10 
       curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
       Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
       
      


   - Check the system status of the container runtime / kubelet ( which enters a crashbackloop until its joined)     
        systemd[1]: kubelet.service: Failed with result 'exit-code'.

     ```
     sudo systemctl status kubelet.service  
     sudo systemctl status docker.service
     ```
     

   - Ensure both are set to start at system startup
      ```
      sudo systemctl enable kubelet.service  
      sudo systemctl enable docker.service
      ```  
### [Step-2]  Bootstrapping the cluster with *kubeadm*  
  customizable with params , but the default flow as below

 | Phase | Details | 
 |---|---| 
 | Preflisght checks | - admin permissions , pull container images of control plane , system resources , compatible runtime in autostart mode  |
 | Creates Certificate authority |  |  
 | Generates kubeconfig files | Authenticate agaisnt API server |  
 | Generates Static Pod mani |  For Control Plane PODs , generated in static file locations|  
 | Start Contol Plane|  |  
 | Taint Master | User pods not to go to master only system pods |  
 | Generate bootstrap token  | used to join other nodes |  
 | Start Addons Pods | DNS + kube-proxy |  

 
#### 2.1 Certificate Authority  
  
 - Self signed CA  
 - Can be part of an external PKI  
 - Securing cluster comunications   
 - Authentication of users and kubelets   
    /etc/kubernetes/pki  
 - Distributed to each node    

 *Study the external CA mode from docs*  

#### 2.2 kubeadm created kubeconfig files
  Used to define how to connect to your cluster.
    Certificate information ( client cert )   
    Cluster location ( where is the API server etc )
 
 > /etc/kubernetes

  | Config File | Description |  
  |---|---|
  |admin.conf|the admin/super user|
  |kubelet.conf| locate the api server and present the right client cert |
  |controller-manager.conf| - same as above - |
  |scheduler.conf| - same as above -|


####  2.3 Static Pod Manifests:  
 kubeadm init generates this for core cluster components in location. Kubelet monitors this directory for any change.
 Enables the start of these components without the start of the cluster.
  > /etc/kubernetes.manifests  

   - etcd  
   - API Server   
   - Controller Manager    
   - Scheduler  

###  2.3 Pod Networking   

  Single un NATed IP address per Pod.
  - Direct rounting - too complex to configure ?

  Overlay Networking ( Software -  Layer 3 Networking )
   - flannel [ L3 virtual N/w ]
   - Calico [ L3 and policy basedtraffic mgmt ]
   - Weavenet [ multihost DEocker ]

### [Step-3] Creating the master

  Download some yaml MANIFESTS that will describe the pod network. ( Find the latest working for the following two)
 
 1. Following describes the security needed to deploy our pod network :   
  > wget https://docs.projectcalico.org/archive/v3.3/kubernetes/installation/hosted/rbac-kdd.yaml

 2. Following describer the actual POD network : 
  > wget https://docs.projectcalico.org/archive/v3.3/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml 
 

 3. Defined the CIDR with kubeadm ( make sure updated in above files as well ) 
   
  Adjust the calico.yaml for the CIDR as per chosen subnet
  > vi calico.yaml  
  > kubeadm init --pod-network-cidr=192.168.0.0/16  
  > kubectl apply -f rbac-kdd.yaml  
  > kubectl apply -f calico.yaml  

  ```
   * understand TLS bootstrapping
  ```
4. Configure the master with current account to have admin access on the API server as a non priviledged account.  
  
   ```
    mkdir -p $HOME/.kube  
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
    sudo chown $(id -u):$(id -g) $HOME/.kube/config  
   ```