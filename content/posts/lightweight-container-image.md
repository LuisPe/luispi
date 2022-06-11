+++
draft = false
date = 2022-06-11T17:49:18-03:00
title = "El tamaño importa"
description = "Como prevenir comportamientos inesperados y como ahorrar transferencia de networking y storage"
authors = ["luispi"]
tags = ["best-practices", "docker"]
categories = []
+++
En la siguiente publicación voy a compartirles algunos consejos y buenas prácticas para desarrollar nuestras imágenes
de container, como ejemplo vamos a crear una imagen para una app en Golang, pero los siguientes consejos aplican
para cualquier lenguaje, ¡vamos!

## Preámbulo

Perseguir que nuestras imágenes de container sean lo más reducida posible en cuanto a su peso (megabytes, gigabytes, etc)
no es una cuestión de gustos, nos ayuda en muchos aspectos, a continuación les comparto algunos:
- Reduce gatos de storage en el registry que utilizamos para gestionar nuestras imágenes.
- Cuando tengamos que obtener la imagen para iniciar el container queda claro que mientras más liviana sea más rápido
  va a ser la inicialización del container, y con esto ganamos en dos puntos.
  - Costos, y costos nos referimos al uso del networking que utilicemos para obtener la imagen y luego inicializar
    el container.
  - Velocidad en auto scaling, está claro que obtener una imagen de 20MB versus una de 900MB la primera, claro está, va a
    inicializarse con mayor velocidad.

Por dar algunos ejemplos.

## Comencemos

Imaginemos que tenemos el siguiente Dockerfile para crear nuestra imagen de container e.g:

```dockerfile
FROM golang:1.18
WORKDIR /build

COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . ./
RUN go build -o ./myapp ./path/to/main

ENTRYPOINT ["/myapp"]
```
Construyamos nuestra imagen `docker build -t myapp:0.0.1 .`

Si listamos las imágenes que tengamos en nuestro host vamos a poder observar que el peso es de aproximadamente `968 MB`

What? 968 MB solo para disponibilizar un binario que pesa unos pocos megas?

## Propuesta/aprendizaje

### Vamos con la primera propuesta.

Siempre es una buena práctica usar imágenes `-alpine`, por convención en el universo de container cuando disponibilizamos
una imagen `-alpine` estamos indicando al cliente que es una imagen reducida en tamaño y la que deberíamos utilizar en
nuestro Dockerfile, entre otras cosas.

Bien, realicemos un pequeño cambio en nuestro Dockerfile y volvamos a construir nuestra imagen

```dockerfile
FROM golang:1.18-alpine3.16
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . ./
RUN go build -o ./myapp ./path/to/main

ENTRYPOINT ["/myapp"]
```

Si prestamos atención el cambio fue sutil, pero efectivo, pasamos de `FROM golang:1.18` a `FROM golang:1.18-alpine3.16`

Si volvemos a listar las imágenes nos vamos a encontrar con que ahora nuestra imagen pesa aproximadamente `331 MB`

Reducimos, si las cuentas no fallan, 637 MB.

Es una excelente "approach" pero repensemos. ¿Hace falta tener una imagen de container de 331 MB para disponibilizar un 
binario que pesa unos cuantos megabytes?.

La respuesta es claramente, no.

### Segunda propuesta

La tecnología de container tiene una característica excelente, que para nuestro caso, nos va a ayudar a construir una
imagen de container muy liviana, por si no lo sabías, estoy hablando de Multistage, te comparto la
[documentación oficial](https://docs.docker.com/develop/develop-images/multistage-build/) para que profundices sobre esta
característica.

En que consiste Multistage?, se trata de construir imágenes por etapas pudiendo así compartir datos entre cada una de 
ellas y vamos a obtener una imagen final de un tamaño muy pequeño.

Lo primero que vamos a hacer es tener una primera etapa de build, donde vamos a construir el binario, y una segunda 
etapa donde vamos a dejarlo disponible para utilizarlo.

Manos a la obra, abramos y realicemos las siguientes modificaciones a nuestro Dockerfile.

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

Como podemos observar la primera modificación consiste en taguear la primera etapa como build.
Luego en la segunda y etapa final con la siguiente línea `COPY --from=builder /build/myapp /myapp` copiamos el binario
desde la etapa que tagueamos como `builder` y lo disponibilizamos en una imagen alpine.

Si listamos ahora nuestras imágenes podemos observar que pesa aproximadamente 9 MB, si si, escribi correctamente 
9 megabytes :sunglasses:.

Podríamos realizar una última optimización o buena práctica, pero creo que vale la pena dejarlo para otra publicación.

Para no aburrirte y por el momento hagamos una pausa.

¡Que pase bien!
