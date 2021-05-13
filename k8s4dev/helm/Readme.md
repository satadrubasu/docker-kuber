
### Old Version of helm via brew on MAC 
 https://zoltanaltfatter.com/2017/09/07/Install-a-specific-version-of-formula-with-homebrew/  
 

###  Install helm on the host machine ()  

 1. Install the bianry on the machine  
    > brew install helm

    Check the active context of cluster of k8s to which helm will interact , just like kubectl.Helm will install to the cluster pointed by   
 
    current-context: docker-desktop  

    > kubectl config view

 2. Install the helm repository holding the charts 

  > helm repo add stable https://kubernetes-charts.storage.googleapis.com/
  > help repo list  


### Installing and uninstalling a chart:  

  1. INstalling mysql chart from stable repository created above: 
  
    > helm install demo-mysql stable/mysql
    > clear; kubectl get all  

   2. Uninstall the helm chart / application
    
     a) Clean Helm cache files on agent machine  
     b) Clean helm secrets in clusters  
     c) clean the app defined nby the charts  

     > helm uninstall demo-mysql
  
    Clean other directories found in :  
    > helm env  



## Helm Template Engine  

 Insert directive in code for runtime replacement to values. E.g for go template - {{ .name}}  

 Helm Template Test , at client level after replacing the values and before pushing to manifests in actual cluster:  
  > helm template [chart]  
  > helm install [release] [chart] --dry-run --debug 2>&1 | less

      ( can generate release name ) and gets to stderr so the &1 section.  

  #### Helm template load from values.yml

   1. Load value from values.yaml - observe the template variable with Caps.
    ```
     apiVersion: v1
     kind: Service
     metadata:
       name: {{.Release.Name}-{.Chart.Name}}
       labels:
         app.kubernetes.io/name: {{.Chart.Name}}

     spec:
       type: {{.Values.service.type}}  
       ports:
       - port: {{.Values.service.port}}
         targetPort: 80
        protocol: TCP
     selector:
       app.kubernetes.io/name: {{.Chart.Name}}
    ```

  #### Helm template Builtin Objects :

  1. Load value from chart.yaml - observe the template variable with Caps.
  ```
  chart.yaml
  name: mychart
  appVersion: "2.1"
  ```

  ```
   apiVersion: v1
   kind: Service
   metadata:
     name: {{.Chart.Name}}
  ```

2. Load value from External File conf.ini ( placed at  root of template dir )  
   ```
   apiVersion: v1
   kind: Service
   metadata:
     annotations: data.{{.Files.Get conf.ini}}
  ```



### Helm Repo commands

Helm stable repo + local repo to be added by helm repo add 

> helm repo list
> helm search [hub|repo] keyword
> helm inspect [all|readme|chart | values] chart_name
> helm show values
> helm fetch chart_name
> helm dependency update chart_name


### Wordpress demo
  
    https://hub.helm.sh/  

  > helm search repo wordpress  
  > helm install demo-wordpress stable/wordpress    


