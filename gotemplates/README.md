
#### Usage Reference :  
 
 > kubectl get pods -o go-template-file=podlist.gotemplate  


```
{{- range .items -}}
    {{.metadata.name}}{{"\n"}}
{{- end -}}
```

To fix the spaces : 
   To fix this, we will need to jump back to the Go template documentation on text and spaces. To summarize, adding a hyphen inside {{ }} will remove whitespace from the side that the hyphen is added to. So {{- }} will remove space to the left, and {{ -}} will remove space to the right, and {{- -}} will remove space from both sides. Let's put this into practice with our podlist.gotemplate file to get the output we are expecting. In your favorite editor, modify the file to look like this:  


### Service with multiple ports , and hunt for a specific one for ssh :

 File : service-selectport.gotemplate
```
{{- range .items -}}
    Service: {{.metadata.name}}
     {{range .spec.ports -}}
        {{- if eq .targetPort "ssh" -}}
            {{"\n"}} Name:   {{.name}}{{" "}}
             Port:   {{.port}}{{" "}}
             TargetPort:   {{.targetPort}}{{" "}}
        {{- end -}}
     {{"\n"}}
     {{- end -}}
{{- end -}}
```


```
spec:
  clusterIP: 193.21.14.58
  ports:
  - name: http
    port: 9009
    protocol: TCP
  - name: http-api
    port: 9090
    protocol: TCP
  - name: ssh
    port: 9024
    protocol: TCP
  type: ClusterIP
```

-------  



#### References :
  https://www.openshift.com/blog/customizing-oc-output-with-go-templates

  https://gist.github.com/so0k/42313dbb3b547a0f51a547bb968696ba
