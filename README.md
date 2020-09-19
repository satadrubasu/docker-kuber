# docker-kuber
https://jsonpathfinder.com/

###  List of containers inside a pod:   
    > kubectl get pods [pod-name-here] -n [namespace] -o jsonpath='{.spec.containers[*].name}'

### kubectl filter and output control ( inbuilt GOLANG Templates )  
  
   > kubectl get no -o go-template='{{range .items}}{{if .spec.unschedulable}}{{.metadata.name}} {{.spec.externalID}}{{"\n"}}{{end}}{{end}}'  
   > or  
   > kubectl get no -o go-template="{{range .items}}{{if .spec.unschedulable}}{{.metadata.name}} {{.spec.externalID}}:{{end}}{{end}}" | tr ":" "\n"  
   
*author - could not find an easy way to print newline characters with inline golang template, so used a trick printing colons and using tr to convert colons to newlines.*

  #### RANGE to iterate  
  
   Kubectl supports a superset of JSONPath, with a special range keyword to iterate over ranges, using the same trick to add newlines:

  > kubectl get no -o jsonpath="{range.items[?(@.spec.unschedulable)]}{.metadata.name}:{end}" | tr ":" "\n"
  

### Print Custom Columns
   
  Print custom columns [Node]  
   >  kubectl get no -A -o=custom-columns=NAME:.metadata.name,CPU:.status.allocatable.cpu
   
 Service :
   >  kubectl get svc -A -o=custom-columns=Pod-Name:.metadata.name,Cluster-IP:spec.clusterIP
