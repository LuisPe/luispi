---
draft: false
title: "When context matters"
description: "Small details can save us from major setbacks"
date: 2022-06-13T18:45:35-03:00
tags: ["best-practices", "docker"]
categories: ["stop-copy-paste"]
---
The following publication could be said to be a continuation of another one where we talk about how having 
light images helps us in many aspects, if you still could not read it here 
I leave [access](https://luispe.github.io/blog/posts/lightweight-container-image/).

Today we are going to make a small, but important improvement, and we are going to find out why we are doing it.

## Preface

As the last proposal in the post I shared above we stayed at this point:

```
# First layer use to build a Golang binary
FROM golang:1.18-alpine3.16 AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . ./
RUN GOOS=linux go build -o ./myapp ./path/to/main

# Final layer expose app to minimal docker image
FROM alpine:3.16.0

COPY --from=builder /build/myapp /myapp

ENTRYPOINT ["/myapp"]
```

If we have as a premise that container technology and its popularization with Docker is disruptive is largely
because of the benefits of being able to build in different places and not find surprises when we start 
the application that is contained in the container, I invite you to think for a few seconds/minutes or as long as we need.

Can an image enhancement be made for the Golang application?

The answer is yes, let's get to work!

## Proposal/learning

Golang has one feature that is really powerful, and I'm not talking about goroutines, 
and that is the great virtue of being able to cross-compile.

What is cross-compilation?, it is the feature of being able to compile from a host with a certain architecture 
and operating system (OS) the binary for another architecture or OS.

So to be a bit more specific we can from a host with OS = linux and architecture = amd64, 
compile a binary for OS = windows, architecture = 386 :fire:.

Let's imagine now that where we run the containers for our applications the computation is linux as 
OS and with amd64 architecture.

With this in mind let's make a small, but important improvement to our Dockerfile.

```
# First layer use to build a Golang binary
FROM golang:1.18-alpine3.16 AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify

ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

COPY . ./
RUN go build -o ./myapp ./path/to/main

# Final layer expose app to minimal docker image
FROM alpine:3.16.0

COPY --from=builder /build/myapp /myapp

ENTRYPOINT ["/myapp"]
```
Let's first analyze the change and why we are making it.
```
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64
```
`CGO_ENABLED=0` we deactivate CGO

`GOARCH=amd64` we indicate the architecture

`GOOS=linux` we indicate the SO

All good luispi, but what gain did we get?

Making sure to compile the application for the environment in which it will be executed will prevent several headaches
or troubleshooting, and needless to say that we no longer care where we are going to do it 
(whatever our continuous integration channel), because by compiling, again, for the environment 
in which it will be executed, we can rest assured that we are shortening the margin of mishaps.

So as not to bore you and for the moment let's take a break.

Have a good time! 👋🏽
