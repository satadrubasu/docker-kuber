Kubernetes with Minikube.Start ther First cluster to proceed.

### 1 SCENARIO: Launch Single Node Kubernetes Cluster

#### 1.1 Start Minikube
    > minikube -version
    > minikube start
#### 1.2 Cluster Info
    > kubectl cluster-info
    > kubectl get nodes
#### 1.3 Deploy Containers and check running pods
    > kubectl run first-deployment --image=katacoda/docker-http-server --port=80
    > kubectl get pods
    
#### 1.4 Expose container via diff n/w options. (Using NodePort - provides dynamic port to a container )
    > kubectl expose deployment first-deployment --port=80 --type=NodePort
    > export PORT=$(kubectl get svc first-deployment -o go-template='{{range.spec.ports}}{{if.nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}')
    
   above command finds the allocated port and executes a http request.
   
#### 1.5 Enable dashboard inminikube and make kubernetes dashboard available by deploying YAML   
    > minikube addons enable dashboard
    > kubectl apply -f /opt/kubernetes-dashboard.yaml
    
### 2. SCENARIO : KUBEADM ( solves handling TLS encryption config , deploy core Kubernetes components and enabling joining of nodes to the cluster. 
    
#### 2.1 Initialise MASTER
  Master  is responsible for running __CONTROL PLANE__ components. etcd / API Server.Clients will communicate to the API to schedule workloads and manage the state of the cluster.In PROD the token is auto generated we dont provide.
  
    > kubeadm init --token=<sometoken> --kubernetes-version $(kubeadm version -o short)

To manage the kube cluster , client config and certs are required.This cofig is created when kubeadm initializes.The command copies the configuration to the users home dir and sets the env variabble for use with the CLI.   

    > sudo cp /etc/kubernetes/admin.conf $HOME/
    > sudo chwon $(id -u):$(id -g) $HOME/admin.conf
    > export KUBECONFIG=$HOME/admin.conf

#### 2.2 Deploy Container Networking Interface (CNI) ( WeaveWorks)
Deployment definition. Which can be deployed using kubectl apply. This will deploy a sereis of Pods on the cluster.

    > cat /opt/weave-kube
    > kubectl apply -f /opt/weave-kube
    > kubectl get pod -n kube-system
    
#### 2.3 Join Cluster
  Once the master and the CNI has been initialized , additional nodes can join the cluster if they have the correct token.Token can be managed via __kubeadm token__
  
    > kubeadm token list
    > kubeadm join --discovery-token-unsafe-skip-ca-verification --token=<token> 172.17.0.26:6443
    > kubectl get nodes
 *kubectl* can now use the configuration to access the cluster.
  
#### 2.4 Deploy POD
  Once the nodes in the cluster are ready , deployments of PODs can be scheduled.Commands are always issued to the Master with each node only responsible for executing the workloads.
   
    > kubectl create deployment http --image=katacoda/docker-http-server:latest
    > kubectl get pods
    > docker ps | grep docker-http-server
    
#### 2.5 Deploy DASHBOARD
  Deploy the dashboard with yaml into the __kube-system__ namespace
  
    > kubectl apply -f dashboard.yaml
    > kubectl get pods -n kube-systems
    
  Creating a service account to access :
  ```
  cat <<EOF | kubectl create -f -
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: admin-user
    namespace: kube-system
  ---
  apiVersion: <blabla>
  kind: ClusterRoleBinding
  metadata:
    name: admin-user
  roleRef:
   apiGroup: <blabla>
   kind: ClusterRole
   name: cluster-admin
  subjects:
   kind: ServiceAccount
   name: admin-user
   namespace: kube-system
 EOF
```

Once the Service Account has been created , the token to login can be found:
   
    > kubectl -n kube-sysem describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
   
### 3. KUBECTL DEPLOYMENTS / REPLICATION CONTROLLERS and expose via SERVICES without YAML

