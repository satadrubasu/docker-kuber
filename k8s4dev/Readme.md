


### Probes  
 
  
#### LivenessProbe: When should POD be restarted 
 

 Can only have following results :   
  Success / Failure / Unknown 

| Type | Desc |
|---|---|
| ExecAction |Executes an action inside the container|
| TCPSocket |TCP Check against the containers IPAddress on a specifieed port |
| HTTPGetAction | HTTP Get request against container |

#### 1) HttpGet liveness (e.g) 
  - Check /index.html on port 80
  - Wait 15 seconds
  - Timeout after 2 seconds
  - Check Every 5 seconds
  - Allow 1 failyre before failing POD
 ```
 containers:
 - name: my-nginx
   image: nginx:alpine
   livenessProbe:
     httpGet:
       path: /index.html
       port: 80
     initialDelaySeconds: 15
     timeoutSeconds: 2
     periodSeconds: 5
     failureThreshold: 1
 ```

 #### 2) ExecAction liveness (e.g) 
  - Define args for container  
  - Define action/command to execute  
  
 ```
 containers:
 - name: liveness
   image: k8s.gcr.io/busybox
   args:
   - /bin/sh
   - -c
   - touch /tmp/healthy; sleep30;
     rm -rf /tmp/healthy; sleep 600
   livenessProbe:
     exec:
       comand:
       - cat
       - /tmp/healthy
     initialDelaySeconds: 15
     timeoutSeconds: 2
     periodSeconds: 5
     
 ```


 #### ReadinessProbe:  

When should traffic be routed to the POD and serve to traffic.

 ---
  ## Deployments 

 |Item|Desc|
 |---|---|
|__POD__ | Can maintain containers within with liveness / readiness and restart as per.|
  | __ReplicaSet__ | Declarative way to manage the PODs. |
  | __Deployment__ | Declarative way to manage PODs , wraps ReplicaSet which came early. Ensure Pods stay running and can be used to scale|


#### A] ReplicaSet - act as POD Controller

 - Self Healing
 - Ensure requested number of PODs are available
 - Provide Fault Tolerance
 - Can be used to Scale PODS
 - Relies on POD Template
 - Used by Deployments

 #### B] Deployments - manages POD

  - PODs are managed using replica Sets
  - Scale ReplicaSets , which scale pods
  - Provides rollback 
  - Creates Unique Label that is assigned to the ReplicaSet and generated PODs
  - YAML is similar to ReplicaSet  

 
   
   * selector is used to select the template to use ( based on labels)

   ```
    apiVersion:apps/v1
    kind: Deployment
    metadata:
      name: frontend
      labels:
        app: my-nginx
        tier: frontend
    spec:
      replicas: 4
      selector:
        matchLabels:
          tier: frontend
      template:
        metadata:
          labels:
            tier: frontend
        spec:
          containers:
          - name: my-nginx
            image: nginx-alpine
   ```

 > kubectl create -f deployments,yml --save-config  
 > kubectl describe <pod | deployment> <podname | deployment-nsame>  
 > kubectl apply -f deployment.yml  
 > kubectl get deployments --show-labels  
 > kubectl get deployments -l app=my-nginx  
 > kubectl scale -f deployment.yml --replicas=4  
