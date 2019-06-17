### Docker Reference commands usecase based
###### ( All notes from practice on course at www.katacoda.com )
 
#### 1.1 SEARCH in the docker repos
  > docker search redis

#### 1.2 RUN docker images
  ```
  docker run -d redis      // run latest image
  docker run -d redis:3.2   // run version 3.2 of image
  docker run -d --name redisHost -p <host-port>:<container-port> redis:latest
  docker run -d --name redisMapped -v /opt/docker/data/redis:/data redis
 ```
#### 1.3 RUN Foreground 
  ```
  docker run -it ubuntu bash   // access bash shell inside of a container which was just run
  docker run ubuntu ps         // launches an Ubuntu container and executes the command ps
  ```
#### 1.4 STATUS commands
  ```
  docker ps                                     // list all running containers
  docker inspect <friendly-name| container-id>  // more details of the running container
  docker logs <friendly-name| container-id>     // logs, the container wrote to stderr/stdout
  docker images                                 // list os all images on the host
  docker ps --filter "label=user=scrapbook"
  docker images --filter "label=vendor=someVendor"
  ```
#### 1.5 PERSIST data external to the container as it gets destroyed 
  Containers are designed to be stateless.Binding of volumes using options.Docker uses $PWD as a placeholder for the current directory.
  option -v <host-dir>:<container-dir>
  > docker run -d --name redisMapped -v /opt/docker/data/redis:/data redispwd
  > docker run -d --name redisMapped -v "$PWD/data"

### 2.0 SCENARIO 
How to create a Docker Image for running a static HTML website using Nginx. Docker Images are built based on the contents of a Dockerfile.Note Dockerfile should be at the root folder where other needful files are stored.

#### CREATE a Dockerfile and also have some index.html in the same location
Dockerfile -->
  ```
  FROM nginx:alpine
  COPY index.html /usr/share/nginx/html/index.html
  EXPOSE 80
  CMD ["nginx", "-g", "daemon off;"]
  ```
#### BUILD Dockerfile 
  // The -t parameter allows you to specify a tag, commonly used as a version number
  ```
  docker build -t <build-directory>
  docker build -t webserver-image:v1 . // built image will have tag of v1
  docker images      // list all images on the host
  docker build -t my-nginx-image:latest .
  ```
1. __RUN__ <command> allows you to execute any command as you would at a command prompt, for example installing different application packages or running a build command. The results of the RUN are persisted to the image so it's important not to leave any unnecessary or temporary files on the disk as these will be included in the image.

2. __COPY__ <src> <dest> allows you to copy files from the directory containing the Dockerfile to the container's image. This is extremely useful for source code and assets that you want to be deployed inside your container

3. __CMD__ line in a Dockerfile defines the default command to run when a container is launched. If the command requires arguments then it's recommended to use an array, for example ["cmd", "-a", "arga value", "-b", "argb-value"], which will be combined together and the command cmd -a "arga value" -b argb-value would be run.
  An alternative approach to CMD is __ENTRYPOINT__. While a CMD can be overridden when the container starts, a ENTRYPOINT defines a command which can have arguments passed to it when the container launches.
In this example, NGINX would be the entrypoint with -g daemon off; the default command.

#### Exposing ports in Dockerfile 
  ```
  EXPOSE <port>   // instruct image to expose the specific port when run in container
  EXPOSE 80 443   // expose specific ports
  EXPOSE 8000-8900 // expose range of ports
  ```
### 3.0 Scenario : how to deploy a Node.js application within a container + CACHE Concepts
File-structure (node js application):
 Makefile , app.js , bin/ , package.json , public/ , routes/ , views/
 The next stage is to install the dependencies required to run the application.For Node.js this means running NPM install.

To keep build times to a minimum, Docker caches the results of executing a line in the Dockerfile for use in a future build. If something has changed, then Docker will invalidate the current and all following lines to ensure everything is up-to-date.

