# DevOps Deployment Project - SmartCow
## Preface
I am Wilson Chan and this is the writeup of the DevOps deployment project, which I have attempted to the best of my ability. For this project, I am using Windows 10 Home edition with the assistance of Docker Desktop and Visual Studio Code IDE.
## Introduction
With the emergence of early OS-level virtualisation such that of FreeBSD jails to the modern Kubernetes, virtualisation has evolved to be a developer-friendly tool highly beneficial to the world of development operations. Especially in the case of the technology stack in question, Docker, containerisation transforms the traditional way of software packaging, delivery and deployment into a streamlined ecosystem. In a software container, applications (along with dependency libraries) are packaged into container images to be executed on a target machine. This brings about multiple benefits, such as deployment consistency, service reusability (by promoting microservices design) and cost-effectiveness. Automation is also an option for container deployment if a user chooses to use container orchestration mediums.  
This project is a Python based React Web application that, as per namesake, contains a Python-based backend API as well as a React-based frontend. When both are running, it displays a web application that shows RAM and CPU usage on two statistics on the local system. As part of this program, I will attempt to run the application from a containerised setting using Docker technology. 

![React](https://user-images.githubusercontent.com/17082681/172428183-6e2b7d60-7f29-4d0e-9d8c-330333aff173.PNG)


## Running the Application in Docker
### Building the images
The first step to running the components in a Docker environment is to create their respective Docker images. To do this, a file known as a Dockerfile (and simply named Dockerfile) is to be created in the root of both frontend and backend components. The Dockerfile for the Python API is as follows.

```
FROM python:3.9                                            #Specifies which Docker base to build the image off, Python 3.9 

WORKDIR /app                                               #Defines the work directory in container 

COPY requirements.txt requirements.txt                     #Copies dependency requirements

RUN pip3 install -r requirements.txt                       #Installs requirements using pip tool

COPY . .                                                   #Copies the greater directory

EXPOSE 3000                                                #Exposes the connection port, 3000

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]  #Task initiation command, written in a list format
```
In the requirements.txt file, I have included the dependencies required to run the application. The Python dependencies required are Flask and Flask Cors, as well as the PSUTIL package required for the app's system monitoring functionalities.

The Dockerfile for the React Frontend is as follows.
```
FROM node:16-alpine                                        #Specifies which Docker base to build the image off, Node Alpine (16)

WORKDIR /app                                               #Defines the work directory in container 

ENV PATH ="./node_modules/.bin:$PATH"                      #Defines node_modules file directory

COPY . .                                                   #Copies the greater directory

RUN npm install                                            #Install and run commands separated for clarity

RUN npm run build

EXPOSE 3000                                                #Exposes the connection port, 3000

CMD ["npm", "start"]                                       #Starts the frontend, written in a list format
```
Alpine OS 16 was picked for this purpose as workaround to Error: error:0308010C:digital envelope routines::unsupported.   
The added ENV path for node_modules ensures Docker may find the module files and prevent further errors.   

Now that the Dockerfiles are created, they must be built individually (and to test). To do this, in directory, run
```
docker build *DockerfileLocation* --tag Name               #Where DockerfileLocation is the file location.
```

![apiBuild](https://user-images.githubusercontent.com/17082681/172440943-b1a25cdb-839d-4696-842a-5f6cfff565f3.PNG)

When the build is successful, the container may be initiated by command...
```
docker run -d -p 8000:5000 dockerapi
```
...or on the Desktop Application. The UI really makes everything easier.

![dockerdesktopapi](https://user-images.githubusercontent.com/17082681/172441844-2e666245-2e43-427f-ab6b-9e21a1a49cb3.PNG)

The react application is to be built in the same manner. With both deployed, the application works as intended.

### Docker Compose
A feature of Docker known as Docker Compose allows the continuous, tandem deployment of multi-containered applications. To do this, we simply have to create a Docker-compose yaml file, creatively named docker-compose.yaml. It is to be placed in the root of the project. The Docker Compose yaml defines the makeup and paths of respective container directories, such as build and exposed ports.

```
version: "3"  
services:
  api:
    build: ./api
    container_name: api_c
    ports:
      - '4000:4000'
    volumes:
      - ./api:/app
  sys-stats:
    build: ./sys-stats
    container_name: sys-stats
    ports:
      - '3000:3000'
```

To deploy, while in directory:
```
docker-compose up
```
![dockercompose](https://user-images.githubusercontent.com/17082681/172444259-40d29e0d-1736-4184-befa-6095d68c5204.PNG)

Again, Docker Desktop helps a ton with the management of local images.
![DockerDOA](https://user-images.githubusercontent.com/17082681/172444504-34040de9-b7d8-41f6-90a8-8cbb58a4a77e.PNG)

NGINX was installed locally and redirected to port 4000


## Deployment to Kubernetes/Minikube
### Tools required
- Kubectl: Kubernetes command line tool
- Minikube: Lightweight local Kubernetes service
- Kubernetes Dashboard (also included in Minikube install)
- Kompose: Kubernetes conversion tool that converts Docker Compose yamls to K8s Manifests

I installed them using Chocolatey since i'm on Windows.  

### Kompose Configuration
Kompose is a conversion tool that handily converts Docker Compose yamls to K8s manifest files, which are required for deployment. To convert a file to manifest, in directory, I used...
```
kompose convert -f docker-compose.yaml -o manifests.yaml
```
...to punch it all neatly into a single manifest.yaml output. I can now apply...
```
kubectl apply -f manifests.yml
```
...to deploy the manifest.   
![configure](https://user-images.githubusercontent.com/17082681/172627132-fba0372e-27b1-4a2a-8d20-6faa1703ffb6.PNG)  

After deployment is complete, show available instances.
```
kubectl get po -A
```

### Minikube Testing
To test minikube functionality, we can trial Minikube itself.
```
minikube start
```
![minikubestart](https://user-images.githubusercontent.com/17082681/172625791-7cae1d64-0234-4264-a1a7-902d093fc4c6.PNG)

```
minikube status
```
![minikubestatus](https://user-images.githubusercontent.com/17082681/172626015-bf9789e1-128c-4fdd-b293-a12c3ae4f115.PNG)   
This shows us that Minikube is running properly.
Minikube also has its own dashboard for monitoring deployment. A separate version can be installed as Kubernetes UI, but we can use it from here
```
minikube dashboard
```
![dashboard](https://user-images.githubusercontent.com/17082681/172626637-03662afd-c847-480a-a5bc-e94fa4b2fcc9.PNG)
![dashboard2](https://user-images.githubusercontent.com/17082681/172626660-820f3d3f-cade-4feb-9734-f1a7171d4787.PNG)

To accurately set the context environment Minikube will run (and instead of the default Docker runtime), we have to point it in the right direction.
```
minikube docker-env
minikube -p minikube docker-env --shell powershell | Invoke-Expression               #WINDOWS ONLY
```  
Now, with this command, the current shell is configured for use with the Minikube environment (daemon).   
I have run into an issue where Minikube will attempt to use other Docker instances, so to point it to the correct environment is important.
In the same shell, we will build the images and run them.
```
docker build -t sys-stats .
docker image ls
```
![imagelsdock](https://user-images.githubusercontent.com/17082681/172763451-9f55073b-5786-4f09-9352-4b16624cc651.PNG)
```
kubectl run sysstats2 --image=sys-stats --image-pull-policy=Never    
```   
It is imperative that the image pull policy tag be set as Minikube will attempt to query and pull from Docker Hub, the Docker default registry, instead of looking around for a local image, which is what it is. This can also be set in the Docker Compose manifest with the same tag under containers. Without it, it will be stuck with the error 'ErrorImgPull' where it fails to pull an image.    
What's left is to run startup for all instances or for Kompose and we should be good to go.  
![bothrun](https://user-images.githubusercontent.com/17082681/172763859-229b4d25-fbd9-40d8-8298-a1b271971780.PNG)  
Container status can also be found in the minikube dashboard too, which is pretty neat.
![pods](https://user-images.githubusercontent.com/17082681/172763981-6d763f76-30d8-421c-b180-4b3bfee229e2.PNG)

### Deployment with Kompose
Now I want to deploy the whole application with its containers, which i will use the Kompose manifest created previously.
```
kubectl apply -f ./manifests.yaml
```   
This would apply the Kompose configuration that I have setup.  
Bug note: I had to add a sleep command to my api section of the Kompose YAML as somehow the api container would always get stuck in a restart loop and fail (CrashLoopBackOff). A way I found to resolve it is to artificially add a command that forces it to stay up. 
```
spec:
          containers:
            - image: api
              name: api-c
              ports:
                - containerPort: 4000
              resources: {}
              volumeMounts:
                - mountPath: /app
                  name: api-claim0
              imagePullPolicy: Never
              command: ["/bin/sh", "-ec", "sleep 1000"]      #Sleep cmd
```
Running 
```kubectl get deployment``` and checking on dashboard confirms the success.

![DeploymentMap](https://user-images.githubusercontent.com/17082681/172779964-f5320856-1220-4619-a666-d7d3565c04c6.PNG)
![Kubectl Deploy](https://user-images.githubusercontent.com/17082681/172780143-dfce2486-0ba1-4812-a249-df5646257aca.PNG)

We can now proceed to expose this deployment with a load balancer, as specified in the deployment manifest.
```
kubectl expose deployment api --type="LoadBalancer"
```





## Deployment to Amazon EC2/EBS
TBD


