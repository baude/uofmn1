# Container Experience 1

All of these activities will be run rootless (unprivileged) with Podman. This activity requires the following dependencies:

* podman
* curl
* jq

## Simple nginx (httpd) container

Run an ngnix container with a random port assignment.  The port must be greater than 1024

```console
$ podman run -dt -p RANDOM_PORT:80 docker.io/library/nginx:latest
```

Note the ID returned by the command.

Use curl to verify the running nginx instance
```console
$ curl http://localhost:RANDOM_PORT
```
You should see a basic nginx response (raw HTML)


## Inspect a container image

You can inspect all types of container components.  Among others, these are the basic ones:
* images
* containers
* networks

Inspecting a container image can reveal:
* CPU architecture (Architecture)
* operating system (OS)
* default command if one exists (cmd)

```console
$ podman inspect docker.io/library/nginx:latest
```

Explore the inspection data.  Ask questions!

## Monitor running container

The `podman ps` commands describes the containers you have run.  If run just as `podman ps`, you will see a list
of running containers.  To see running and stopped containers, run `podman ps -a`.

```console
$ podman ps
```

Run another container that will simply execute a command and exit.

```console
$ podman run -it docker.io/library/alpine:latest true
```

Note: the `-it` switch means run this container with an interactive (i) terminal (t). Try running without the `-it`.

After the container runs, we should see a bigger different in `podman ps` and `podman ps -a`.

```console
$ podman ps -a
```

In the output from `podman ps -a`, notice is shows the command the container is running or ran.  If a container does not have a process, it will exit.

## Checkout the nginx process that are running

You can view the processes running in a container with `podman top`.

```console
$ podman top <CONTAINER_HASH>
```

You should see many nginx worker processes and one nginx main process.  Notice the main process of nginx is PID 1.  Why is that?

## Inspect a container

You can inspect containers and images for metadata information describing the container or image. Inspect your nginx container.


```console
$ podman inspect <CONTAINER_HASH>
```

Look throught the output and examine what is reported. Look at the *State* and *Network* outputs.  Ask questions about what things are.

If you want to see only the *State* of containers, you can add a format directive.

```console
$ podman inspect --format "{{.State}}" <CONTAINER_HASH>
```

Notice how the formatting is plain text.  The intent is usually for machine parsing and not human readibility.  A tool called `jq` can help.

```console
$ podman inspect <CONTAINER_HASH> | jq .[0].State
```

This shows the first inspected container's *State* stanza.


## "Pop"(exec) into a container

You can execute a command in a running using `podman exec`.  Much like `podman run`, passing `-it` gives you an interactive terminal.

Before we run anything in a container, lets get some information on the host operating system.

```console
$ cat /etc/os-release
```

Now lets "ask" the same question within the container namespace.

```console
$ podman exec -it <CONTAINER_HASH> cat /etc/os-release
```

Is this what you expected? Ask questions.

The *exec* command can also be something like `/bin/bash` (or `/bin/sh`).  Let's explore this.

```console
$ podman exec -it /bin/bash
```

You know have a shell "in" the container. Try running the command `whoami`?  Is the output what you expected?  Only root can install packages.  Will this work?

```console
root@<HASH>:/# apt-get update && apt-get install -y procps
root@<HASH>:/# ps -u
root@<HASH>:/# exit
```

Is this what you expected?  Why are processes from the system not listed here?


## Stop all of your containers

Stop the podman container

```console
$ podman stop -a
```


## Build a custom ngnix image locally

In this section, you will learn how to build a custom nginx image. Look at the `Containerfile` in the *simplenginxcustom* dir.  This uses the "stock" nginx image as a starting point. Then it copies a custom index file into the image.  And lastly, it installs `vim` inside the container image.  The rest of this section assumes you are inside the *simplenginxcustom* dir.

### Customize the index file

First, edit the index file to say whatever you wish.

### Build a new container image

The `build` verb in podman allows you to create a new container image in local storage. The two basic requirements are:

1. instructions on what to build (Containerfile/Dockerfile)
1. the context directory 

In the command below, -f are the instructions on what to build.  And the `.` defines the context directory (CWD).


```console
$ podman build -t localhost/mynginx -f Containerfile .
```

Note: If the `Containerfile` or `Dockerfile` is in the context directory and that context directory is your CWD, then you can ommit the -f
switch as it is assumed

### Run the new image

Now that the image build is complete, we can run the new image.  Here the `:latest` tag is added for completeness.

```console
$ podman run -dt -p <RANDOM_PORT:80> localhost/mynginx:latest
```

### Verify your index is in the image

To verify our image contains the custom nginx index file we made, we can use curl:

```console
$ curl http://localhost:<RANDOM_PORT>
```

You should see your custom file.

### Clean up
 
 To clean up container related content, simply issue:

```console
$ podman rmi -fa
```


## Challenges

Try these challenges and partner up to find solutions as needed.

1. If `podman run -it --rm docker.io/library/python:latest python -V` shows version 3.13 or 3.14, how could you use a container to test something with Python-3.10?  Prove with `python -V`

1. Using the stock nginx container image (docker.io/library/nginx:latest), run a container that mounts (hint: -v) 
 your custom index.html into the container.  Prove by curl'ing the container and seeing your file. (hint: man podman-run)

1. Using the provided content in the directory `hello` to generate an image that will output the asciiart below.
Hint: Containerfile/Dockerfile [primative verbs]([https://docs.docker.com/build/concepts/dockerfile/)

```
!... Hello Podman World ...!

         .--"--.           
       / -     - \         
      / (O)   (O) \        
   ~~~| -=(,Y,)=- |         
    .---. /`  \   |~~      
 ~/  o  o \~~~~.----. ~~   
  | =(X)= |~  / (O (O) \   
   ~~~~~~~  ~| =(Y_)=-  |   
  ~~~~    ~~~|   U      |~~ 

Project:   https://github.com/containers/podman
Website:   https://podman.io
Desktop:   https://podman-desktop.io
Documents: https://docs.podman.io
YouTube:   https://youtube.com/@Podman
X/Twitter: @Podman_io
Mastodon:  @Podman_io@fosstodon.org
```