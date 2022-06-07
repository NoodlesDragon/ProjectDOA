# DevOps Deployment Project - SmartCow
## Preface
I am Wilson Chan and this is the writeup of the DevOps deployment project, which I have attempted to the best of my ability. For this project, I am using Windows 10 Home edition with the assistance of Docker Desktop and Visual Studio Code IDE.
## Introduction
Virtualisation as a concept is hardly a new idea, ranging back to the 1980s. With the emergence of early OS-level virtualisation such that of FreeBSD jails to the modern Kubernetes, virtualisation has evolved to be a developer-friendly tool highly beneficial to the world of development operations. Especially in the case of the technology stack in question, Docker, containerisation transforms the traditional way of software packaging, delivery and deployment into a streamlined ecosystem. In a software container, applications (along with dependency libraries) are packaged into container images to be executed on a target machine. This brings about multiple benefits, such as deployment consistency, service reusability (by promoting microservices design) and cost-effectiveness. Automation is also an option for container deployment if a user chooses to use container orchestration mediums.  
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
From the completed Docker compose yaml, I used the Kompose conversion tool to convert to a Kubernetes manifest file. Initially, it did not work as the file paths had to be manually corrected. I managed, preliminarily, on Windows Docker Desktop’s Kubernetes integration as well as Kubernetes UI build with kubectl.

![kubectl](https://user-images.githubusercontent.com/17082681/172453412-208ba22e-8fb5-4ceb-a2f1-80b69dfd214b.PNG)


## Deployment to Amazon EC2/EBS
TBD


