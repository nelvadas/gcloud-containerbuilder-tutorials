
# Build the container image and push it to GCR

GCP offers various services to build and host containers apps including a private container registry GCR.
In case you want to deploy a k8s cluster on GCP, it may be a good idea to have a local container registry in GCP infrastructure.
In case you are relying on external registries, you can end up with a high latency when pulling images
GCR store users images in a Google Storage Bucket.

In to push images in GCR, you can either rely on various methods including
* Pushing images in GCR with  Google Cloud container Builder
* Pushing images into GCR using  docker push command.
* Pusing images to GCR using Skopeo client

In the following lines we will see how to push a basic docker images using both methods in GCR

[Cloud Container Builder] (https://cloud.google.com/container-builder/docs/quickstart-docker) 


![](https://github.com/nelvadas/gcloud-containerbuilder-tutorials/raw/master/gcrview.png "Images tags 1.0 and 1.1")

## Option1: gcloud remote docker build

```
$ gcloud container builds submit --tag gcr.io/devoxx-201614/gc-cb-hello:1.0

Creating temporary tarball archive of 2 file(s) totalling 137 bytes before compression.
Uploading tarball of [.] to [gs://devoxx-201614_cloudbuild/source/1526591541.15-75cfe5f14b4d4db488e557c9225b5f0b.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/devoxx-201614/builds/c36d6c03-70d4-49b3-81f8-9a8e06a8abef].
Logs are available at [https://console.cloud.google.com/gcr/builds/c36d6c03-70d4-49b3-81f8-9a8e06a8abef?project=1031916757118].
------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------
starting build "c36d6c03-70d4-49b3-81f8-9a8e06a8abef"

FETCHSOURCE
Fetching storage object: gs://devoxx-201614_cloudbuild/source/1526591541.15-75cfe5f14b4d4db488e557c9225b5f0b.tgz#1526591543200702
Copying gs://devoxx-201614_cloudbuild/source/1526591541.15-75cfe5f14b4d4db488e557c9225b5f0b.tgz#1526591543200702...
/ [1 files][  278.0 B/  278.0 B]                                                
Operation completed over 1 objects/278.0 B.                                      
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM alpine
latest: Pulling from library/alpine
Digest: sha256:8c03bb07a531c53ad7d0f6e7041b64d81f99c6e493cb39abba56d956b40eacbc
Status: Downloaded newer image for alpine:latest
 ---> 3fd9065eaf02
Step 2/4 : ADD quickstart.sh /
 ---> ddd31940776b
Step 3/4 : RUN  chmod +x /quickstart.sh
 ---> Running in 5a04fa8f7d3d
Removing intermediate container 5a04fa8f7d3d
 ---> 8ab0dae18534
Step 4/4 : CMD ["/quickstart.sh"]
 ---> Running in 5a5b0e6e94f9
Removing intermediate container 5a5b0e6e94f9
 ---> 661b3612974a
Successfully built 661b3612974a
Successfully tagged gcr.io/devoxx-201614/gc-cb-hello:1.0
PUSH
Pushing gcr.io/devoxx-201614/gc-cb-hello:1.0
The push refers to repository [gcr.io/devoxx-201614/gc-cb-hello]
c27eece03e0d: Preparing
bb1a7010f114: Preparing
cd7100a72410: Preparing
cd7100a72410: Layer already exists
c27eece03e0d: Pushed
bb1a7010f114: Pushed
1.0: digest: sha256:45e1ec513a84d235e2d82eecc8dfa2cff604f3aa08a92797c77ca281010c6c67 size: 942
DONE
------------------------------------------------------------------------------------------------------------------------

ID                                    CREATE_TIME                DURATION  SOURCE                                                                                   IMAGES                                STATUS
c36d6c03-70d4-49b3-81f8-9a8e06a8abef  2018-05-17T21:12:24+00:00  11S       gs://devoxx-201614_cloudbuild/source/1526591541.15-75cfe5f14b4d4db488e557c9225b5f0b.tgz  gcr.io/devoxx-201614/gc-cb-hello:1.0  SUCCESS

```


##  Local docker build + docker push in gcr.io

### Build from Dockerfile on your localhost
```
$ cd docker
$ docker build -t  gcr.io/devoxx-201614/gc-cb-hello:1.1 .
Sending build context to Docker daemon  132.6kB
Step 1/4 : FROM alpine
 ---> 3fd9065eaf02
Step 2/4 : ADD  quickstart.sh /
 ---> Using cache
 ---> 9df091e06b25
Step 3/4 : RUN  chmod +x /quickstart.sh
 ---> Using cache
 ---> c62880f072f8
Step 4/4 : CMD ["/quickstart.sh"]
 ---> Using cache
 ---> 84ad7f553ccc
Successfully built 84ad7f553ccc
Successfully tagged gcr.io/devoxx-201614/gc-cb-hello:1.1

```


### Run the container locally 
```
$ docker run  gcr.io/devoxx-201614/gc-cb-hello:1.1
Hello, world! The time is Thu May 17 15:26:27 UTC 2018.
```

### Push the image in GCR.IO

Make sure your docker daemon can connect to GCR by installing  [GCR Docker credential addons](https://github.com/GoogleCloudPlatform/docker-credential-gcr)

```
$ docker push  gcr.io/devoxx-201614/gc-cb-hello:1.1
The push refers to repository [gcr.io/devoxx-201614/gc-cb-hello]
55af4a1634d5: Pushed 
352831fe6af7: Pushed 
cd7100a72410: Layer already exists 
```


## Using Skopeo
==================== In progress ======================




Your image tag 1.0 and 1.1 should now be available in Google Container Registry.



