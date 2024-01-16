# Containerization and Orchestration Technologies - TP1 - Basics of containerization

## Theorical questions

1. What is a container? What are the main differences comparing to virtual machines? (short answer)
2. What is the difference between a container and an image?
3. Cite 3 advantages of using containers.
4. What is the purpose of the Docker engine?
5. How is built a Docker image? Give technical aspects (layers, ...) and how it is practically implemented? (complete answer)
6. What is the name of the resource used to publish and retrieve Docker images?
7. What happens to the data in a container when the container is stopped? And when it is removed?
8. What technique is used to prevent it?
9. By default, is it possible for a container to reach another running container on the same host? Explain why and how Docker can help.
10. If a container is using the host network and running a service on port 8080, can I run a service on the same port on the host? Why?
11. Same question with a bridge network.

## Practical work

### Run Hello World container

The Docker Hub registry contains a `hello-world` image that tests your ability to run Docker containers. When you run this image, it prints an informational message and exits.

1. Pull the `hello-world` image from the Docker Hub registry:
    
```bash
docker pull hello-world
```

2. Run the `hello-world` image and verify that it prints a message and exits:

```bash
docker run hello-world
```

It should print something like this:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Create our own Hello World image

In the previous step, we used an image that was created by someone else (the Docker team). Now, we will create our own image.

The main steps to create this image are:

1. Choose a base image (e.g. `alpine`)
2. Define the commands to run when the container is started
3. Build the image
4. Run a container based on this image
5. Check that everything works as expected

### Dockerize a simple web application

In this step, we will create a Docker image for a simple NodeJS web application. The application is a simple web server that returns a "Hello World" message.

To get the source code of the application, clone the following repository:

```bash
git clone TODO/helloworld-js
```

To run the application, NodeJS must be installed, and the following commands must be executed:

```bash
cd helloworld-js
npm install # install dependencies
npm start # start the application
```

The application is now running on port 3000. You can check it by opening the following URL in your browser: http://localhost:3000

The goal of this exercise is to create a Dockerfile using the previous commands to build and run the application in a container exposing the port 3000.

The main steps to create this image are:

1. Choose a base image (e.g. `node:slim`)
2. Copy the source code of the application in the image (e.g. in `/app`, think to change the working directory)
3. Install the dependencies
4. Define the commands to run when the container is started
5. Expose the port used by the application
6. Build the image, with the tag `helloworld-js:latest`
7. Run a container based on this image
8. If everything works as expected, the application should be accessible at http://localhost:3000

Congratulations, you have dockerized your first application!

### Dockerize using multi-stage builds

Docker provides a feature called [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) that allows to build an image in multiple stages. This feature is useful to reduce the size of the final image by removing the build dependencies.

In this exercise, we will use this feature to build a Docker image for a simple TypeScript web application. The application is a simple web server that returns a "Hello World" message.

To get the source code of the application, clone the following repository:

```bash
git clone TODO/helloworld-go
```

//TODO: add instructions to build and run the application
//TODO: Write the Dockerfile using multi-stage builds

### Mini chat application

In this exercise, we will create two Docker containers that will communicate with each other. We won't use Dockerfiles, but we will use the `docker run` command to create the containers, using the `busybox` image.

To do so, we will use the `nc` command, which is a utility to read and write data across network connections, using TCP or UDP protocol.

We want you to do the following steps:

1. Create a network called `chat`
2. Create a container called `server` that will be attached to the `chat` network
3. In this container, run the following command: `nc -l -p 3000` (this command will listen on port 3000)
4. Create a container called `client` that will be attached to the `chat` network
5. In this container, run the following command: `nc server 3000` (this command will connect to the `server` container on port 3000)
6. In the `client` container, type a message and press enter
7. Check that the message is received in the `server` container
8. In the `server` container, type a message and press enter
9. Check that the message is received in the `client` container

### Persistent Hello World container

In this exercise, we want to create a container that will store data in a persistent volume, and when the container is removed, the data must be kept.

To do so, we will use the `busybox` image, and the `docker run` command. First, we want to mount a volume inside the container, in the `/data` directory. Then, we want to create a file in this directory, and write "Hello persistent world" in `/data/persistent.txt`.

Remove the container, and create a new one, mounting the same volume in the same directory. Check that the file `/data/persistent.txt` exists, and contains the text "Hello persistent world".

### Publish an image on a registry

In this exercise, we want to publish the image we created in the previous exercise on a registry. To do so, we will use the [GitLab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/) from the [GitLab instance of the University of Geneva](https://gitlab.unige.ch/).

To publish an image on the registry, you must:

1. Create a new project on the GitLab instance
2. Activate the Container Registry feature: `Settings > General > Visibility, project features, permissions > Container Registry > Enabled`
3. Create a personal access token: `Account Preferences > Access Tokens > Add new token`, then select the `read_registry` and `write_registry` scopes, you can give it any name you want
4. Login to the registry using the `docker login -u <username> -p <access token> registry.gitlab.unige.ch` command
5. Tag the image using the following command: `docker tag <image> <registry>/<namespace>/<image>:<tag>`
6. Push the image using the following command: `docker push <registry>/<namespace>/<image>:<tag>`
7. Check that the image is available on the registry: go to the project page, then `Deploy > Container Registry`, you should see the image you just pushed

Now we can try to pull the image from the registry, and run a container based on this image:

1. Remove the image from your local machine
2. Pull the image from the registry
3. Run a container based on this image

