
# Building the first cluster

1) Install k8s from packages  
2) Create Master Node  
3) Create Pod Networking  
4) Join additional Nodes as workers  



## Getting and installing K8S on ubuntu VMs ( DO on All Nodes )

| NodeType | hostname | IP | OS | ARCH | RAM | Processor |
|---|---|---|---|---|---|---|
|Master|satWS|192.168.1.5|Ubuntu 20.02.1 LTS|x86-64| 8 GB| Intel® Core™ i5-7200U CPU @ 2.50GHz × 4 |
|Worker|wrasp|192.168.1.11|Ubuntu 20.10|arm64| 8 GB| Raspberry Pi 4 ModelB rev 1.4 | 

** Some Handy Commands to check hardware :
  |1

### [Step-0] Setup all nodes with hosts file entry
   

### [Step-1] Linux host Steps  
   - Disable Firewall  
      > ufw disable  

   - Diable Swap  
      > swapoff -a  
   - edit the fstab removing any entry for swap partitions and ReBOOT 
      > vi /etc/fstab   OR  swapoff -a; sed -i '/swap/d' /etc/fstab

   - Update sysctl settings for Kubernetes networking  
     ```
     cat >>/etc/sysctl.d/kubernetes.conf<<EOF
     net.bridge.bridge-nf-call-ip6tables = 1
     net.bridge.bridge-nf-call-iptables = 1
     EOF
     sysctl --system
     ```
### [Step 2a] COntainer ( Docker Enginer Setup  )
```
 {  
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common  
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -  
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"  
  apt update  
  apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io  
  } 
```

### [Step-2b] K8S Setup 
- kubelet  
- kubeadm  
- kubectl  
- Container Runtime ( Docker)  
  ** show current installed package version **  
   >  apt list kubelet

- Add Googles apt repo gpg key  
   > curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -  
     
- Add the K8s apt repo into local repo  
     ``` 
     sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
      OR
     echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
     ```

- Refresh the system repo with details updated and GPG key configured    
       
     ```
       sudo apt-get update
       apt-cache policy kubelet | head -n 20  # see which versions available
       apt-cache policy docker.io | head -n 20  
     ```
- Install the required packages and __HOLD__ to prevent autoupdate  

     > i)  kubelet ii) 1.2 kubeadm   iii) kubectl   
     > iv) docker.io ( container )
     
     ```
     sudo apt-get install -y docker.io kubelet kubeadm kubectl  
     sudo apt-get hold -y docker.io kubelet kubeadm kubectl    
       OR    
     apt-mark hold docker.io kubelet kubeadm kubectl    
     apt-mark showhold    
     ```
    For Specific Versions :
     > sudo apt update && apt install -y kubeadm=1.18.5-00 kubelet=1.18.5-00 kubectl=1.18.5-00

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
     

### [Theory of kubeadm internals ]  Bootstrapping the cluster with *kubeadm*  
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
Initialize
 
 1. Update the below command with the ip address of kmaster   
  > kubeadm init --apiserver-advertise-address=192.168.1.11 --pod-network-cidr=192.168.25.0/16  --ignore-preflight-errors=all

 2. Deploy the Calico Network   
  > kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml 
   * Adjust the calico.yaml for the CIDR as per chosen subnet
   * refer to the repo calico.yaml checked in  

 3. Cluster join command   
   >  kubeadm token create --print-join-command  

4. To be able to run kubectl commands as non-root current user    
   ```
    mkdir -p $HOME/.kube  
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
    sudo chown $(id -u):$(id -g) $HOME/.kube/config  
   ```

  ```
  understand TLS bootstrapping
  ```
### [Step-4] Creating the Workers

 - Join the cluster  ( Use the output from kubeadm token create command in previous step from the master server )

   Sample Outputwhen kubeadm init of master:  
   ```
    
   ``` 
 - Verify the nodes and components status  
   >  kubectl get nodes
   >  kubectl get cs