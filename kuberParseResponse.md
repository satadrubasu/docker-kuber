
## 1. Variants to extract data from kubectl get commands

  * Custom Columns and Sorting
  * Golang templates
  * jsonpath

### 1.1 Custom Columns and Sorting  


  > kubectl get nodes -o wide --sort-by=.metadata.name  

  > kubectl get nodes -o=custom-columns=NAME:.metadata.name,POD CIDR:.spec.podCIDR,UNSCHEDULABLE:.spec.unschedulable  
 

### 1.2 Golang templates  

Printing colons and using tr to convert colons to newlines   

 > kubectl get no -o go-template='{{range .items}}{{if .spec.unschedulable}}{{.metadata.name}} {{.spec.externalID}}{{"\n"}}{{end}}{{end}}'  
 
 OR  
 > kubectl get no -o go-template="{{range .items}}{{if .spec.unschedulable}}{{.metadata.name}} {{.spec.externalID}}:{{end}}{{end}}" | tr ":" "\n"  


### 1.3 JsonPath  

 > kubectl get no -o jsonpath="{.items[?(@.spec.unschedulable)].metadata.name}"  

 > kubectl get no -o jsonpath="{range.items[?(@.spec.unschedulable)]}{.metadata.name}:{end}" | tr ":" "\n"  


## 2 Kinds  


### 2.1 Node   

 items  
 items[*].kind  
 items[*].metadata.name  
 items[*].metadata.creationTimestamp  
 items[*].metadata.labels {}  

 items[*].spec  
 items[*].spec.podCIDR  

 items[*].metadata.status  
 items[*].metadata.status.addresses[]  
  ```  {  
    "address":"192.168.200.3",  
    "type":"ExternalIP"  
    },  
    {  
    "address":"192.168.200.3",  
    "type":"InternalIP"  
    },  
    {  
    "address":"svi-tb18-master-1",  
    "type":"Hostname"  
    }  
```
items[*].status.allocatable.cpu  
items[*].status.allocatable.ephemeral-storage  
items[*].status.allocatable.memory  
items[*].status.allocatable.pods  


items[*].status.capacity.cpu  
items[*].status.capacity.ephemeral-storage  
items[*].status.capacity.memory  
items[*].status.capacity.pods  


items[*].status.nodeInfo.osImage  
items[*].status.nodeInfo.operatingSystem  
items[*].status.nodeInfo.kernelVersion  
items[*].status.nodeInfo.kubeletVersion  
items[*].status.nodeInfo.architecture  
items[*].status.nodeInfo.containerRuntimeVersion    
items[*].status.conditions[]  
   ```

{  
"lastHeartbeatTime":"2020-05-20T16:44:34Z",  
"lastTransitionTime":"2020-05-20T16:44:34Z",  
"message":"Calico is running on this node",  
"reason":"CalicoIsUp",  
"status":"False",  
"type":"NetworkUnavailable"  
},  
{    
"message":"kubelet has sufficient memory available",  
"reason":"KubeletHasSufficientMemory",  
"status":"False",  
"type":"MemoryPressure"  
},  
{  
 
"message":"kubelet has no disk pressure",  
"reason":"KubeletHasNoDiskPressure",  
"status":"False",  
"type":"DiskPressure"  
},  
{  
 
"message":"kubelet has sufficient PID available",  
"reason":"KubeletHasSufficientPID",  
"status":"False",  
"type":"PIDPressure"  
},  
  {  
 
   "message":"kubelet is posting ready status. AppArmor enabled",  
    "reason":"KubeletReady",  
   "status":"True",  
   "type":"Ready"  
  }  
```

## Command references :

#### Get ExternalIPs of all nodes  
 >  kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'  
 
#### List pods Sorted by Restart Count  
 >  kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'  

#### Get the version label of all pods with label app=cassandra  
 >  kubectl get pods --selector=app=cassandra rc -o jsonpath='{.items[*].metadata.labels.version}'  
 
#### Check which nodes are ready  

 ``` 
 JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \  
 && kubectl get nodes -o jsonpath=$JSONPATH | grep "Ready=True"
 ```
