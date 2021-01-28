---
title: "Building and Running a Docker Container"
linkTitle: "Building and Running a Docker Container""
weight: 110
description: >
     Building and Running a Docker Container
---

# Building and Running a Docker Container

 - [Build a Docker Image](#build-a-docker-image)
 - [Running a Docker Container](#run-a-docker-container)
 - [Running a JAVA 11 Docker Container](#run-a-JAVA11-Docker-Container)
 
## Build a Docker Image

This section explains how to create a Docker image.

## Dockerfile

Docker build images by reading instructions from a _Dockerfile_. A _Dockerfile_ is a text document that contains all the commands a user could call on the command line to assemble an image. `docker image build` command uses this file and executes all the commands in succession to create an image.

`build` command is also passed a context that is used during image creation. This context can be a path on your local filesystem or a URL to a Git repository.

Dockerfile is usually called Dockerfile. The complete list of commands that can be specified in this file are explained at https://docs.docker.com/reference/builder/. The common commands are listed below:

## Common commands for Dockerfile

| Command | Purpose | Example |
:------------ | :-------------| :-------------|
| FROM | First non-comment instruction in _Dockerfile_ | `FROM ubuntu`
| COPY | Copies mulitple source files from the context to the file system of the container at the specified path | `COPY .bash_profile /home`
| ENV | Sets the environment variable | `ENV HOSTNAME=test`
| RUN | Executes a command | `RUN apt-get update`
| CMD | Defaults for an executing container | `CMD ["/bin/echo", "hello world"]`
| EXPOSE | Informs the network ports that the container will listen on | `EXPOSE 8093`
|==================

## Create your first image

Create a new directory `hellodocker`.

In that directory, create a new text file `Dockerfile`. Use the following contents:

```
FROM ubuntu:latest

CMD ["/bin/echo", "hello world"]
```

This image uses `ubuntu` as the base image. `CMD` command defines the command that needs to run. It provides a different entry point of `/bin/echo` and gives the argument "`hello world`".

## Build the image using the command:

```
  docker image build . -t helloworld
```

`.` in this command is the context for the command `docker image build`. `-t` adds a tag to the image.

The following output is shown:

```
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM ubuntu:latest
latest: Pulling from library/ubuntu
9fb6c798fa41: Pull complete 
3b61febd4aef: Pull complete 
9d99b9777eb0: Pull complete 
d010c8cf75d7: Pull complete 
7fac07fb303e: Pull complete 
Digest: sha256:31371c117d65387be2640b8254464102c36c4e23d2abe1f6f4667e47716483f1
Status: Downloaded newer image for ubuntu:latest
 ---> 2d696327ab2e
Step 2/2 : CMD /bin/echo hello world
 ---> Running in 9356a508590c
 ---> e61f88f3a0f7
Removing intermediate container 9356a508590c
Successfully built e61f88f3a0f7
Successfully tagged helloworld:latest
```

## List the images available using `docker image ls`:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
helloworld          latest              e61f88f3a0f7        3 minutes ago       122MB
ubuntu              latest              2d696327ab2e        4 days ago          122MB
```

Other images may be shown as well but we are interested in these two images for now.

Run the container using the command:

```
docker container run helloworld
```

to see the output:

```
hello world
```

If you do not see the expected output, check your Dockerfile that the content exactly matches as shown above. Build the image again and now run it.

Change the base image from `ubuntu` to `busybox` in `Dockerfile`. Build the image again:

```  
docker image build -t helloworld:2 .
```

and view the images using `docker image ls` command:


```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
helloworld          2                   7fbedda27c66        3 seconds ago       1.13MB
helloworld          latest              e61f88f3a0f7        5 minutes ago       122MB
ubuntu              latest              2d696327ab2e        4 days ago          122MB
busybox             latest              54511612f1c4        9 days ago          1.13MB
```

`helloworld:2` is the format that allows to specify the image name and assign a tag/version to it separated by `:`.

## Create your first image using Java

### Create a simple Java application

Please Note: If you are running OpenJDK 9, `mvn package` may fail with

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project helloworld: Compilation failure: Compilation failure:
[ERROR] Source option 1.5 is no longer supported. Use 1.6 or later.
[ERROR] Target option 1.5 is no longer supported. Use 1.6 or later.
```

because support for Java 5 http://openjdk.java.net/jeps/182[was dropped in JDK9].

You can add

```
  <properties>
    <maven.compiler.source>1.6</maven.compiler.source>
    <maven.compiler.target>1.6</maven.compiler.target>
  </properties>
```

to the generated `pom.xml` to target 1.6 instead. See also the link:chapters/ch03-build-image-java-9.adoc[Build a Docker Image for Java 9] chapter.


## Create a new Java project:

```
mvn archetype:generate -DgroupId=org.examples.java -DartifactId=helloworld -DinteractiveMode=false
```

## Build the project:

```
cd helloworld
mvn package
```

## Run the Java class:

```
java -cp target/helloworld-1.0-SNAPSHOT.jar org.examples.java.App
```

This shows the output:

```
Hello World!
```

Let's package this application as a Docker image.

## Java Docker image

Run the OpenJDK container in an interactive manner:

```
docker container run -it openjdk
```

This will open a terminal in the container. Check the version of Java:

```
root@8d0af9da5258:/# java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-8u141-b15-1~deb9u1-b15)
OpenJDK 64-Bit Server VM (build 25.141-b15, mixed mode)
```

A different JDK version may be shown in your case. 

Exit out of the container by typing `exit` in the container shell.

## Package and run Java application as Docker image

Create a new Dockerfile in `helloworld` directory and use the following content:

```
FROM openjdk:latest

COPY target/helloworld-1.0-SNAPSHOT.jar /usr/src/helloworld-1.0-SNAPSHOT.jar

CMD java -cp /usr/src/helloworld-1.0-SNAPSHOT.jar org.examples.java.App
```

## Build the image:

```
docker image build -t hello-java:latest .
```
Run the image:

```
docker container run hello-java:latest
```

This displays the output:

```
Hello World!
```

This shows the exactly same output that was printed when the Java class was invoked using Java CLI.

## Package and run Java Application using Docker Maven Plugin

[Docker Maven Plugin](https://github.com/fabric8io/docker-maven-plugin) allows you to manage Docker images and containers using Maven. It comes with predefined goals:

| S. No | Goal | Description |
:------------ | :-------------| :-------------|
| 1| `docker:build` | Build images | 
| 2 |`docker:start` | Create and start containers| 
| 3 |`docker:stop` | Stop and destroy containers| 
| 4 |`docker:push` | Push images to a registry| 
| 5 |`docker:remove` | Remove images from local docker host| 
| 6 | `docker:logs` | Show container logs | 
|==================




Complete set of goals are listed at https://github.com/fabric8io/docker-maven-plugin.

Clone the sample code from https://github.com/arun-gupta/docker-java-sample/.

Create the Docker image:

```
mvn -f docker-java-sample/pom.xml package -Pdocker
```

This will show an output like:

```
[INFO] Copying files to /Users/argu/workspaces/docker-java-sample/target/docker/hellojava/build/maven
[INFO] Building tar: /Users/argu/workspaces/docker-java-sample/target/docker/hellojava/tmp/docker-build.tar
[INFO] DOCKER> [hellojava:latest]: Created docker-build.tar in 87 milliseconds
[INFO] DOCKER> [hellojava:latest]: Built image sha256:6f815
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

The list of images can be checked using the command `docker image ls | grep hello-java`:

```
hello-java                            latest              ea64a9f5011e        5 seconds ago       643 MB
```

## Run the Docker container:

```
mvn -f docker-java-sample/pom.xml install -Pdocker
```

This will show an output like:

```
[INFO] DOCKER> [hellojava:latest]: Start container 30a08791eedb
30a087> Hello World!
[INFO] DOCKER> [hellojava:latest]: Waited on log out 'Hello World!' 510 ms
```

This is similar output when running the Java application using `java` CLI or the Docker container using `docker container run` command.

The container is running in the foreground. Use `Ctrl` + `C` to interrupt the container and return back to terminal.

Only one change was required in the project to enable Docker packaging and running. A Maven profile is added in `pom.xml`:

```
<profiles>
    <profile>
        <id>docker</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>io.fabric8</groupId>
                    <artifactId>docker-maven-plugin</artifactId>
                    <version>0.22.1</version>
                    <configuration>
                        <images>
                            <image>
                                <name>hello-java</name>
                                <build>
                                    <from>openjdk:latest</from>
                                    <assembly>
                                        <descriptorRef>artifact</descriptorRef>
                                    </assembly>
                                    <cmd>java -cp maven/${project.name}-${project.version}.jar org.examples.java.App</cmd>
                                </build>
                                <run>
                                    <wait>
                                        <log>Hello World!</log>
                                    </wait>
                                </run>
                            </image>
                        </images>
                    </configuration>
                    <executions>
                        <execution>
                            <id>docker:build</id>
                            <phase>package</phase>
                            <goals>
                                <goal>build</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>docker:start</id>
                            <phase>install</phase>
                            <goals>
                                <goal>start</goal>
                                <goal>logs</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

## Dockerfile Command Design Patterns

### Difference between CMD and ENTRYPOINT

*TL;DR* `CMD` will work for most of the cases.

Default entry point for a container is `/bin/sh`, the default shell.

Running a container as `docker container run -it ubuntu` uses that command and starts the default shell. The output is shown as:

```
docker container run -it ubuntu
root@88976ddee107:/#
```

`ENTRYPOINT` allows to override the entry point to some other command, and even customize it. For example, a container can be started as:

```
docker container run -it --entrypoint=/bin/cat ubuntu /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
. . .
```

This command overrides the entry point to the container to `/bin/cat`. The argument(s) passed to the CLI are used by the entry point.

## Difference between ADD and COPY

*TL;DR* `COPY` will work for most of the cases.

`ADD` has all capabilities of `COPY` and has the following additional features:

. Allows tar file auto-extraction in the image, for example, `ADD app.tar.gz /opt/var/myapp`.
. Allows files to be downloaded from a remote URL. However, the downloaded files will become part of the image. This causes the image size to bloat. So its recommended to use `curl` or `wget` to download the archive explicitly, extract, and remove the archive.

### Import and export images

Docker images can be saved using `image save` command to a `.tar` file:

```
docker image save helloworld > helloworld.tar
```
These tar files can then be imported using `load` command:

```
docker image load -i helloworld.tar
```

## Run a Docker Container

The first step in running an application using Docker is to run a container. If you can think of an open source software, there is a very high likelihood that there will be a Docker image available for it at https://store.docker.com[Docker Store]. Docker client can simply run the container by giving the image name. The client will check if the image already exists on Docker Host. If it exists then it'll run the container, otherwise the host will first download the image.

## Pull Image

Let's check if any images are available:

```
docker image ls
```

At first, this list is empty. If you've already downloaded the images as specified in the setup chapter, then all the images will be shown here. 

List of images can be seen again using the `docker image ls` command. This will show the following output:

```
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
hellojava                    latest              8d76bf5691c4        32 minutes ago      740MB
hello-java                   latest              93b1180c5d91        36 minutes ago      740MB
helloworld                   2                   7fbedda27c66        41 minutes ago      1.13MB
helloworld                   latest              e61f88f3a0f7        About an hour ago   122MB
mysql                        latest              b4e78b89bcf3        3 days ago          412MB
ubuntu                       latest              2d696327ab2e        4 days ago          122MB
jboss/wildfly                latest              9adbdb00cded        8 days ago          592MB
openjdk                      latest              6077adce18ea        8 days ago          740MB
busybox                      latest              54511612f1c4        9 days ago          1.13MB
tailtarget/hadoop            2.7.2               ee6b539c886e        6 months ago        1.15GB
tailtarget/jenkins           2.32.3              71a7d9bcfe2b        6 months ago        859MB
arungupta/couchbase          travel              7929a80707db        7 months ago        583MB
arungupta/couchbase-javaee   travel              2bb52abaad5f        7 months ago        595MB
arungupta/javaee7-hol        latest              da5c9d4f85ca        2 years ago         582MB
```

More details about the image can be obtained using `docker image history jboss/wildfly` command:

```
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9adbdb00cded        8 days ago          /bin/sh -c #(nop)  CMD ["/opt/jboss/wildfl...   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  EXPOSE 8080/tcp              0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  USER [jboss]                 0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV LAUNCH_JBOSS_IN_BAC...   0B                  
<missing>           8 days ago          /bin/sh -c cd $HOME     && curl -O https:/...   163MB               
<missing>           8 days ago          /bin/sh -c #(nop)  USER [root]                  0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV JBOSS_HOME=/opt/jbo...   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV WILDFLY_SHA1=9ee3c0...   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV WILDFLY_VERSION=10....   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/lib/...   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  USER [jboss]                 0B                  
<missing>           8 days ago          /bin/sh -c yum -y install java-1.8.0-openj...   204MB               
<missing>           8 days ago          /bin/sh -c #(nop)  USER [root]                  0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  MAINTAINER Marek Goldma...   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  USER [jboss]                 0B                  
<missing>           8 days ago          /bin/sh -c #(nop) WORKDIR /opt/jboss            0B                  
<missing>           8 days ago          /bin/sh -c groupadd -r jboss -g 1000 && us...   296kB               
<missing>           8 days ago          /bin/sh -c yum update -y && yum -y install...   28.7MB              
<missing>           8 days ago          /bin/sh -c #(nop)  MAINTAINER Marek Goldma...   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  LABEL name=CentOS Base ...   0B                  
<missing>           8 days ago          /bin/sh -c #(nop) ADD file:1ed4d1a29d09a63...   197MB               
```

## Run Container

### Interactively

Run WildFly container in an interactive mode.

```
docker container run -it jboss/wildfly
```

This will show the output as:

```
=========================================================================

  JBoss Bootstrap Environment

  JBOSS_HOME: /opt/jboss/wildfly

  JAVA: /usr/lib/jvm/java/bin/java

. . .

00:26:27,455 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
00:26:27,456 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
00:26:27,457 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 10.1.0.Final (WildFly Core 2.2.0.Final) started in 3796ms - Started 331 of 577 services (393 services are lazy, passive or on-demand)
```

This shows that the server started correctly, congratulations!

By default, Docker runs in the foreground. `-i` allows to interact with the STDIN and `-t` attach a TTY to the process. Switches can be combined together and used as `-it`.

Hit Ctrl+C to stop the container.

=== Detached container

Restart the container in detached mode:

```
docker container run -d jboss/wildfly
254418caddb1e260e8489f872f51af4422bc4801d17746967d9777f565714600
```

`-d`, instead of `-it`, runs the container in detached mode.

The output is the unique id assigned to the container. Logs of the container can be seen using the command `docker container logs <CONTAINER_ID>`, where `<CONTAINER_ID>` is the id of the container.

Status of the container can be checked using the `docker container ls` command:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
254418caddb1        jboss/wildfly       "/opt/jboss/wildfl..."   2 minutes ago       Up 2 minutes        8080/tcp            gifted_haibt
```

Also try `docker container ls -a` to see all the containers on this machine.

=== With default port

If you want the container to accept incoming connections, you will need to provide special options when invoking `docker run`. The container, we just started, can't be accessed by our browser. We need to stop it again and restart with different options.

```
docker container stop `docker container ps | grep wildfly | awk '{print $1}'`
```

Restart the container as:

```
docker container run -d -P --name wildfly jboss/wildfly
```

`-P` map any exposed ports inside the image to a random port on Docker host. In addition, `--name` option is used to give this container a name. This name can then later be used to get more details about the container or stop it. This can be verified using `docker container ls` command:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
89fbfbceeb56        jboss/wildfly       "/opt/jboss/wildfl..."   9 seconds ago       Up 8 seconds        0.0.0.0:32768->8080/tcp   wildfly
```

The port mapping is shown in the `PORTS` column. Access WildFly server at http://localhost:32768. Make sure to use the correct port number as shown in your case.

NOTE: Exact port number may be different in your case.

The page would look like:

![My Image)(wildfly-first-run-default-page.png)

## With specified port

Stop and remove the previously running container as:

```
docker container stop wildfly
docker container rm wildfly
```

Alternatively, `docker container rm -f wildfly` can be used to stop and remove the container in one command. Be careful with this command because `-f` uses `SIGKILL` to kill the container.

## Restart the container as:

```
docker container run -d -p 8080:8080 --name wildfly jboss/wildfly
```

The format is `-p hostPort:containerPort`. This option maps a port on the host to a port in the container. This allows us to access the container on the specified port on the host.

Now we're ready to test http://localhost:8080. This works with the exposed port, as expected.

Let's stop and remove the container as:

```
docker container stop wildfly
docker container rm wildfly
```

## Deploy a WAR file to application server

Now that your application server is running, lets see how to deploy a WAR file to it.

Create a new directory `hellojavaee`. Create a new text file and name it `Dockerfile`. Use the following contents:

```
FROM jboss/wildfly:latest

RUN curl -L https://github.com/javaee-samples/javaee7-simple-sample/releases/download/v1.10/javaee7-simple-sample-1.10.war -o /opt/jboss/wildfly/standalone/deployments/javaee-simple-sample.war
```

## Create an image:

```
docker image build -t javaee-sample .
```

Start the container:

```
docker container run -d -p 8080:8080 --name wildfly javaee-sample
```

## Access the endpoint:

```
curl http://localhost:8080/javaee-simple-sample/resources/persons
```

See the output:

```
<persons>
	<person>
		<name>
		Penny
		</name>
	</person>
	<person>
		<name>
		Leonard
		</name>
	</person>
	<person>
		<name>
		Sheldon
		</name>
	</person>
	<person>
		<name>
		Amy
		</name>
	</person>
	<person>
		<name>
		Howard
		</name>
	</person>
	<person>
		<name>
		Bernadette
		</name>
	</person>
	<person>
		<name>
		Raj
		</name>
	</person>
	<person>
		<name>
		Priya
		</name>
	</person>
</persons>
```

Optional: `brew install XML-Coreutils` will install XML formatting utility on Mac. This output can then be piped to `xml-fmt` to display a formatted result.

## Stop container

Stop a specific container by id or name:

```
docker container stop <CONTAINER ID>
docker container stop <NAME>
```

## Stop all running containers:

```
docker container stop $(docker container ps -q)
```

## Stop only the exited containers:

```
docker container ps -a -f "exited=-1"
```

## Remove container

Remove a specific container by id or name:

```
docker container rm <CONTAINER_ID>
docker container rm <NAME>
```

## Remove containers meeting a regular expression

```
docker container ps -a | grep wildfly | awk '{print $1}' | xargs docker container rm
```

## Remove all containers, without any criteria

```
docker container rm $(docker container ps -aq)
```

##  Additional ways to find port mapping

The exact mapped port can also be found using `docker port` command:

```
docker container port <CONTAINER_ID> or <NAME>
```

This shows the output as:

```
8080/tcp -> 0.0.0.0:8080
```

Port mapping can be also be found using `docker inspect` command:

```
docker container inspect --format='{{(index (index .NetworkSettings.Ports "8080/tcp") 0).HostPort}}' <CONTAINER ID>
```

## Build a Docker Image with JDK 9

This chapter explains how to create a Docker image with JDK 9.

The link:ch03-build-image.adoc[prior chapter] explained how, in general, to build a Docker image with Java.
This chapter expands on this topic and focuses on JDK 9 features.

## Create a Docker Image using JDK 9

Create a new directory, for example `docker-jdk9`.

In that directory, create a new text file `jdk-9-debian-slim.Dockerfile`.
Use the following contents:

```
# A JDK 9 with Debian slim
FROM debian:stable-slim
# Download from http://jdk.java.net/9/
# ADD http://download.java.net/java/GA/jdk9/9/binaries/openjdk-9_linux-x64_bin.tar.gz /opt
ADD openjdk-9_linux-x64_bin.tar.gz /opt
# Set up env variables
ENV JAVA_HOME=/opt/jdk-9
ENV PATH=$PATH:$JAVA_HOME/bin
CMD ["jshell", "-J-XX:+UnlockExperimentalVMOptions", \
               "-J-XX:+UseCGroupMemoryLimitForHeap", \
               "-R-XX:+UnlockExperimentalVMOptions", \
               "-R-XX:+UseCGroupMemoryLimitForHeap"]
```

This image uses `debian` slim as the base image and installs the OpenJDK build
of JDK for linux x64 (see the link:ch01-setup.adoc[setup section] for how to download this into the
current directory).

The image is configured by default to run `jshell` the Java REPL. Read more JShell at link:https://docs.oracle.com/javase/9/jshell/introduction-jshell.htm[Introduction to JShell]. The
experimental flag `-XX:+UseCGroupMemoryLimitForHeap` is passed to the REPL
process (the frontend Java process managing user input and the backend Java
process managing compilation).  This option will ensure container memory
constraints are honored.

## Build the image using the command:

```
docker image build -t jdk-9-debian-slim -f jdk-9-debian-slim.Dockerfile .
```


## List the images available using `docker image ls`:

```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
jdk-9-debian-slim       latest              023f6999d94a        4 hours ago         400MB
debian                  stable-slim         d30525fb4ed2        4 days ago          55.3MB
```

Other images may be shown as well but we are interested in these two images for
now.  The large difference in size is attributed to JDK 9, which is larger
in size than JDK 8 because it also explicitly provides Java modules that we
shall see more of later on in this chapter.

## Run the container using the command:

```
docker container run -m=200M -it --rm jdk-9-debian-slim
```

to see the output:

```
INFO: Created user preferences directory.
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro

jshell>
```

Query the available memory of the Java process by typing the following
expression into the Java REPL:

  Runtime.getRuntime().maxMemory() / (1 << 20)

to see the output:

```
jshell> Runtime.getRuntime().maxMemory() / (1 << 20)
$1 ==> 100
```

Notice that the Java process is honoring memory constraints (see the `--memory`
of `docker container run`) and will not allocate memory beyond that specified for the
container.

In a future release of the JDK it will no longer be necessary to specify an
experimental flag (`-XX:+UnlockExperimentalVMOptions`) once the mechanism by
which memory constraints are efficiently detected is stable.

JDK 9 supports the set CPUs constraint (see the `--cpuset-cpus` of
`docker container run`) but does not currently support other CPU constraints such as
CPU shares.  This is ongoing work http://openjdk.java.net/jeps/8182070[tracked]
in the OpenJDK project.

Note: the support for CPU sets and memory constraints have also been backported
to JDK 8 release 8u131 and above.

Type `Ctrl` + `D` to exit out of `jshell`.

To list all the Java modules distributed with JDK 9 run the following command:

```
docker container run -m=200M -it --rm jdk-9-debian-slim java --list-modules
```

This will show an output:

```
java.activation@9
java.base@9
java.compiler@9
java.corba@9
java.datatransfer@9
java.desktop@9
java.instrument@9
java.logging@9
java.management@9
java.management.rmi@9
java.naming@9
java.prefs@9
java.rmi@9
java.scripting@9
java.se@9
java.se.ee@9
java.security.jgss@9
java.security.sasl@9
java.smartcardio@9
java.sql@9
java.sql.rowset@9
java.transaction@9
java.xml@9
java.xml.bind@9
java.xml.crypto@9
java.xml.ws@9
java.xml.ws.annotation@9
jdk.accessibility@9
jdk.aot@9
jdk.attach@9
jdk.charsets@9
jdk.compiler@9
jdk.crypto.cryptoki@9
jdk.crypto.ec@9
jdk.dynalink@9
jdk.editpad@9
jdk.hotspot.agent@9
jdk.httpserver@9
jdk.incubator.httpclient@9
jdk.internal.ed@9
jdk.internal.jvmstat@9
jdk.internal.le@9
jdk.internal.opt@9
jdk.internal.vm.ci@9
jdk.internal.vm.compiler@9
jdk.jartool@9
jdk.javadoc@9
jdk.jcmd@9
jdk.jconsole@9
jdk.jdeps@9
jdk.jdi@9
jdk.jdwp.agent@9
jdk.jlink@9
jdk.jshell@9
jdk.jsobject@9
jdk.jstatd@9
jdk.localedata@9
jdk.management@9
jdk.management.agent@9
jdk.naming.dns@9
jdk.naming.rmi@9
jdk.net@9
jdk.pack@9
jdk.policytool@9
jdk.rmic@9
jdk.scripting.nashorn@9
jdk.scripting.nashorn.shell@9
jdk.sctp@9
jdk.security.auth@9
jdk.security.jgss@9
jdk.unsupported@9
jdk.xml.bind@9
jdk.xml.dom@9
jdk.xml.ws@9
jdk.zipfs@9
```

In total there should be 75 modules:

```
$ docker container run -m=200M -it --rm jdk-9-debian-slim java --list-modules | wc -l
      75
```

##  Create a Docker Image using JDK 9 and Alpine Linux

Instead of `debian` as the base image it is possible to use Alpine Linux
with an early access build of JDK 9 that is compatible with the muslc library
shipped with Alpine Linux.

Create a new text file `jdk-9-alpine.Dockerfile`.
Use the following contents:

```
# A JDK 9 with Alpine Linux
FROM alpine:3.6
# Add the musl-based JDK 9 distribution
RUN mkdir /opt
# Download from http://jdk.java.net/9/
# ADD http://download.java.net/java/jdk9-alpine/archive/181/binaries/jdk-9-ea+181_linux-x64-musl_bin.tar.gz
ADD jdk-9-ea+181_linux-x64-musl_bin.tar.gz /opt
# Set up env variables
ENV JAVA_HOME=/opt/jdk-9
ENV PATH=$PATH:$JAVA_HOME/bin
CMD ["jshell", "-J-XX:+UnlockExperimentalVMOptions", \
               "-J-XX:+UseCGroupMemoryLimitForHeap", \
               "-R-XX:+UnlockExperimentalVMOptions", \
               "-R-XX:+UseCGroupMemoryLimitForHeap"]
```

This image uses `alpine` 3.6 as the base image and installs the OpenJDK build
of JDK for Alpine Linux x64 (see the link:ch01-setup.adoc[Setup Environments]
chapter for how to download this into the current directory).

The image is configured in the same manner as for the `debian`-based image.

## Build the image using the command:

```
docker image build -t jdk-9-alpine -f jdk-9-alpine.Dockerfile .
```

## List the images available using `docker image ls`:

```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
jdk-9-debian-slim       latest              023f6999d94a        4 hours ago         400MB
jdk-9-alpine            latest              f5a57382f240        4 hours ago         356MB
debian                  stable-slim         d30525fb4ed2        4 days ago          55.3MB
alpine                  3.6                 7328f6f8b418        3 months ago        3.97MB
```

Notice the difference in image sizes.  Alpine Linux by design has been carefully
crafted to produce a minimal running OS image. A cost of such a design is
an alternative standard library https://www.musl-libc.org/[musl libc] that is
not compatible with the C standard library (libc).  As a result the JDK requires
modifications to run on Alpine Linux.  Such modifications have been proposed
by the OpenJDK http://openjdk.java.net/projects/portola/[Portola Project].


##  Create a Docker Image using JDK 9 and a Java application

Clone the GitHib project https://github.com/PaulSandoz/helloworld-java-9 that
contains a simple Java 9-based project:

```
git clone https://github.com/PaulSandoz/helloworld-java-9.git
```

(If you have a github account you may wish to fork it and then clone the fork
so you can make modifications.)

Enter the directory `helloworld-java-9` and build the project from within a
running Docker container with JDK 9 installed:

```
docker container run --volume $PWD:/helloworld-java-9 --workdir /helloworld-java-9 \
      -it --rm openjdk:9-jdk-slim \
      ./mvnw package
```

(If you have JDK 9 installed locally on the host system you can build directly
with `./mvnw package`.)

In this case we are using the `openjdk:9-jdk-slim` on Docker hub that has been
configured to work with SSL certificates so that the maven wrapper tool can
successfully download the maven tool.  This image is not produced or in anyway
endorsed by the OpenJDK project (unlike the JDK 9 distributions that were
previously required).  It is anticipated that future releases of the JDK from
the OpenJDK project will have root CA certificates (see issue
https://bugs.openjdk.java.net/browse/JDK-8189131[JDK-8189131])

To build Docker image for this application use the file `helloworld-jdk-9.Dockerfile` from the checked out repo to build your image. The contents of the file are shown below:

```
# Hello world application with JDK 9 and Debian slim
FROM jdk-9-debian-slim
COPY target/helloworld-1.0-SNAPSHOT.jar /opt/helloworld/helloworld-1.0-SNAPSHOT.jar
# Set up env variables
CMD java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap \
  -cp /opt/helloworld/helloworld-1.0-SNAPSHOT.jar org.examples.java.App
```

Build a Docker image containing the simple Java application based of the Docker
image `jdk-9-debian-slim`:

```
docker image build -t helloworld-jdk-9 -f helloworld-jdk-9.Dockerfile .
```

## List the images available using `docker image ls`:

```
REPOSITORY              TAG                 IMAGE ID            CREATED              SIZE
helloworld-jdk-9        latest              eb0539e9529a        19 seconds ago       400MB
jdk-9-debian-slim       latest              023f6999d94a        5 hours ago          400MB
jdk-9-alpine            latest              f5a57382f240        5 hours ago          356MB
openjdk                 9-jdk-slim          6dca67f4790e        3 days ago           372MB
debian                  stable-slim         d30525fb4ed2        4 days ago           55.3MB
alpine                  3.6                 7328f6f8b418        3 months ago         3.97MB
```

Notice how large the application image `helloworld-jdk-9`.

Run the `jdeps` tool to see what modules the application depends on:

```
docker container run -it --rm helloworld-jdk-9 jdeps --list-deps /opt/helloworld/helloworld-1.0-SNAPSHOT.jar
```

and observe that the application only depends on the `java.base` module.

## Reduce the size of a Docker Image using JDK 9 and a Java application

The Java application is extremely simple and as a result uses very little of the
functionality shipped with JDK 9 distribution, specifically the application
only depends on functionality present in the `java.base` module.  We can create
a custom Java runtime that only contains the `java.base` module and include
that in application Docker image.

Create a custom Java runtime that is small and only contains the `java.base`
module:

```
docker container run --rm \
      --volume $PWD:/out \
      jdk-9-debian-slim \
      jlink --module-path /opt/jdk-9/jmods \
        --verbose \
        --add-modules java.base \
        --compress 2 \
        --no-header-files \
        --output /out/target/openjdk-9-base_linux-x64
```

This command exists as `create-minimal-java-runtime.sh` script in the repo earlier checked out from link:https://github.com/PaulSandoz/helloworld-java-9[helloworld-java-9].

The JDK 9 tool `jlink` is used to create the custom Java runtime. Read more jlink in the https://docs.oracle.com/javase/9/tools/jlink.htm[Tools Reference]. The tool
is executed from with the container containing JDK 9 and directory where the
modules reside, `/opt/jdk-9/jmods`, is declared in the module path.  Only the
`java.base` module is selected.

The custom runtime is output to the `target` directory:

```
$ du -k target/openjdk-9-base_linux-x64/
24      target/openjdk-9-base_linux-x64//bin
12      target/openjdk-9-base_linux-x64//conf/security/policy/limited
8       target/openjdk-9-base_linux-x64//conf/security/policy/unlimited
24      target/openjdk-9-base_linux-x64//conf/security/policy
68      target/openjdk-9-base_linux-x64//conf/security
76      target/openjdk-9-base_linux-x64//conf
44      target/openjdk-9-base_linux-x64//legal/java.base
44      target/openjdk-9-base_linux-x64//legal
72      target/openjdk-9-base_linux-x64//lib/jli
16      target/openjdk-9-base_linux-x64//lib/security
19824   target/openjdk-9-base_linux-x64//lib/server
31656   target/openjdk-9-base_linux-x64//lib
31804   target/openjdk-9-base_linux-x64/
```

To build Docker image for this application use the file `helloworld-jdk-9-base.Dockerfile` from the checked out repo. The contents of the file are shown below:

```
# Hello world application with custom Java runtime with just the base module and Debian slim
FROM debian:stable-slim
COPY target/openjdk-9-base_linux-x64 /opt/jdk-9
COPY target/helloworld-1.0-SNAPSHOT.jar /opt/helloworld/helloworld-1.0-SNAPSHOT.jar
# Set up env variables
ENV JAVA_HOME=/opt/jdk-9
ENV PATH=$PATH:$JAVA_HOME/bin
CMD java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap \
  -cp /opt/helloworld/helloworld-1.0-SNAPSHOT.jar org.examples.java.App
```

Build a Docker image containing the simple Java application based of the Docker
image `debian:stable-slim`:

```
docker image build -t helloworld-jdk-9-base -f helloworld-jdk-9-base.Dockerfile .
```

## List the images available using `docker image ls`:

```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
helloworld-jdk-9-base   latest              7052483fdb77        24 seconds ago      87.7MB
helloworld-jdk9         latest              eb0539e9529a        17 minutes ago      400MB
jdk-9-debian-slim       latest              023f6999d94a        5 hours ago         400MB
jdk-9-alpine            latest              f5a57382f240        5 hours ago         356MB
openjdk                 9-jdk-slim          6dca67f4790e        3 days ago          372MB
debian                  stable-slim         d30525fb4ed2        4 days ago          55.3MB
alpine                  3.6                 7328f6f8b418        3 months ago        3.97MB
[source, text]
```

The `helloworld-jdk-9-base` is much smaller and could be reduced further if
Alpine Linux was used instead of Debian Slim.

A realistic application will depend on more JDK modules but it's still possible
to significantly reduce the Java runtime to only the required modules (for
example many applications will not require Corba or RMI nor the compiler tools).