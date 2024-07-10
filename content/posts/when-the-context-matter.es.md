---
title: "Cuando el contexto importa"
description: "Pequeños detalles pueden evitarnos grandes contratiempos"
date: 2022-06-14T18:45:35-03:00
tags: ["best-practices", "docker"]
categories: ["stop-copy-paste"]
toc: true
---
La siguiente publicación se podría decir que es una continuación de otra donde hablamos de como tener imágenes livianas 
nos ayuda en muchos aspectos, si aún no pudiste leerla acá te dejo 
[el acceso](https://luispe.github.io/blog/posts/lightweight-container-image/).

Hoy vamos a realizar una pequeña, pero importante mejora, y vamos a descubrir porque la estamos realizando.

## Preámbulo
Como última propuesta en la publicación que compartí anteriormente nos quedamos en este punto:

```dockerfile {linenos=table,filename="Dockerfile"}
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

Si tenemos como premisa que la tecnología de containers y su popularización con Docker es disruptiva es en gran medida
por los beneficios de poder construir en diferentes lugares y no encontrarnos con sorpresas cuando iniciamos la aplicación
que está contenida en el container, te invito a que pensemos durante unos segundos/minutos o el tiempo que necesitemos.

¿Se puede realizar una mejora en la imagen para la aplicación de Golang?

La respuesta es si, ¡manos a la obra!

## Propuesta/aprendizaje
Golang posee una característica que es realmente poderosa, y no estoy hablando de las goroutines, y es la gran virtud de
poder realizar compilación cruzada.

¿Qué es compilación cruzada?, es la característica de poder compilar desde un host con una determinada arquitectura y
sistema operativo (SO) el binario para otra arquitectura o SO.

Entonces para ser un poco más específicos podemos desde un host con SO = linux y arquitectura = amd64, compilar un binario 
para SO = windows, arquitectura = 386 😲.

Imaginemos ahora que donde corremos los contenedores para nuestras aplicaciones el cómputo es linux como SO y con 
arquitectura amd64.

Con esto en mente realicemos una pequeña, pero importante mejora en nuestro Dockerfile.

```dockerfile {linenos=table,hl_lines=[7,8,9,10]filename="Dockerfile"}
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
Primero analicemos el cambio y porque lo realizamos.
```
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64
```
`CGO_ENABLED=0` desactivamos CGO

`GOARCH=amd64` indicamos la arquitectura

`GOOS=linux` indicamos el SO

Todo bien luispi, ¿pero qué ganancia obtuvimos?

Asegurarnos de compilar la aplicación para el entorno en el que va a ser ejecutado nos va a prevenir varios dolores de
cabeza o "troubleshooting", y de más esta decir que ya no nos importa donde vamos a hacerlo (cualquiera sea nuestro canal
de integración continua), porque realizando la compilación, de nuevo, para el entorno en que va a ser ejecutado, nos 
quedamos tranquilos de que estamos acortando el margen de contratiempos.

Para no aburrirte y por el momento hagamos una pausa.

¡Que pase bien! 👋🏽
