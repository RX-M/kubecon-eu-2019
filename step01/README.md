![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 01 - gRPC

gRPC is a modern RPC system that allows interfaces to evolve gracefully, often without impacting existing clients, much
like a REST API. Because gRPC is based on HTTP/2 it is easy to integrate with common web based tools and infrastructure.
Perhaps one of the best things about gRPC is that it uses the powerful but concise Protocol Buffers IDL (interface
description language). IDLs make describing applications interfaces a pleasure. Oh, also it doesn't hurt to mention that
gRPC is fast, typically an order of magnitude or more faster than the equivalent REST API (your mileage may vary).


### 1. Install Go

We'll build our microservice in Go. Go is the language gRPC is written in, and though not required, writing gRPC
services in Go is convenient. Kubernetes and Docker are also written in Go and it is a generally popular language in the
container and microservices spaces.

We'll use Go 1.12, clone, extract and install the release:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ curl -sLO https://dl.google.com/go/go1.12.linux-amd64.tar.gz

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ tar zxf go1.12.linux-amd64.tar.gz

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ sudo mv ./go/ /usr/local/

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Add the Go bin to your path in your bashrc and bash_profile, then source it:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ echo "export PATH=/usr/local/go/bin:$PATH" >> ~/.bashrc

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ echo "[[ -r ~/.bashrc ]] && . ~/.bashrc" >> ~/.bash_profile

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ source ~/.bash_profile

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Test your Go installation:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ go version

go version go1.12 linux/amd64

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Great! Go is installed and ready to go (har)!


### 2. Install gRPC and Protobuf

Now install the Go gRPC libraries on your cloud based lab system (be patient, this can take a minute):

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ go get -u google.golang.org/grpc

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Next we'll install Protocol Buffers. The PB release is zipped so install unzip first:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ sudo apt install unzip

Reading package lists... Done
Building dependency tree
Reading state information... Done
Suggested packages:
  zip
The following NEW packages will be installed:
  unzip
0 upgraded, 1 newly installed, 0 to remove and 80 not upgraded.
Need to get 158 kB of archives.
After this operation, 530 kB of additional disk space will be used.
Get:1 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 unzip amd64 6.0-20ubuntu1 [158 kB]
Fetched 158 kB in 0s (1,262 kB/s)
Selecting previously unselected package unzip.
(Reading database ... 57482 files and directories currently installed.)
Preparing to unpack .../unzip_6.0-20ubuntu1_amd64.deb ...
Unpacking unzip (6.0-20ubuntu1) ...
Processing triggers for mime-support (3.59ubuntu1) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up unzip (6.0-20ubuntu1) ...

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Now download the PB release and unzip it:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ wget https://github.com/protocolbuffers/protobuf/releases/download/v3.7.1/protoc-3.7.1-linux-x86_64.zip

--2019-05-20 17:11:20--  https://github.com/protocolbuffers/protobuf/releases/download/v3.7.1/protoc-3.7.1-linux-x86_64.zip
Resolving github.com (github.com)... 192.30.253.112
Connecting to github.com (github.com)|192.30.253.112|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/23357588/a97d5c80-50a2-11e9-869c-ffc2e5e27052?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20190520%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190520T171121Z&X-Amz-Expires=300&X-Amz-Signature=5c08b3b21a17411ff5d4ae65c0a634d8ba6757b74d8f4c67b28187287ca0887b&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dprotoc-3.7.1-linux-x86_64.zip&response-content-type=application%2Foctet-stream [following]
--2019-05-20 17:11:21--  https://github-production-release-asset-2e65be.s3.amazonaws.com/23357588/a97d5c80-50a2-11e9-869c-ffc2e5e27052?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20190520%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190520T171121Z&X-Amz-Expires=300&X-Amz-Signature=5c08b3b21a17411ff5d4ae65c0a634d8ba6757b74d8f4c67b28187287ca0887b&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dprotoc-3.7.1-linux-x86_64.zip&response-content-type=application%2Foctet-stream
Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 52.217.1.228
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|52.217.1.228|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1529306 (1.5M) [application/octet-stream]
Saving to: ‘protoc-3.7.1-linux-x86_64.zip’

protoc-3.7.1-linux-x86_64.zip           100%[==============================================================================>]   1.46M  2.56MB/s    in 0.6s

2019-05-20 17:11:22 (2.56 MB/s) - ‘protoc-3.7.1-linux-x86_64.zip’ saved [1529306/1529306]

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ unzip protoc-3.7.1-linux-x86_64.zip

Archive:  protoc-3.7.1-linux-x86_64.zip
   creating: include/
   creating: include/google/
   creating: include/google/protobuf/
  inflating: include/google/protobuf/wrappers.proto
  inflating: include/google/protobuf/field_mask.proto
  inflating: include/google/protobuf/api.proto
  inflating: include/google/protobuf/struct.proto
  inflating: include/google/protobuf/descriptor.proto
  inflating: include/google/protobuf/timestamp.proto
   creating: include/google/protobuf/compiler/
  inflating: include/google/protobuf/compiler/plugin.proto
  inflating: include/google/protobuf/empty.proto
  inflating: include/google/protobuf/any.proto
  inflating: include/google/protobuf/source_context.proto
  inflating: include/google/protobuf/type.proto
  inflating: include/google/protobuf/duration.proto
   creating: bin/
  inflating: bin/protoc
  inflating: readme.txt

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Finally add the Go ProtoBuf plugin for gRPC:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ go get -u github.com/golang/protobuf/protoc-gen-go

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Alright, we're ready to use IDL to create an RPC API.


### 3. Create a service definition using Protocol Buffers IDL

To define a gRPC service we use protocol buffers (PB). PB service definitions include functions that take a message
and return a message. Messages are like structs or records in other languages and can be composed of various other
types. For this simple example we'll create a service definition that allows users to create Open Source Software
Project entries and retrieve them.

Here's the IDL we'll use:

```cpp
syntax = "proto3";

service OSSProject {
  rpc ListProjects (ProjectName) returns (ProjectTitles) {}
  rpc CreateProject (Project) returns (ProjectCreateStatus) {}
}

message ProjectName {
  string name = 1;
}
message ProjectTitles {
  repeated string name = 1;
  repeated string custodian = 2;
}
message ProjectCreateStatus {
  int32 status = 1;
}
message Project {
    string name = 1;
    string custodian = 2;
    string description = 3;
    int32 inceptionYear = 4;
}
```

Our gRPC service is named OSSProject and it has two methods, ListProjects and CreateProject. The methods make use of the
four messages defined later in the file. This file will be created with a .proto extension. Create the proto file:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ vim ossproject.proto

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ cat ossproject.proto

syntax = "proto3";

service OSSProject {
  rpc ListProjects (ProjectName) returns (ProjectTitles) {}
  rpc CreateProject (Project) returns (ProjectCreateStatus) {}
}

message ProjectName {
  string name = 1;
}
message ProjectTitles {
  repeated string name = 1;
  repeated string custodian = 2;
}
message ProjectCreateStatus {
  int32 status = 1;
}
message Project {
    string name = 1;
    string custodian = 2;
    string description = 3;
    int32 inceptionYear = 4;
}

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

The rpc methods defined in the service definition specify their request and response types. When a field is marked as
`repeated`, it is allowed to be repeated any number of times (0 or more), much like passing an array or list. The
messages we have defined make use of primitive types, though they need not.  The integer assignments at the end of each
line in the proto file designate the field ids of the messages. This allows the fields to be identified on the wire
without having to pass large field names.


### 4. Compile the IDL

Interface definitions in proto files are free of any implementation. To make use of them for network RPC we need to
generate code in some target language. We will use the protoc PB IDL compiler to generate Go code in this step.

Before we compile the IDL, export the path to the bin directory where the Go gRPC plugin was installed.  

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ export PATH=$PATH:~/go/bin

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Next make an output directory for the generated files:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ mkdir ossproject

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Finally use the protoc IDL compiler to compile the IDL:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ bin/protoc ossproject.proto --go_out=plugins=grpc:ossproject

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ ls -l ossproject

total 16
-rw-rw-r-- 1 ubuntu ubuntu 12453 May 20 17:27 ossproject.pb.go

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

The --go_out switch tells the protoc compiler which output language to use and where to put the generated files. It also
specifies the invocation of the grpc protobuf plugin which generates the rpc client/server stubs.


### 5. Code the service

Now we can quickly code up a go service. Here's the code:

```go
package main

import (
        "context"
        "log"
        "net"

        "google.golang.org/grpc"
        pb "./ossproject"
)

const (
        port = ":50088"
)

type server struct{}

func (s *server) ListProjects(ctx context.Context, in *pb.ProjectName) (*pb.ProjectTitles, error) {
        log.Printf("Received: %v", in.Name)
        names := []string{in.Name}
        custs := []string{"cncf"}
        return &pb.ProjectTitles{Name: names, Custodian: custs}, nil
}

func (s *server) CreateProject(ctx context.Context, in *pb.Project) (*pb.ProjectCreateStatus, error) {
        log.Printf("Received: %v", in.Name)
        return &pb.ProjectCreateStatus{Status: 0}, nil
}

func main() {
        lis, err := net.Listen("tcp", port)
        if err != nil {
                log.Fatalf("failed to listen: %v", err)
        }
        s := grpc.NewServer()
        pb.RegisterOSSProjectServer(s, &server{})
        if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
        }
}
```

Our service will listen on port 50088 on all interfaces. We use the pb generated RegisterOSSProjectServer to register
our service with the gRPC server. Once running clients can connect and make gRPC calls to either of the two methods.

Now build and run the service:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ vim server.go

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ cat server.go

package main

import (
        "context"
        "log"
        "net"

        "google.golang.org/grpc"
        pb "./ossproject"
)

const (
        port = ":50088"
)

type server struct{}

func (s *server) ListProjects(ctx context.Context, in *pb.ProjectName) (*pb.ProjectTitles, error) {
        log.Printf("Received: %v", in.Name)
        names := []string{in.Name}
        custs := []string{"cncf"}
        return &pb.ProjectTitles{Name: names, Custodian: custs}, nil
}

func (s *server) CreateProject(ctx context.Context, in *pb.Project) (*pb.ProjectCreateStatus, error) {
        log.Printf("Received: %v", in.Name)
        return &pb.ProjectCreateStatus{Status: 0}, nil
}

func main() {
        lis, err := net.Listen("tcp", port)
        if err != nil {
                log.Fatalf("failed to listen: %v", err)
        }
        s := grpc.NewServer()
        pb.RegisterOSSProjectServer(s, &server{})
        if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
        }
}

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ go run server.go