With NPM we only want to re-run npm install if something within our package.json file has changed. If nothing has changed then we can use the cache version to speed up deployment. By using COPY package.json <dest> we can cause the RUN npm install command to be invalidated if the package.json file has changed. If the file has not changed, then the cache will not be invalidated, and the cached results of the npm install command will be used
 
#### Dockerfile
  ```
   FROM node:10-alpine
   RUN mkdir -p /src/app
   WORKDIR /src/app
   COPY package.json /src/app/package.json
   RUN npm install
   COPY . /src/app
   EXPOSE 3000
   CMD [ "npm", "start" ]
  ```
   __docker build -t my-nodejs-app .__
   __docker run -d --name runningNodejsAapp -p 3000:3000 my-nodejs-app__
   __curl http://docker:3000__
After we've installed our dependencies, we want to copy over the rest of our application's source code. Splitting the installation of the dependencies and copying out source code enables us to use the cache when required.
   If we copied our code before running npm install then it would run every time as our code would have changed. By copying just package.json we can be sure that the cache is invalidated only when our package contents have changed.
   
 #### ENVIRONMENT VARIABLE
  environment variables can be defined when we launch the container. E.g with Node.js applications, define an environment variable for NODE_ENV when running in production.Using -e option, set the name and value as 
 > __-e NODE_ENV=production__
  ```
  docker run -d --name my-production-running-app __-e NODE_ENV=production__ -p 3000:3000 my-nodejs-app
  ```
 ### 4.0 SCENARIO : Optimise Dockerfile using the OnBuild instruction
   While Dockerfile's are executed in order from top to bottom, we can trigger an instruction to be executed later when the image is used as the base for another image.The result is you can delay your execution to be dependent on the application which you're building, for example the application's package.json file.
Below is the Node.js OnBuild Dockerfile. Unlike in our previous scenario the application specify commands have been prefixed with ONBUILD.
```
FROM node:7
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
ONBUILD COPY package.json /usr/src/app/
ONBUILD RUN npm install
ONBUILD COPY . /usr/src/app
CMD [ "npm", "start" ]
```
The result is that we can build this image but the application specific commands won't be executed until the built image is used as a base image. They'll then be executed as part of the base image's build.

### 5.0 Scenario : .dockerignore (Large temp folders / passwords etc).
  Just like .gitignore use this file to add all filre / folders / expressions to ignore copying to image
  Ignore sending tmp files for filesize calculation to engine during build

### 6.0 Scenario : Data Containers alternative to -v <host-dir>:<container-dir>
  __docker ps__  : Data containers are not listed in docker ps
 Sole responsibility is to be a place to store/manage data.Using __busybox__ as the base as it's small and lightweight image.
 1. Create a Data Container for storing configuration files using 
    > docker create -v /config --name dataContainer busybox
 2. Copy config.conf file into our dataContainer and the dir config.
    > docker cp config.conf dataContainer:/config/
 
 #### Mount volumes FROM
 Now our Data Container has our config, we can reference the container when we launch dependent containers requiring the configuration file.Using the __--volumes-from__ <container> option we can use the mount volumes from other containers inside the container being launched. 
 3. launch an Ubuntu container which has reference to our Data Container. When we list the config directory, it will show the files from the attached(/mounted dataContainer) container.
   > docker run --volumes-from dataContainer ubuntu ls /config
 * /config directory already existed then, the volumes-from would override and be the directory used*

 4. Export / Import Containers( DATA )
   > docker export dataContainer > dataContainer.tar
   > docker import dataContainer.tar
  
