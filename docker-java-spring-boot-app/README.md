# Using Docker with your spring-boot application

This is a simple spring-boot application that is both built and executed using Docker.  This project is intended to give ideas/inspiration on how you might manage your Java application's build and release process.

## Motivation

Having spent some time building Docker images for Java and spring-boot applications over the past couple of years, I've found the current tooling and practices around using Docker with maven and gradle frustrating and overcomplicated.  I also feel that by using a maven plugin, we are missing out on the benefit of isolating the JDK and maven/gradle dependencies using Docker.

The maven/gradle plugin approach still assumes that the environment that you are running in has the correct JDK available for your application's build, and then also has the correct version of maven.  This usually means manually configuring these installs in your CI system.

After working with Docker built projects in other languages, I thought I'd come back and try to apply some of the practices I'd seen to see if it would make building a Java application seem less complex.

### Build Containers

By building and testing our applications in Docker containers, we can be sure that we have isolated our build environment from external dependency and configuration changes.  In this case, each project can specify its JDK and Maven/Gradle requirements, directly in its Dockerfile.  This reduces development environment and CI server setup complexity.


### Lightweight Runtime Containers

This setup creates a lightweight runtime container that is based on `alpine:3.3`, that only contains an JRE 8 install and our built jar.

## Usage

### Build, test and run

```shell
$ ./script/run make test jar image
$ docker run --rm -p 8080:8080 -it tombee/spring-boot-app
```

### Building with a specified version number

```shell
$ export VERSION=0.0.1
$ ./script/run make test jar image
$ docker run --rm -p 8080:8080 -it tombee/spring-boot-app:0.0.1
```

### Pushing an image

```shell
$ VERSION=0.0.1 ./script/run make push
```

### Using compose for development

```shell
$ docker-compose up
```

## How does it work?

### `Dockerfile.build`

The `Dockerfile.build` describes the full build/test environment that is required for this java application.  It is based on `maven`, since the spring-boot application requires maven for its build and test.  It includes the `docker` binary to allow this container to construct our runtime image.

### `Dockerfile`

The `Dockerfile` describes our lightweight runtime container, it should contain only the minimum set of files for our application to run.  At the moment, this produces a container that is approximately 139.4 MB.

### `Makefile`

The `Makefile` provides a series of targets to record the commonly executed commands to build the jar file and create the Docker image.  There are some variables in here to control the naming of the image produced, and the version number.  This Makefile can be further extended to suit your release process if an image needs to be pushed to Docker Hub or a private registry.

### `script/run`

This is a helper script to run our build environment container, it accepts parameters that you wish to run inside the container.  Given `./script/run make`, it will build the jar file and create a runtime docker image.

As an optimisation, it breaks filesystem isolation by volume mounting `~/.m2` onto the host's filesystem.  This helps to make incremental builds faster, this can be safely removed if this behaviour is unwanted.

It also bind mounts `~/.docker/config.json` to `/root/.docker/config.json` to use your registry authentication (`docker login`) details inside the container to be able to push.  If you don't want this behaviour, you could either tweak the scripts to accept registry credentials.  Another alternative would be to just run `make push` on the host.

### `docker-compose.yml`

I've also included a sample `docker-compose.yml` file here that can be used to show how to run this application during development.  It allows a developer to quickly get started by just running `docker-compose up` and having a working development environment, without having to have any of the maven/java tooling installed upon their host.  It uses our dev image for the container, since we need the build and test tooling.