```

Great the server is up and running! Now let's test it with a Go client (when you are done you can stop the server with
^C).


### 6. Create a test client

Now we'll create a client to test the server, here's the code:

```go
package main

import (
        "context"
        "log"
        "os"
        "time"

        "google.golang.org/grpc"
        pb "./ossproject"
)

const (
        port = ":50088"
)

func main() {
        host := os.Args[1]
        req := os.Args[2]
        conn, err := grpc.Dial(host+port, grpc.WithInsecure())
        if err != nil {
                log.Fatalf("did not connect: %v", err)
        }
        defer conn.Close()
        c := pb.NewOSSProjectClient(conn)

        name := "fluentd"
        if len(os.Args) > 2 {
                name = req
        }
        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
        defer cancel()
        r, err := c.ListProjects(ctx, &pb.ProjectName{Name: name})
        if err != nil {
                log.Fatalf("could not get project: %v", err)
        }
        log.Printf("Projects: %v", r)
}
```

Create the client program:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ vim client.go

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ cat client.go

package main

import (
        "context"
        "log"
        "os"
        "time"

        "google.golang.org/grpc"
        pb "./ossproject"
)

const (
        port = ":50088"
)

func main() {
        host := os.Args[1]
        req := os.Args[2]
        conn, err := grpc.Dial(host+port, grpc.WithInsecure())
        if err != nil {
                log.Fatalf("did not connect: %v", err)
        }
        defer conn.Close()
        c := pb.NewOSSProjectClient(conn)

        name := "fluentd"
        if len(os.Args) > 2 {
                name = req
        }
        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
        defer cancel()
        r, err := c.ListProjects(ctx, &pb.ProjectName{Name: name})
        if err != nil {
                log.Fatalf("could not get project: %v", err)
        }
        log.Printf("Projects: %v", r)
}

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

This simple command line client takes the ip or hostname to connect to on the command line and the project to lookup.
Build and run it:

```
ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$ go run client.go localhost fluentd

2019/05/20 18:23:17 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Cool! Working gRPC!! Well, there's a lot more to learn about gRPC but let's move on to containerd and see how we can
run this RPC app in a container:  [../step02/README.md](../step02/README.md)


<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
