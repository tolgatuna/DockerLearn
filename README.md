# Docker Cheat Sheet

## Docker Commands
- `docker run <image_name>` <br/>
Run an image

- `docker run <image_name> <command>` <br/>
Run an image with a command (it is replacing RUN default command which is already exist in docker image itself)

- `docker ps`<br/>
Display running containers

- `docker ps --all` = `docker ps -a`<br/>
Display all created containers

- `docker run <image_name>` = `docker create <image_name>` + `docker start -a <container_id>`<br/>
`-a` : show output

- `docker run -d <image_name>`<br/>
`-d` : Run in the background and it will just return the id of the running container

- `docker run <image_name> <command>`<br/>
`<command>` : Run image with a command instead of default command

- `docker start -a <container_id>` <br/>
Run created container (you can run again a container which is already created which you listed with docker ps -a)

- `docker system prune`<br/>
It is gonna remove:
	- all stopped containers
	- all networks not used by at least one working container
	- all dangling images
	- all build cache

- `docker logs <container_id>`<br/>
Printing logs of container (for example if you forgot to put -a while starting the container, you can use that command to see what happened)

- `docker attach <container_id>`<br/>
Attaching our terminal to the running container and we can see outputs and we can give inputs.

- `docker stop <container_id>` <br/>
That one sending a SIGTERM message to container. With this signal, container complete it is process and than close itself and it can take some time.

- `docker kill <container_id>` <br/>
That one sending a SIGKILL message to container. With this signal, container kill itself IMMEDIATELY

- `docker exec -it <container_id> <command>` <br/>
Execute an additional(second) command in container.<br/>
	`-it` = `-i` + `-t` = allows us to enter some input with given command and show output of command (if command has input of course)<br/>
	`-i` = adding input channel<br/>
	`-t` = display output with proper format (it adds auto complete also)
	#### Example:
	- `docker run redis` <br/>
			it is gonna run redis-server
	- `docker exec -it <redis_container_id> redis-cli`<br/>
			it is gonna run redis-cli and give chance us to use cli to get or set values to our redis-server

- `docker exec -it <container_id> sh` OR `docker exec -it <container_id> /bin/bash`<br/>
sh or /bin/bash gives full terminal access of that container. (bash or sh changes according to container)

- `docker run -it <image_name> sh`<br/>
We can directly run a image with `-it` and `sh` and we can directly interactive with the container.

- `docker build -t tolga/redis:latest .` <br/>
`-t` means tag the image which we created. Tags are created with given pathern `<your_docker_id>/<image_name>:<version(optional)>` <br/>
`.` means build Dockerfile in current directory

- `docker build -f Dockerfile.dev .` <br/>
`-f` means file which docker should be built from.

- `docker run -p <incoming_request_port_on_local_host>:<port_inside_the_container> <image_name>` <br/>
Docker run with port mapping (Ex: `docker run -p 8080:8080 myImage`)

- `docker run -v <bookmarked_volume_path> -v <local_path>:<mapped_volume> <image_name>` <br/>
There are two usage of `-v` we have in docker:
	- If we put `:` while giving the volume  (__in command:__ `-v <local_path>:<mapped_volume>`), it means we are mapping our __local_path__ with the __mapped_volume__. That means we are actually using the __local_path__ as volume and __mapped_volume__ is just used as reference of our __local_path__.
	- If we do __NOT__ put `:` while giving volume path (__in command:__ `-v <bookmarked_volume_path>`), it is going to put bookmark on given path. That means we are saying if we map that volume somehow, do not use __bookmarked\_volume\_path__ as a reference. Use it still as a memory inside the docker container.

	##### Real example for both cases:
	`docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app myReactAppImage` <br/>
	Here we are using our pwd as app. And app just a reference for our current directory. We actually mapped all app directory. But inside the app directory, we bookmarked the node\_modules. And we are saying keep that node\_modules outside of that mapping and use node_modules which is already exists inside the docker container.



## Docker File Properties
- `FROM <base_image>` <br/>
Specify base image (ex: `FROM alpine`)

- `FROM <base_image> as <step>` <br/>
Specify base image and give as step. We can use it as multistep buildings.

- `WORKDIR <Path>` <br/>
Any following command will be executed relative to this path in the container

- `RUN <Command>` <br/>
Run the given command (ex: `RUN apk add --update redis`)

- `CMD ["<command>"]` <br/>
cmd gives startup command to the image. (ex: `CMD ["redis-server"]`, ex: `CMD ["npm", "start"]`)

- `COPY <copy_path_from_your_computer> <destination_path_inside_the_image>` <br/>
It copy files/directories from your computer to the image. (ex: `COPY ./ ./`)

- `COPY --from=<step> <copy_path_from_step> <destination_path_inside_the_image>`<br/>
If we use multistep builds we can give step name `--from=<step>` which means copy from that image step

- `EXPOSE <port>` <br/>
We do not need to say -p before run the image inside the elasticbeanstalk. For local computer it means nothing and it is not going to do anything automatically. 

