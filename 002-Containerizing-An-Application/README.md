# Containerizing an Application

This lab will begin our DevOps journey by running a simple Golang application locally using Docker. Containers are at the core of how Kubernetes, and many other DevOps implementations, operate. This lab will set the stage for later, more advanced, labs.

## About the Application
The source code for the application located in the `src/link-unshorten` directory is a simple API that we will eventually Dockerize and deploy using Kubernetes. The API takes a shortened link as a parameter in the URL and returns the destination URL as JSON. The application is built using a lightweight HTTP web framework called [Gin-Gonic](https://github.com/gin-gonic/gin).

### Task 1: Create a Docker container for NodeJS/.NET CORE
#### NodeJS DockerFile

1. Make package.json: 
```
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "First Last <first.last@example.com>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```
2. Create a server.js file:
```
'use strict';

const express = require('express');

// Constants
const PORT = 8080;
const HOST = '0.0.0.0';

// App
const app = express();
app.get('/', (req, res) => {
  res.send('Hello world\n');
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```
3. Create a .dockerignore file:
```
node_modules
npm-debug.log
```
4. Create a Dockerfile:
```
FROM node:10

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]
```

#### .Net Core DockerFile

See https://docs.microsoft.com/en-us/dotnet/core/docker/build-container for more info.
```
FROM mcr.microsoft.com/dotnet/core/runtime:3.0

COPY app/bin/Release/netcoreapp3.0/publish/ app/

ENTRYPOINT ["dotnet", "app/myapp.dll"]
```

### Task 2: Browse the Application
Open up the files in `src/link-unshorten` in your favorite IDE or the Cloud Shell editor and familiarize yourself with the application.

### Task 3: Build the Docker Image
In the `src/link-unshorten` directory run the following command (substituting <yourname> with your own identifier) to build the image on the Playground Shell:
```
docker build -t <yourname>/link-unshorten:0.1 .
```

Inspect your Docker images:
```
docker images
```

### Task 4: Run the Docker Image
To run the application using Docker use the following command:
```
docker run -d -p 8080:8080 <yourname>/link-unshorten:0.1
```

Make sure the image is running without any errors:
```
docker ps
```

#### GKE specific
We will use the `Web Preview` feature within Cloud Shell to view our running container.

First, in the top-bar navigation of Cloud Shell (next to the editor button) click `Web Preview` and select "Preview on Port 8080".

This will open a new tab with a `appspot.com` URL and likely throw a 404. That's ok - we need to provide a valid API path.

Replace URL path with like so (your URL will look slightly different):

`https://8080-dot-1234567-dot-devshell.appspot.com/api/check?url=bit.ly/test`

#### minikube specific

Open `http://localhost:8080/api/check?url=bit.ly/test` in the browser.

### Task 5: Run Container as Non-Root User
1. First, let's take a look at the user our container is running as by using `exec` to get a shell.
```
# Grab container name from docker ps
docker ps

# use exec to get a shell into the running container
docker exec -it <container_name> /bin/bash
```

2. If we use the `whoami` command inside of the container we will see that the user is root
```
whoami
# root
exit
```
3. Open the `Dockerfile` located in the `src/link-unshorten` directory and remove the commented lines that declare a new user and build a new version of the image:

```
# First, stop the running container
docker stop <container_name>

# Uncomment the User lines in the Dockerfile then build the new image
docker build -t <yourname>/link-unshorten:0.2 .

# Run the image
docker run -d -p 8080:8080 <yourname>/link-unshorten:0.2
```

4. Run the following command to ensure you are no longer running as root:

```
# Get a shell to the new running container
docker exec -it <container_name> /bin/bash

# Once you are in the shell run the following commands
whoami
groups nonrootuser
```

### Bonus 1: Use the `--user` flag in `docker run` to run the original container as a non-root user.

### Bonus 2: Change the value for `port` in main.go an environment variable instead of a hardcoded value

Hint 1: Check out the [OS](https://golang.org/pkg/os) package for Golang

Hint 2: Use the `-e` flag to pass an environment variable into your `docker run` command

Hint 3: Yes, the answer is commented in the source code

Hint 4: You will need to run `docker stop` on the first running container before running another one with the same port

### Bonus 3: Inspect the Docker image
[dive](https://github.com/wagoodman/dive) is an OSS project that helps with visualization and optimization of images.

Install `dive` in Cloud Shell or on your local machine and inspect the unshorten image that was created.

Hint 1: For Cloud shell install using the instructions for Ubuntu/Debian.

### Bonus 4: Debug the Docker Container
How to debug a NodeJS Container in VS Code: https://dev.to/alex_barashkov/how-to-debug-nodejs-in-a-docker-container-bhi
