# Landing of container images on Google Container Registry
1. [Google Cloud Builder](#gcb)
2. [Docker push](#dockerpush)
3. [Skopeo](#skopeo)


Google Cloud Platform (GCP) offers various services to build and host containers apps including a private container registry: Google Container Registry (GCR).
In case you want to deploy a cloud app on GCP, it may be a good idea to have a local container registry in GCP infrastructure.
In case you are relying on external registries, you can end up with a high latency in the image building/pulling processes.
GCR storesimages in a [Google Storage Bucket](https://cloud.google.com/storage/docs/key-terms#buckets).

In to push images in GCR, you can either rely on various methods including
* Pushing images in GCR with  Google [Cloud container Builder](https://cloud.google.com/container-builder/docs/quickstart-docker)
* Pushing images into GCR using  [docker push command](https://docs.docker.com/engine/reference/commandline/push/)
* Pusing images to GCR using [Skopeo client](https://github.com/projectatomic/skopeo)

In the following lines we will see how to push a basic docker images using both methods in GCR



![](https://github.com/nelvadas/gcloud-containerbuilder-tutorials/raw/master/gcrview.png "Images tags 1.0 and 1.1")

## GCloud remote docker build <a name="gcb"/>

Let's authenticate on gcloud and select the  devoxx-201614 project
```
$ gcloud config set project devoxx-201614
```




we can release the tag 1.0 for  this image in grc.io using 
 
```
$ git clone https://github.com/nelvadas/gcloud-containerbuilder-tutorials.git
$ cd gcloud-containerbuilder-tutorials/docker
$ gcloud container builds submit --tag gcr.io/devoxx-201614/gc-cb-hello:1.0
```
Run the previous command from your cloud shell terminal.
The command trigger a remote docker build+tag on GCP and store your image in a  [Google Storage Bucket](https://cloud.google.com/storage/docs/key-terms#buckets)
*gs://devoxx-201614_cloudbuild/* in this case

```
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


##  Local docker build + docker push in gcr.io  <a name="dockerpush"/>

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

Make sure your docker daemon can securely connect to GCR by installing  [GCR Docker credential addons](https://github.com/GoogleCloudPlatform/docker-credential-gcr) as described in the readme section.
Once installed, you can use the tradictionnal docker pull/push command with GCR urls.

```
$ docker push  gcr.io/devoxx-201614/gc-cb-hello:1.1
```

```
The push refers to repository [gcr.io/devoxx-201614/gc-cb-hello]
55af4a1634d5: Pushed 
352831fe6af7: Pushed 
cd7100a72410: Layer already exists 
```
 * devoxx-201614 represents the project id
 * gc-cb-hello the image name
 * 1.1 : image tag

## Using Skopeo <a name="skopeo"/>
you can also rely on [skopeo](https://github.com/projectatomic/skopeo) to inspect and copy images to GCR


```
$ skopeo inspect docker://gcr.io/devoxx-201614/gc-cb-hello:1.1
{
    "Name": "gcr.io/devoxx-201614/gc-cb-hello",
    "Digest": "sha256:25b82bbdcff30cc1fc36072058ed7c319e86674f4d05a04c35327235069200ee",
    "RepoTags": [
        "1.0",
        "1.1"
    ],
    "Created": "2018-05-17T15:04:02.2561802Z",
    "DockerVersion": "18.03.1-ce",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:ff3a5c916c92643ff77519ffa742d3ec61b7f591b6b7504599d95a4a41134e28",
        "sha256:b86749d6543d3c50e0aa92d7063166f6e368a2ba830a7fa7c3ac1b2bbd427c1d",
        "sha256:60e0bc099b38c8ad8b15687d2efbad90a5a9dcbd129fcc645c69ab69d6cea609"
    ]
}
```
 
Build a local gc-cb-hello:1.2 tag  
```

$ docker build -t  127.0.0.1:5000/gc-cb-hello:1.2 .
```

copy from various format: oci, tarball, copy the 1.2 to Google container registry using skopeo
```
$ skopeo  --insecure-policy copy   docker://gcr.io/devoxx-201614/gc-cb-hello:1.1 docker://gcr.io/devoxx-201614/gc-cb-hello:1.2
```

Skopeo relies on docker configuration previously set to inspect and copy images into your GCR.
```
Getting image source signatures
Skipping fetch of repeat blob sha256:ff3a5c916c92643ff77519ffa742d3ec61b7f591b6b7504599d95a4a41134e28
Skipping fetch of repeat blob sha256:b86749d6543d3c50e0aa92d7063166f6e368a2ba830a7fa7c3ac1b2bbd427c1d
Skipping fetch of repeat blob sha256:60e0bc099b38c8ad8b15687d2efbad90a5a9dcbd129fcc645c69ab69d6cea609
Copying config sha256:84ad7f553cccddee5feaf8edc94d9a8fe300bdc761084153677244bd3dbba811
 0 B / 1.99 KB [------------------------------------------------------------] 0s
Writing manifest to image destination
Storing signatures
```
WARNING: in your production systems, provide a correct [/etc/containers/policy.json](https://github.com/containers/image/blob/master/docs/policy.json.md) and remove --insecure-policy option.
the gc-cb-hello image  tags 1.0, 1.1 and 1.2 should now be available in Google Container Registry.

---


