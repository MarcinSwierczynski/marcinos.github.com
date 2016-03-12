---
date: 2016-03-12 18:21:00 +0100
layout: post
slug: migrating-spring-boot-application-to-docker
status: publish
title: Migrating Spring Boot Application to Docker
categories:
- Programming
- Java
- Spring Boot
- Docker
---

## What are we going to do?

In this post we’re going to prepare a Spring Boot application to work under Docker. Then, we’ll publish our app image to [DockerHub][1].

### Prerequisites

### Docker
We’ll need a Docker installed on a local machine.

To run Docker on machine running under different system than Linux, we’ll also need [VirtualBox][2] installed.

Nice introduction on how to set up Docker on different operating systems is available on [Docker’s site][3].

To make sure Docker is installed properly, please run `docker` command from a terminal.

### Gradle
The last tool that is required is Gradle installed. I use [sdkman][4] to install Gradle for me.

### Spring Boot application
Besides, we of course need a Spring Boot application we want to dockerize. Of course we can create something very simple just for the purpose of this article.

Given we have all parts up and running, we can start with real work. First, we’re going to create Gradle build file.

## Creating Gradle build file

The Gradle build file could like the following.

```groovy
	buildscript {
		// ...
	    dependencies {
	        // ...
	        classpath('se.transmode.gradle:gradle-docker:1.2')
	    }
	}

	group = 'senco'

	apply plugin: 'docker'

	jar {
	    baseName = 'smog24'
	}

	// ...

	task buildDocker(type: Docker, dependsOn: build) {
	    push = true
	    applicationName = jar.baseName
	    dockerfile = file('src/main/docker/Dockerfile')
	    doFirst {
	        copy {
	            from jar
	            into stageDir
	        }
	    }
	}

```

The important parts are `gradle-docker` dependency, JAR `baseName` and of course the `buildDocker` task.

Since we mentioned `Dockerfile` in the build script, it’s time to create it.

## Creating Dockerfile

The file itself is pretty simple.

```
	FROM java:8
	VOLUME /tmp

	ADD smog24.jar app.jar
	RUN bash -c 'touch /app.jar'

	ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

```

So what is it all about?

* `FROM java:8` defines an image our app image will be based on. In this case it’s an image contains Java 8.
* `VOLUME /tmp` defines a directory where our application can write into
* `ADD smog24.jar app.jar` adds a JAR with our app to the final image and name it `app.jar`
* `RUN bash -c 'touch /app.jar'` updates the jar modification time
* `ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]`executes our jar, with `urandom` as Tomcat entropy source to [reduce Tomcat startup time][5]

Now that we have everything ready, we can build our image.

## Logging in to DockerHub

Since we want to make our image deployable to external hosts, like Amazon EC2, we want to publish our app image to [￼DockerHub￼][6].

First thing is to create an account there. Given we have our username and password, we can log in to the service:

```
	$ docker login
	Username: senco
	Password:
	Email: marcin@something.com
	WARNING: login credentials saved in /Users/marcin/.docker/config.json
	Login Succeeded
```

## Building and publishing Docker image

To build the image, we need to run the following command:

```
	$ gradle clean build buildDocker
```

The result is:

```
	:buildDocker
	Sending build context to Docker daemon  18.5 MB
	Step 1 : FROM java:8
	 ---> 736600fd4ae5
	Step 2 : VOLUME /tmp
	 ---> Using cache
	 ---> 11a7f1da28c1
	Step 3 : ADD smog24.jar app.jar
	 ---> a261a4f54700
	Removing intermediate container 0692c3f8de5b
	Step 4 : RUN bash -c 'touch /app.jar'
	 ---> Running in dd47828d028f
	 ---> 1ae49efd21a0
	Removing intermediate container dd47828d028f
	Step 5 : ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar /app.jar
	 ---> Running in ef3dde2e8e05
	 ---> 3e31536bfd75
	Removing intermediate container ef3dde2e8e05
	Successfully built 3e31536bfd75

	The push refers to a repository [docker.io/senco/smog24]

	//...

	BUILD SUCCESSFUL
```

Now, we can run our app with:

```
	$ docker run -p 8080:8080 -t senco/smog24
```

And verify it’s running:

```
	$ docker ps
	CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
	1328062c2da0        senco/smog24        "java -Djava.security"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   jolly_thompson

```

## Wrapping up

In this article, we learned how to dockerize Spring Boot application.

* We modified Gradle build script, so it supports Docker
* We wrote a new Dockerfile from scratch
* Then we authenticated to DockerHub and published our image there
* Finally, we run our app from the freshly created and published image

Now, the application is dockerized and ready to deploy live!

[1]:	https://hub.docker.com
[2]:	https://www.virtualbox.org
[3]:	https://docs.docker.com/engine/installation/#installation
[4]:	http://sdkman.io
[5]:	http://wiki.apache.org/tomcat/HowTo/FasterStartUp#Entropy_Source
[6]:	https://hub.docker.com