### 7.0 Multiple containers intercommunication
  Illustration on how the application container will connect to datastore ( Redis e.g ). The key aspect when creating a link is the friendly name of the container.
  
 1. Start Data Store ( start redis server with friendly name redis-server )
   > docker run -d --name redis-server redis
    
 2. To connect to a source container use the --link <container-name|id>:<alias> option when launching a new container.The container name refers to the source container we defined in the previous step while the alias defines the friendly name of the host.By setting an alias we separate how our application is configured to how the infrastructure is called.Bring up a Alpine container which is linked to our redis-server. Defined the alias as redis
 a) First, Docker will set some environment variables based on the linked to the container
   > docker run --link redis-server:redis alpine env
 
 b) Docker will update the HOSTS file of the container with an entry for our source container with three names, the original, the alias and the hash-id. Output the containers host entry using cat /etc/hosts
    
    > docker run --link redis-server:redis alpine cat /etc/hosts
    > 172.18.0.2      redis f53b3d584a47 redis-server
 ping the source container in the same way as if it were a server running in your network.
    
    > docker run --link redis-server:redis alpine ping -c 1 redis
    
### 8.0 Scenario : Docker networking ( Using DNS / Naming server ) /etc/resolv.conf
 Approach 1 --> --link which updates the /etc/hosts and environment vars and allow containers to discover and communicate.
 Approach 2 --> __docker Network__ where all containers are connected to,solves autoscaling better
                 With this approach the /etc/hosts and env variables are not updated
                 EMBEDDED DNS Server assigned to all containers via IP __127.0.0.11__ and set in __resolv.conf __
                 
  #### Create Network with name backend-network
    > docker network create backend-network
  #### Connect to network while running a container
    > docker run -d --name=redis --net=backend-network redis
  #### Query DNS server entry once a container is wired to created network
    > docker run --net=backend-network alpine cat /etc/resolv.conf
   When containers attempt to access other containers via a well-known name, such as Redis, the DNS server will return the IP address of the correct Container. In this case, the fully qualified name of Redis will be redis.backend-network.
    
    > docker run --net=backend-network alpine ping -c1 redis
    
 __Supports for multiple networks and containers being attached to more than one network at a time.__
   Note in the second command below __connect__ can be used to associate a container with a network apart from during a run
   
  #### Create another network ( frontend-network )
    > docker network create frontend-network
    > docker network connect frontend-network redis
    > docker run -d -p 3000:3000 --net=frontend-network katacoda/redis-node-docker-example
  
  #### Connect container with an alias
   Following command will connect our Redis instance to the frontend-network with the alias of db.When containers attempt to access a service via the name db, they will be given the IP address of our Redis container.
     
     > docker network create frontend-network2
     > docker network connect --alias db frontend-network2 redis
     > docker run --net=frontend-network2 alpine ping -c1 db
     
  #### Network Status and inspection / disconnect container
  
    > docker network ls
    > docker network inspect frontend-network
    > docker network disconnect frontend-network redis
 
 ### 9.0 Scenario : PERSISTING DATA USING VOLUMES
   Docker Volumes allow directories to be shared between containers and container versions.
   Docker Volumes allows you to upgrade containers, restart machines and share data without data loss. This is essential when updating database or application versions.Docker Volumes allow directories to be shared between containers and container versions.Docker Volumes allows you to upgrade containers, restart machines and share data without data loss. This is essential when updating database or application versions.
    
   #### Create data volume with -v
    > docker run  -v /docker/redis-data:/data \
    > --name r1 -d redis \
    > redis-server --appendonly yes
   We can pipe data into the Redis instance using the following command.
   
    > cat data | docker exec -i r1 redis-cli --pipe
  Redis will save this data to disk. On the host we can see the mapped direct which should contain the Redis data file.
    
    > ls /docker/redis-data
This same directory can be mounted to a second container. One usage is to have a Docker Container performing backup operations on your data.
    
    > docker run  -v /docker/redis-data:/backup ubuntu ls /backup
    
  #### Shared Volumes
   Data Volumes mapped to the host are great for persisting data. However, to gain access to them from another container you need to know the exact path which can make it error-prone.
