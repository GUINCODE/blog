---
title: "Understand Docker layers by example : RUN instructions Impact"
seoTitle: "Docker layers"
seoDescription: "Understand Docker layers by example : RUN instructions Impact"
datePublished: Tue Dec 12 2023 09:14:48 GMT+0000 (Coordinated Universal Time)
cuid: clq24mj1k000208jo3bxs32pj
slug: understand-docker-layers-by-example-run-instructions-impact
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1702372797530/dcaebe38-50eb-438e-ace7-3fe49ae96bca.png
tags: docker, kubernetes, cloud-native, docker-images

---

I recently discussed with several people about how to build a Container and, more specifically, why it's important to minimize the usage of RUN instructions in a Dockerfile.

In this article, we will explore together the impact of unnecessarily multiplying multiple RUN instructions, as well as how to inspect the different layers of your images.

## Dockerfile

We will take the simplest possible Dockerfile for this article and compare two configurations:

A simple Dockerfile with only one RUN instruction: Dockerfile.1run

```bash
# Utilisation d'une image de base
FROM ubuntu:latest
# Create and delete secret file 
RUN echo "secret_content" > /AAA && rm /AAA
```

and another with 2 RUN instructions: Dockerfile.2run.

```bash
FROM ubuntu:latest
# Create and delete secret file 
RUN echo "secret_content" > /AAA 
RUN rm /AAA
```

The result may seem exactly the same but not really.

Each time you put a RUN instruction, Docker will create a new layer in the image with the state of the file system for each layer, which can sometimes pose a risk if it is also combined with other bad practices.

Let's look at this together and build the 2 images:

```bash
docker build -t 1run -f Dockerfile.1run 
docker build -t 2run -f Dockerfile.2run
```

Now let's analyze the image containing 2 instructions with the dive tool.

```bash
dive 2run
```

Surprisingly, we have 3 layers in this image:

* **1 for the base image**
    
* **1 for our RUN instruction that creates the file**
    
* **1 for our RUN instruction that deletes the file**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702370273325/9d5ead8c-7980-41b0-bfa3-466c1d22e4f8.png align="left")

On the right, we can observe that the file is present in this layer.

Let's see how we can manage to retrieve it.

## Retrieve files from docker layers

We have kept aside the ID of the layer, which is found in the layer details section.

Thanks to the docker command, we have the ability to easily extract the file system of the image.

```bash
docker save 2run -o test.tar
tar -xvf test.tar 

$ tar -xvf test.tar 

33570c3886dd27db6e7b8bd88205951f82a6a8048fd9b9292c5f556180dc894e/
33570c3886dd27db6e7b8bd88205951f82a6a8048fd9b9292c5f556180dc894e/VERSION
33570c3886dd27db6e7b8bd88205951f82a6a8048fd9b9292c5f556180dc894e/json
33570c3886dd27db6e7b8bd88205951f82a6a8048fd9b9292c5f556180dc894e/layer.tar
5a00f72ab3686152f0cf5067e06ff9937877cf31138ef4b8749a184c46b09898/
5a00f72ab3686152f0cf5067e06ff9937877cf31138ef4b8749a184c46b09898/VERSION
5a00f72ab3686152f0cf5067e06ff9937877cf31138ef4b8749a184c46b09898/json
5a00f72ab3686152f0cf5067e06ff9937877cf31138ef4b8749a184c46b09898/layer.tar
a23855d9376693f12ed1a1e68dd307dfd384615e4b1860a87978c5f9a7969120.json
d10815363623202ac07b49763f180d45a6941ef6a0ea905a92a7ddfa6bb45422/
d10815363623202ac07b49763f180d45a6941ef6a0ea905a92a7ddfa6bb45422/VERSION
d10815363623202ac07b49763f180d45a6941ef6a0ea905a92a7ddfa6bb45422/json
d10815363623202ac07b49763f180d45a6941ef6a0ea905a92a7ddfa6bb45422/layer.tar
manifest.json
repositories
```

During the `docker save` command, docker will extract all the layers of the image and create a .tar file for each layer.

All that's left is to extract the right layer.

In our case, the ID is:

`d10815363623202ac07b49763f180d45a6941ef6a0ea905a92a7ddfa6bb45422`

```bash
$ cd d10815363623202ac07b49763f180d45a6941ef6a0ea905a92a7ddfa6bb45422/
$ ls 
json  layer.tar  VERSION
$ tar -xvf layer.tar 
AAA
$ cat AAA 
secret_content
```

Nous avons pu retrouver le contenu du fichier AAA.

## Reduce numbers of RUN instructions

Let's test our image with a single instruction combining the two instructions.

```bash
dive 1run
```

We can see on the left 2 layers:

* 1 for the base image
    
* 1 for our RUN instruction (create + delete)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702370057239/d0412ec8-0384-40e8-a476-b3d303264a2d.png align="center")

Let's follow the same process with the ID corresponding to the ID of the layer of our RUN instruction `cf388386fad4ed6625a7eb438b9bdd915ab52e7be0d0a76a97f1cb4aa0ec40a5`:

```bash
docker save 1run -o test.tar 
tar -xvf test.tar 
cd cf388386fad4ed6625a7eb438b9bdd915ab52e7be0d0a76a97f1cb4aa0ec40a5
$ cd cf388386fad4ed6625a7eb438b9bdd915ab52e7be0d0a76a97f1cb4aa0ec40a5
$ ls 
json  layer.tar  VERSION
$ tar -xvf layer.tar
$ ls 
json  layer.tar  VERSION
```

It is not possible to find this file in this image.

## Advice

Minimizing the number of RUN instructions in a Dockerfile is crucial for reducing the size of the final image, improving build and deployment performance, and enhancing security by avoiding the creation of unnecessary layers that might contain sensitive or redundant data. A best practice is to combine related commands into a single RUN instruction, thus reducing the number of layers and optimizing the image's efficiency.