#### 3.1 kubectl run ( like docker run but at a cluster level )

    > kubectl run http --image=katacoda/docker-http-server:latest --replicas=1
    > kubectl get deployments
    > kubectl describe deployment http
    
#### 3.2 kubectl expose ( create a service which exposes the PODS on a port )
  This command to expose the container port 80 on the host 8000 binding to the external-ip of the host.
  
    > kubectl expose deployment http --external-ip="172.17.0.36" --port=8000 --target-port=80
        OR
    > kubectl run httpexposed --image=katacoda/docker-http-server:latest --replicas=1 --port=80 --hostport=8001
       ( Under the covers this exposes the POD via Docker Port Mapping , so not listed as service when)
    > kubectl get svc
    > docker ps | grep httpexposed
 
#### 3.3 Scale Containers
  use kubectl to scale the number of replicas.Scaling the deployment will request Kubernetes to launch additional Pods.These Pods can be automatically load balanced using the exposed service.Once each pod starts it will be added to the LB service.By describing the service we can view the endpoint and associated Pods
  
    > kubectl scale --replicas=3 deployment http
    > kubectl get pods
    > kubectl describe svc http
    
 ### 4. KUBECTL DEPLOYMENTS / REPLICATION CONTROLLERS and expose via SERVICES with YAML 
   Most common object is the Kubernetes Deployment Object.It defines the container spec required,along with the name and labels used by other parts of Kubernetes to discover and connect to the application
    
    
 #### 5. Scenario : Deploy Guestbook example on Kube
   Store notes from guests in Redis via javascript API calls. Redis contains master ( for storage ) and a replicated set of redis *slaves*.Core concepts of following will be covered:
     - Pods
     - Replication Controllers 
     - Services
     - Nodeports
     
##### 5.1.1 REDIS-MASTER-CONTROLLER.YAML

Start the Redis Master. A Kubernetes service deployment has, at least, two parts. 
   i) __Replication controller__:
     defines how many instances should be running, the Docker Image to use, and a name to identify the service.Additional options can be utilized for configuration and discovery.If Redis were to go down, the replication controller would restart it on an active node.
     
     > kubectl create -f redis-master-controller.yaml
     > kubectl create -f redis-master-controller.yaml
     > kubectl get pods
     
   ii) __Service__ :  A Kubernetes service is a __named load balancer__ that proxies traffic to one or more containers. The proxy works even if the containers are on different nodes.Services proxy communicate within the cluster and rarely expose ports to an outside interface.
   A service looks like you cannot connect using curl or netcat unless you start it as part of Kubernetes. The recommended approach is to have a LoadBalancer service to handle external communications.

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: redis:3.0.7-alpine
        ports:
        - containerPort: 6379
  ```
##### 5.1.2 REDIS-MASTER-SERVICE.YAML

    > kubectl create -f redis-master-service.yaml
    > kubectl get services
    > kubectl describe services redis-master
    
```
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
    targetPort: 6379
  selector:
    name: redis-master

  ```
#####REDIS-SLAVE-CONTROLLER.YAML
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  replicas: 2
  selector:
    name: redis-slave
  template:
    metadata:
      labels:
        name: redis-slave
    spec:
      containers:
      - name: worker
        image: gcr.io/google_samples/gb-redisslave:v1
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access an environment variable to find the master
          # service's host, comment out the 'value: dns' line above, and
          # uncomment the line below.
          # value: env
        ports:
        - containerPort: 6379

```
#####REDIS-SLAVE-SERVICE.YAML
```
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
  selector:
    name: redis-slave
```

#####FRONTEND-CONTROLLER.YAML
``` 
apiVersion: v1
kind: ReplicationController
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  replicas: 3
  selector:
    name: frontend
  template:
    metadata:
      labels:
        name: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80
   ```     
#####FRONTEND-SERVICE.YAML
```
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  # type: LoadBalancer
  type: NodePort
  ports:
    # the port that this service should serve on
    - port: 80
      nodePort: 30080
  selector:
    name: frontend
  ```

  
  
  
    
    
  
  
  


   
 
