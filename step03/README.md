![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 03 - Harbor

Harbor is an open source cloud native registry that stores, signs, and scans container images for vulnerabilities.
Harbor solves common challenges by delivering trust, compliance, performance, and interoperability. It fills a gap for
organizations and applications that cannot use a public or cloud-based registry, or want a consistent experience across
clouds.

In this step we're going to push our ossproject image to the Harbor registry service.


### 1. Login to the Harbor registry with docker

The lab environment has a Harbor instance running at reg.rx-m.net. We're going to use it as a target for pushing and pulling images. In
the next step we will use these images with an AWS hosted Kubernetes cluster.

A repository has been created in the Harbor registry for us called "cndev" and we can login with the creds:

- kubecon/CNdev2019

Use Docker to login to the registry:

```
ubuntu@ip-172-31-30-5:~$ sudo docker login reg.rx-m.net

Username: kubecon
Password:
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

ubuntu@ip-172-31-30-5:~$
```

Good, now docker has authed with Harbor and we can push images.


### 2. Push your image

List the images on you system:

```
ubuntu@ip-172-31-30-5:~$ sudo docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ossp                latest              6af45da7b705        3 hours ago         12.4MB
<none>              <none>              d8a7447ed025        3 hours ago         609MB
<none>              <none>              d33d9f82416e        4 hours ago         393MB
<none>              <none>              b2847157dd59        4 hours ago         355MB
<none>              <none>              0e1526bc762b        4 hours ago         355MB
golang              1.12.5-alpine3.9    c7330979841b        10 days ago         350MB

ubuntu@ip-172-31-30-5:~$
```

Our ossp:latest image is the one we want to push but docker is picky about image names. They have 4 components:

- host - where to find the registry server to push/pull against
- account - account/group/organization/user to push/pull against
- repository - a collection of related images
- tag - the specific image within a repo

So the name we need is:  "reg.rx-m.net/cndev/wra.ossp:latest"

The "latest" tag is the default so we can leave that out. Note the repo name "wra.ossp". To avoid interacting with other
users in the session, use your initials and your lab machine IP to ensure you repo name is unique (wra.ossp is already
taken!). For example if you are Bob J. Smith on box 10.2.2.3 you should use the repo name: "bjs.10.2.2.3.ossp".

We can use docker tag to add another name to the ossp image. Try it:

```
ubuntu@ip-172-31-30-5:~$ sudo docker image tag ossp reg.rx-m.net/cndev/wra.ossp

ubuntu@ip-172-31-30-5:~$
```

Now push it!

```
ubuntu@ip-172-31-30-5:~$ sudo docker image push reg.rx-m.net/cndev/wra.ossp

The push refers to repository [reg.rx-m.net/cndev/wra.ossp]
089757526db1: Pushed
latest: digest: sha256:cc8ae1c9e75007ef3c63ac3b4ee3cdc8875f8aa34a3cef67bdf7b87a52c7b297 size: 528

ubuntu@ip-172-31-30-5:~$
```

Great our image is up on Harbor.


### 3. Pull your image

Next let's try pulling the image from containerd. First delete any images on the containerd side:

```
ubuntu@ip-172-31-30-5:~$ sudo ctr image ls

REF                           TYPE                                                 DIGEST                                                                  SIZE     PLATFORMS   LABELS
docker.io/library/ossp:latest application/vnd.oci.image.manifest.v1+json           sha256:490b962d255fe9f65f742651e80d740fcbb5e83e68badecc18fdef952eb59ebb 11.9 MiB linux/amd64 -
docker.io/rxmllc/hello:latest application/vnd.docker.distribution.manifest.v2+json sha256:0067dc15bd7573070d5590a0324c3b4e56c5238032023e962e38675443e6a7cb 57.0 MiB linux/amd64 -

ubuntu@ip-172-31-30-5:~$ sudo ctr image rm docker.io/library/ossp:latest

docker.io/library/ossp:latest

ubuntu@ip-172-31-30-5:~$ sudo ctr image rm docker.io/rxmllc/hello:latest

docker.io/rxmllc/hello:latest

ubuntu@ip-172-31-30-5:~$ sudo ctr image ls

REF TYPE DIGEST SIZE PLATFORMS LABELS

ubuntu@ip-172-31-30-5:~$
```

Now try to pull your service image with ctr:

```
ubuntu@ip-172-31-30-5:~$ sudo ctr images pull reg.rx-m.net/cndev/wra.ossp:latest
reg.rx-m.net/cndev/wra.ossp:latest:                                               resolved       |++++++++++++++++++++++++++++++++++++++|
manifest-sha256:cc8ae1c9e75007ef3c63ac3b4ee3cdc8875f8aa34a3cef67bdf7b87a52c7b297: done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:697acb5cab349fd38fa838d15ccfc6eff04331b08bc455a1ea3112fc118c5f5c:    done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:6af45da7b705f03d5504a62f24bddc26552e34835443b1c3facc033bf02e4c20:   done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 0.6 s                                                                    total:  5.9 Mi (9.9 MiB/s)
unpacking linux/amd64 sha256:cc8ae1c9e75007ef3c63ac3b4ee3cdc8875f8aa34a3cef67bdf7b87a52c7b297...
done

ubuntu@ip-172-31-30-5:~$
```

Super awesome, our image is live on the internet.


### 4. Check out the GUI

Lastly point a browser at https://reg.rx-m.net and again login with the creds: kubecon/CNdev2019. Poke around a bit but
do not change anything.

Ok, we are starting to get things pushed out to the cloud. Next step is to deploy the container in the cloud with
Kubernetes:  [../step04/README.md](../step04/README.md)


<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