An alternate approach is to use __-volumes-from__. The parameter maps the mapped volumes from the source container to the container being launched.
In this case, we're mapping our Redis container's volume to an Ubuntu container. The /data directory only exists within our Redis container, however, because of -volumes-from our Ubuntu container can access the data.
    
    > docker run --volumes-from r1 -it ubuntu ls /data
This allows us to access volumes from other containers without having to be concerned how they're persisted on the host.
   
  #### Permissions on volumes
   Available options are there e.g ro would mean read only
     
     > docker run -v /docker/redis-data:/data:ro -it ubuntu rm -rf /data
     
   
### 10.0 Scenario : MANAGING LOGS
  When we start a container, Docker will track the Standard Out and Standard Error outputs from the process and make them available via the client.y default, the Docker logs are outputting using the json-file logger meaning the output stored in a JSON file on the host. This can result in large files filling the disk. As a result, you can change the log driver to move to a different destination.
    
    > docker logs redis-server
   
 #### SysLog  
   The Syslog log driver will write all the container logs to the central syslog on the host.This log-driver is designed to be used when syslog is being collected and aggregated by an external system.
     Command to Redirect the redis logs to syslog:
      
      > docker run -d --name redis-syslog --log-driver=syslog redis
 ####  Accessing Logs :
   Attempting to view the logs using the client you'll recieve the error FATA[0000] "logs" command is supported only for "json-file" logging driver.Instead, you need to access them via the syslog stream
      
 #### Disable Logging ( For very verbose containers )
    > docker run -d --name redis-none --log-driver=none redis
    
 #### Which Configuration (*inspect* command )
    > docker inspect --format '{{ .HostConfig.LogConfig }}' redis-server
    > docker inspect --format '{{ .HostConfig.LogConfig }}' redis-syslog
    > docker inspect --format '{{ .HostConfig.LogConfig }}' redis-none

### 11.0 Scenario : PERSISTING DATA USING VOLUMES


### 12.0 Scenario : UPTIME for CONTAINER
   Like processes, containers can crash.Need to know hot to auto-restart if containers cash unexpectedly.
   Docker considers any containers to exit with a non-zero exit code to have crashed. By default a crashed container will remain stopped.
   Docker can automatically retry to launch the Docker a specific number of times before it stops trying.
   
 #### Restart on Failure ( attempt n times )
    > docker run -d --name restartThrice --restart=on-failure:3 scrapbook/docker-restart-example
    
 #### Restart Always
    > docker run -d --name restartAlways --restart=always scrapbook/docker-restart-example   
    
### 13.0 Scenario : METADATA and LABELS ( -l label=value )
  
#### Single Label
   
    > docker run -l user=21345 -d redis
 
#### External File with multi key value labels
  
    > echo 'user=1234' >> labels && echo 'role=cache' >> labels
    > docker run --label-file=labels -d redis
    
#### Labeling in docker IMAGE
  Labeling in image files is done in the Dockerfile
  
    > LABEL vendor=someValue    

#### Inspect Labels ( docker inspect )
By providing the running container's friendly name or hash id, you can query all of it's metadata.
Or by using the -f option to filter the json    
    
    > docker inspect rd
    > docker inspect -f "{{json .Config.Labels }}" rd
    
Inspect Lables of an IMAGE --> use ContainerConfig instead of Config
   
    > docker inspect -f "{{ json .ContainerConfig.Labels }}" example-tag

### 14.0 LOAD BALANCING CONTAINERS    
 E.g with NGINX web server to load balance requests between two containers running on the host.With Docker, there are two main ways for containers to communicate with each other. The first is via links which configure the container with environment variables and host entry allowing them to communicate. The second is using the Service Discovery pattern where uses information provided by third parties, in this scenario, it will be Docker's API.

The Service Discovery pattern is where the application uses a third party system to identify the location of the target service. For example, if our application wanted to talk to a database, it would first ask an API what the IP address of the database is. This pattern allows you to quickly reconfigure and scale your architectures with improved fault tolerance than fixed locations
  