- Multistep Docker file example :

	``` bash
	FROM node:alpine as builder
	WORKDIR '/app'
	COPY . .
	RUN npm install
	RUN npm run build

	FROM nginx
	COPY --from=builder /app/build /usr/share/nginx/html
	```


## Docker Compose File & Commands
### Docker Compose File
- Simple docker-compose.yml example:

	``` yaml
	version: '3'
	services:
		redis-server:
			image: 'redis'
		node-app:
			build: .
			ports:
				- "8081:8081"
	```

In given file, we have two services one is directly image 'redis' and other one is `build .` (our docker image). And for our image we mapped port 8081 to 8081 as we did in run command<br/>

___PS:___ Do ___NOT___ forget to change redis url as `redis-server` in application.yaml or settings file which you have to connect right redis server!

- docker-compose file restart policies:
	- ___'no'___ : Never attempt to restart this container it it stops or crashes.
	- ___always___: If this container stops *for any reason* always attempt to restart it
	- ___on-failure___: Only restart if the container stops with an error code
	- ___unless-stopped___: Always restart unless user forcibly stop it

	Example:
	
	```yaml
	node-app:
		restart: always
		build: .
		ports:
			- "8081:8081"
	```

- we can put `-it` tag also with `stdin_open: true` inside the docker-compose.

- We can give Dockerfile also in build (Same as `docker build -f Dockerfile.dev .`)
	
	``` yaml
	build:
		context: .
		dockerfile: Dockerfile.dev
	```

- We can change also default command as we did in docker run command `docker run <image_name> <command>` :<br/>
	`command: ["npm", "run", "test"]`

### Docker Compose Commands
- `docker-compose up` <br/>
Run docker compose file services

- `docker-compose up -d` <br/>
Run docker compose file services in the background

- `docker-compose ps` <br/>
Prints status of services which the compose file has.

- `docker-compose down` <br/>
Stop docker compose file services

- `docker-compose up --build` <br/>
Rebuild the images if exist and run docker compose



## Kubernetes Commands & Notes
### Kubernetes Commands
- `kubectl cluster-info` <br/>
Returns information about kubernetes server

- `kubectl apply -f <filename>` <br/>
Feeding a config file to Kubectl

- `kubectl get pods` <br/>
Print the status of __all__ running pods.

- `kubectl get services` <br/>
Print the status of __all__ services pods.

### Kubernetes Notes
- Each API version defines a different set of __Objects__ which we can use and decleared as `apiVersion: <Version>` (__Ex:__ `apiVersion: v1`, `apiVersion: apps/v1`...) at the beginning of the config file.

- Config files of kubernetes create __Objects__ which serve different purposes like running a container, monitoring a container, setting up networking, etc. Type of object decleared with `kind: <TYPE>` (__Ex:__ `kind: Service`, `kind: Pod`...) in a config file. Object type examples of kubernetes:
	- StatefulSet
	- ReplicaController
	- Pod (runs one or more closely related containers)
	- Service (sets up networking in a Kubernetes Cluster). Some subtypes of service :
		- ClusterIP
		- NodePort (exposes a container to the outside world and only good for __development__ porposes.)
		- LoadBalancer
		- Ingress

- Kubernetes __important takeaways__:
	- Kubernetes is a system to deploy containerized apps
	- __Nodes__ are individual machines (or virtual machines) that run containers
	- __Masters__ are machines (or virtual machines) with a set of programs to manage nodes
	- Kubernetes didn't build our images. It got them from some hub or somewhere else
	- Kubernetes (the master) decided where to run each container \- each node can run a dissimilar set of containers.
	- To deploy something, we upgrade the desired state of the master wuth a config file
	- The master works constantly to meet your desired state
- __Imperative Deployments__ : Do exactly there steps to arrive at this container setup<br/>
__Declarative Deployments__ : Our container setup should look like this make it happen


## NOTES
- Dockerfile is the default file which keeps building commands. If you want to use another file you need to use `-f` while using `docker build` command.
- docker-compose.yml is the default file for docker compose.
- By default docker image can access any port from inside to outside. But to access any port of container from outside to inside, we have to define port mapping.
- Ports do NOT have to be identical while running an image with port mapping <br/>
(Ex: `docker run -p 5000:8080 myImage` -> this will redirect local port 5000 to docker port 8080)
- What is docker compose? and what is the purpose?
	- Separate CLI that gets installed along with Docker
	- Used to start up multiple Docker containers at the same time
	- Automates some of the long-winded arguments we were passing to `docker run`
- Docker compose commands looking for the current directory and searching for the docker-compose.yaml in the current directory.
- Travis.ci works with travis.yaml
- Kubernetes is a system for running many different containers over multiple different machines. It is used to run many different containers with different images.
- Some notes about kubernetes: 
	- Kubernetes expects all images to already be build
	- One config file per object we want to create
	- We have to manually set up all networking
