---
draft: false
title: "Size matters"
description: "How to create lightweight images and save networking and storage transfer."
date: 2022-06-11T17:49:18-03:00
tags: ["best-practices", "docker"]
categories: ["stop-copy-paste"]
---
In the following post I am going to share with you some tips and best practices to develop our container images, 
as an example we are going to create an image for an app in Golang, but the following tips apply to any language, let's go!

## Preface

To make our container images as small as possible in terms of weight (megabytes, gigabytes, etc.) 
is not a matter of taste, it helps us in many ways, here are some of them:
- Reduce storage costs in the registry we use to manage our images.
- When we have to obtain the image to start the container it is clear that the lighter it is the faster the 
initialization of the container will be, and with this we win in two points.
  - Costs, and by costs we mean the use of the networking we use to obtain the image and then initialize the container.
  - Speed in auto scaling, it is clear that to obtain a 20 MB image versus a 900 MB image the first one, of course, 
  is going to initialize with a higher speed.

To give a few examples.

## Let's start

Let us imagine that we have the following Dockerfile to create our container image e.g:

```dockerfile
FROM golang:1.18
WORKDIR /build

COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . ./
RUN go build -o ./myapp ./path/to/main

ENTRYPOINT ["/myapp"]
```
Let's build our `docker build -t myapp:0.0.1 .` image.

If we list the images that we have in our host we will be able to observe that the weight is approximately `968 MB`.

What? 968 MB just to make available a binary that weighs a few megabytes?

>NOTE
> 
> In all my publications you will find concepts, the idea is that we learn and not copy and paste.
> 
> To give an example `RUN go build -o ./myapp ./path/to/main` where `./path/to/main` should be the main of 
> your Golang app

## Proposal/learning

### Let's go with the first proposal.

It is always a good practice to use `-alpine` images, by convention in the container universe when we make available 
an `-alpine` image we are indicating to the client that it is an image reduced in size and the one we should use in 
our Dockerfile, among other things.

Well, let's make a small change in our Dockerfile and build our image again

```dockerfile
FROM golang:1.18-alpine3.16
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . ./
RUN go build -o ./myapp ./path/to/main

ENTRYPOINT ["/myapp"]
```

If we pay attention the change was subtle, but effective, we went from `FROM golang:1.18` to 
`FROM golang:1.18-alpine3.16`.

Let's build our `docker build -t myapp:0.0.2 .` image again. 

If we list the images again we will find that now the `myapp:0.0.2` image weighs approximately `331 MB`.

We reduce, if the accounts do not fail, 637 MB.

It's an excellent approach, but let's rethink: do you have to have an image with all of Golang inside 
the container weighing about 331 MB to make available a binary that weighs a few megabytes?

The answer is clearly, no.

### Second proposal

The container technology has an excellent feature, that for our case, will help us to build a very light 
container image, in case you didn't know, I'm talking about Multistage, I share with you the 
[official documentation](https://docs.docker.com/develop/develop-images/multistage-build/) 
to deepen about this feature.

What does Multistage consist of? It is about building images in stages so that we can share data between each 
one of them, and we will obtain a final image of a very small size.

The first thing we are going to do is to have a first stage of build, where we are going to build the binary, 
and a second stage where we are going to make it available for use.

Let's get to work, let's open and make the following modifications to our Dockerfile.

```dockerfile
# First layer use to build a Golang binary
FROM golang:1.18-alpine3.16 AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . ./
RUN go build -o ./myapp ./path/to/main

# Final layer expose app to minimal docker image
FROM alpine:3.16.0

COPY --from=builder /build/myapp /myapp

ENTRYPOINT ["/myapp"]
```

As we can see, the first modification consists of tagging the first stage as a build.
Then in the second and final stage with the following line `COPY --from=builder /build/myapp /myapp` we copy 
the binary from the stage that we have selected as `builder` and we make it available in an alpine image.

If we list now our images we can see that it weighs approximately 9 MB, yes yes, 
I wrote correctly 9 megabytes :sunglasses:.

We could do one last optimization or best practice, but I think it's worth leaving it for another post.

So as not to bore you and for the moment let's take a break.

Have a good time! 👋🏽