#### NGINX Proxy  
  In this scenario, we want to have a NGINX service running which can dynamically discovery and update its load balance configuration when new containers are loaded. Thankfully this has already been created and is called nginx-proxy.

Nginx-proxy accepts HTTP requests and proxies the request to the appropriate container based on the request Hostname. This is transparent to the user with happens without any additional performance overhead.

##### Properties ( NGINX )
There are three keys properties required to be configured when launching the proxy container.

The first is binding the container to port 80 on the host using -p 80:80. This ensures all HTTP requests are handled by the proxy.

The second is to mount the docker.sock file. This is a connection to the Docker daemon running on the host and allows containers to access its metadata via the API. Nginx-proxy uses this to listen for events and then updates the NGINX configuration based on the container IP address. Mounting file works in the same way as directories using -v /var/run/docker.sock:/tmp/docker.sock:ro. We specify :ro to restrict access to read-only.

Finally, we can set an optional _-e DEFAULTHOST=<domain>. If a request comes in and doesn't make any specified hosts, then this is the container where the request will be handled. This enables you to run multiple websites with different domains on a single machine with a fall-back to a known website.

#### Launch nginx-proxy
Because we're using a DEFAULT_HOST, any requests which come in will be directed to the container that has been assigned the HOST proxy.example.
   
    > docker run -d -p 80:80 -e DEFAULT_HOST=proxy.example -v /var/run/docker.sock:/tmpdocker.sock.ro --name nginx jwilder/nginx-proxy
     
####  Starting container 1
For Nginx-proxy to start sending requests to a container you need to specify the VIRTUAL\_HOST environment variable. This variable defines the domain where requests will come from and should be handled by the container.In this scenario we'll set our HOST to match our DEFAULT_HOST so it will accept all requests.
    
    > docker run -d -p 80 -e VIRTUAL_HOST=proxy.example katacoda/docker-http-server
    
#### CLUSTER
We now have successfully created a container to handle our HTTP requests. If we launch a second container with the same VIRTUAL_HOST then nginx-proxy will configure the system in a round-robin load balanced scenario. This means that the first request will go to one container, the second request to a second container and then repeat in a circle. There is no limit to the number of nodes you can have running
 
    > docker run -d -p 80 -e VIRTUAL_HOST=proxy.example katacoda/docker-http-server
   
#### Generated NGINX Configuration
While nginx-proxy automatically creates and configures NGINX for us, if you're interested in what the final configuration looks like then you can output the complete config file with docker exec as shown below.

    > docker exec nginx cat /etc/nginx/conf.d/default.conf
    > docker logs nginx
     
 ### 15.0 SCENARIO : ORCHESTRATION using DOCKER COMPOSE
   Docker Compose is a tool to manage the orchestration of launching containers. It is based on a __docker-compose.yml__ file.
    container_name:
      property: value
      - or options
 #### 15.1 Defining First container
   Node js Application requires connecting to Redis.docker-compose.yml file in same project location as the Dockerfile.
   1. lets define a container called web based on the build of the current dir
      ```
       web:
        build: .
      ``` 
   2. Docker Compose supports all of the properties which can be defined using docker run
      ```
      links:
        - redis
      ports:
       -"3000"
       -"8000"
      ``` 
   3. In last step we used Dockerfile in the current dir as the base of our container.In this step we want to use an existing image from Docker Hub as a second container.
        ```
        redis:
        image: redis:alpine
        volumes:
         - /var/redis/data:/data
        ```
   4. Launch all applications with a single command of __up__
      If we want to bring up a single container then __up <container-name>__
      
          > docker-compose up -d
 
   5. Docker-Compose can also __SCALE__ number of containers  
       
         > docker-compose scale web=3
         > docker-compose scale web=1
         
   6. Docker-compose __STOP__
       
          > docker-compose stop
          > docker-compose rm
          

