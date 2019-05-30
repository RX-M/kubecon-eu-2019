![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 02 - containerd

containerd has been a graduated top level project at the Cloud Native Computing Foundation since February 2019.
Developers working on containerd can use the ctr tool to quickly test features and functionality. containerd is written
in Go and exposes a gRPC API.

While containerd is a lean and efficient option for running containers, it is not an image build environment. To build
our application into a container image we'll use Docker. Installing Docker installs containerd because Docker uses
containerd as its low level container manager. containerd in turn uses the OCI standard runtime runC. You can install
containerd by itself but we'll install Docker and containerd together.

After we have used docker to build an image we'll use containerd directly to run and explore our container.


### 1. Install Docker

In your cloud instance, run the following command to install Docker and containerd:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ wget -qO- https://get.docker.com/ | sh

# Executing docker install script, commit: 2f4ae48
+ sudo -E sh -c apt-get update -qq >/dev/null
+ sudo -E sh -c apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
+ sudo -E sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | apt-key add -qq - >/dev/null
+ sudo -E sh -c echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable" > /etc/apt/sources.list.d/docker.list
+ sudo -E sh -c apt-get update -qq >/dev/null
+ [ -n  ]
+ sudo -E sh -c apt-get install -y -qq --no-install-recommends docker-ce >/dev/null
+ sudo -E sh -c docker version
Client:
 Version:           18.09.6
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        481bc77
 Built:             Sat May  4 02:35:27 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.6
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       481bc77
  Built:            Sat May  4 01:59:36 2019
  OS/Arch:          linux/amd64
  Experimental:     false
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker ubuntu

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

No need to execute any of the commands the install output suggests.


### 2. Run a Test Container

To test our container system we will run a simple container, the rx-m hello-world image. First get top level help by
running the ctr command with no arguments:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ ctr
NAME:
   ctr -
        __
  _____/ /______
 / ___/ __/ ___/
/ /__/ /_/ /
\___/\__/_/

containerd CLI


USAGE:
   ctr [global options] command [command options] [arguments...]

VERSION:
   1.2.5

COMMANDS:
     plugins, plugin           provides information about containerd plugins
     version                   print the client and server versions
     containers, c, container  manage containers
     content                   manage content
     events, event             display containerd events
     images, image, i          manage images
     leases                    manage leases
     namespaces, namespace     manage namespaces
     pprof                     provide golang pprof outputs for containerd
     run                       run a container
     snapshots, snapshot       manage snapshots
     tasks, t, task            manage tasks
     install                   install a new package
     shim                      interact with a shim directly
     cri                       interact with cri plugin
     help, h                   Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug                      enable debug output in logs
   --address value, -a value    address for containerd's GRPC server (default: "/run/containerd/containerd.sock")
   --timeout value              total timeout for ctr commands (default: 0s)
   --connect-timeout value      timeout for connecting to containerd (default: 0s)
   --namespace value, -n value  namespace to use with commands (default: "default") [$CONTAINERD_NAMESPACE]
   --help, -h                   show help
   --version, -v                print the version
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

We can use the `ctr image` command to pull the "rxmllc" org "hello" repository "latest" image.

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr image pull "docker.io/rxmllc/hello:latest"

docker.io/rxmllc/hello:latest:                                                    resolved       |++++++++++++++++++++++++++++++++++++++|
manifest-sha256:0067dc15bd7573070d5590a0324c3b4e56c5238032023e962e38675443e6a7cb: done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:5c9b4d01f86330ec6c9c7bc5e87d8b039cd5626861b1615f973647f2bfa18361:    done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:140ed4505e0d8418d29b0255cc7770a2c40c0fdd8828732812e86f10059ee55e:   done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:9fb6c798fa41e509b58bccc5c29654c3ff4648b608f5daa67c1aab6a7d02c118:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:3b61febd4aefe982e0cb9c696d415137384d1a01052b50a85aae46439e15e49a:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:9d99b9777eb02b8943c0e72d7a7baec5c782f8fd976825c9d3fb48b3101aacc2:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:d010c8cf75d7eb5d2504d5ffa0d19696e8d745a457dd8d28ec6dd41d3763617e:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:7fac07fb303e0589b9c23e6f49d5dc1ff9d6f3c8c88cabe768b430bdb47f03a9:    done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 2.7 s                                                                    total:  57.0 M (21.1 MiB/s)
unpacking linux/amd64 sha256:0067dc15bd7573070d5590a0324c3b4e56c5238032023e962e38675443e6a7cb...
done

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

If you know docker, you will see that ctr is similar but different. Run the image you just pulled:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr run docker.io/rxmllc/hello:latest hi

 _________________________________
/ RX-M - Cloud Native Consulting! \
\ rx-m.com                        /
 ---------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now try listing the images and containers on your system:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr image ls

REF                           TYPE                                                 DIGEST                                                                  SIZE     PLATFORMS   LABELS
docker.io/rxmllc/hello:latest application/vnd.docker.distribution.manifest.v2+json sha256:0067dc15bd7573070d5590a0324c3b4e56c5238032023e962e38675443e6a7cb 57.0 MiB linux/amd64 -

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr container ls

CONTAINER    IMAGE                            RUNTIME
hi           docker.io/rxmllc/hello:latest    io.containerd.runtime.v1.linux

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now remove your completed container:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr container rm hi

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr container ls

CONTAINER    IMAGE    RUNTIME

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```


### 3. Create a Dockerfile for the Step01 Microservice

Dockerfiles are usually placed at the root of a folder hierarchy containing everything you need to build one or more
Docker images. This folder is called the build context. The entire build context will be packaged and sent to the Docker
Engine for building. In simple cases the build context will have nothing in it but a Dockerfile.

We'll use the kubecon dir as our context. Dockerfiles give you repeatable, predictable builds by using an isolated,
repeatable container environment for the build process. Create a Dockerfile with the build steps needed to recreate your
Go service:

```
FROM golang:1.12.5-alpine3.9 as server-build
WORKDIR /go/src/app
COPY . .
RUN apk add --no-cache git
RUN go get -u google.golang.org/grpc
RUN go get -u github.com/golang/protobuf/protoc-gen-go
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo server.go

# Package the binary in its own isolated container
FROM scratch
COPY --from=server-build /go/src/app/server /server
EXPOSE 50088
ENTRYPOINT [ "/server" ]
```

This type of Dockerfile is known as a multistage build. The first stage begins with `FROM golang...` and performs the
Go build of our source files into an single static executable. Because the resulting executable is statically linked,
it has no external dependencies and can be copied into a second container image by itself with no other files, to
execute standalone. The second stage of the docker build begins with `FROM scratch` which means the container image is
not based on a preexisting image.

Create the Dockerfile for your service as per below:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ vim Dockerfile

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat Dockerfile

FROM golang:1.12.5-alpine3.9 as server-build
WORKDIR /go/src/app
COPY . .
RUN apk add --no-cache git
RUN go get -u google.golang.org/grpc
RUN go get -u github.com/golang/protobuf/protoc-gen-go
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo server.go

# Package the binary in its own isolated container
FROM scratch
COPY --from=server-build /go/src/app/server /server
EXPOSE 50088
ENTRYPOINT [ "/server" ]

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now build your Docker image:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo docker build . -t ossp

Sending build context to Docker daemon  5.103MB
Step 1/11 : FROM golang:1.12.5-alpine3.9 as server-build
 ---> c7330979841b
Step 2/11 : WORKDIR /go/src/app
 ---> Using cache
 ---> 68103c47304c
Step 3/11 : COPY . .
 ---> ee8579408b53
Step 4/11 : RUN apk add --no-cache git
 ---> Running in 7923fab7e451
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/6) Installing nghttp2-libs (1.35.1-r0)
(2/6) Installing libssh2 (1.8.2-r0)
(3/6) Installing libcurl (7.64.0-r1)
(4/6) Installing expat (2.2.6-r0)
(5/6) Installing pcre2 (10.32-r1)
(6/6) Installing git (2.20.1-r0)
Executing busybox-1.29.3-r10.trigger
OK: 20 MiB in 21 packages
Removing intermediate container 7923fab7e451
 ---> 1f53af5a124b
Step 5/11 : RUN go get -u google.golang.org/grpc
 ---> Running in 9f2cf840b0db
Removing intermediate container 9f2cf840b0db
 ---> 4b87a5bcd213
Step 6/11 : RUN go get -u github.com/golang/protobuf/protoc-gen-go
 ---> Running in 74006f8cc118
Removing intermediate container 74006f8cc118
 ---> f3e83a74cdd2
Step 7/11 : RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo server.go
 ---> Running in c4e471b2fa87
Removing intermediate container c4e471b2fa87
 ---> d8a7447ed025
Step 8/11 : FROM scratch
 --->
Step 9/11 : COPY --from=server-build /go/src/app/server /server
 ---> 761b996ea020
Step 10/11 : EXPOSE 50088
 ---> Running in ccd597153f0e
Removing intermediate container ccd597153f0e
 ---> 1b1bf438477b
Step 11/11 : ENTRYPOINT [ "/server" ]
 ---> Running in a8984e043dd1
Removing intermediate container a8984e043dd1
 ---> 6af45da7b705
Successfully built 6af45da7b705
Successfully tagged ossp:latest

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

This will take some time, as the build completes installations and compilations.

Now lets test the service!


### 4. Run the Containerized Service

Docker and containerd share many things but not container images. containerd supports namespaces allowing multiple
independent tenants on top of containerd. We can however easily export our Docker build to containerd. Try it:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo docker image save ossp > ossp.image

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr image import ossp.image

unpacking docker.io/library/ossp:latest (sha256:490b962d255fe9f65f742651e80d740fcbb5e83e68badecc18fdef952eb59ebb)...done

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now use ctr to run your service in its new container:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr image ls

REF                           TYPE                                                 DIGEST                                                                  SIZE     PLATFORMS   LABELS
docker.io/library/ossp:latest application/vnd.oci.image.manifest.v1+json           sha256:490b962d255fe9f65f742651e80d740fcbb5e83e68badecc18fdef952eb59ebb 11.9 MiB linux/amd64 -
docker.io/rxmllc/hello:latest application/vnd.docker.distribution.manifest.v2+json sha256:0067dc15bd7573070d5590a0324c3b4e56c5238032023e962e38675443e6a7cb 57.0 MiB linux/amd64 -

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo ctr run --net-host docker.io/library/ossp:latest ossproject


```

With the server running, from another shell run the client:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ go run client.go localhost fluentd

2019/05/20 21:34:06 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Containerized service up and running! Now we need to push this image up to a registry so that it can be used in CI/CD
and other systems:  [../step03/README.md](../step03/README.md)


<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
