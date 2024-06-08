---
draft: false
title: "Interfaces y \"copy paste\""
description: "Prevenir el abuso o mal uso de las interfaces favorece a la mantenibilidad del código."
date: 2022-06-20T10:47:02-03:00
tags: ["best-practices", "golang", "go"]
categories: ["stop-copy-paste"]
---
En la siguiente publicación quiero compartir un error común que solía cometer en mis proyectos en Golang. Y como bien
dice el título y descripción hoy vamos a hablar de las interfaces.

Seguramente se encuentren con algunas referencias a otra publicación que hice sobre 
[arquitectura hexagonal](https://luispe.github.io/blog/posts/hexagonal-architecture/), creo que parte del mal uso de las
interfaces es la consecuencia de seguir a rajatabla publicaciones de "medium" y no tomarse el tiempo para entender el 
concepto subyacente. 

## Preámbulo

Si hay algo que tenemos que admitir en el ecosistema de Golang es el enrosque que a veces nos damos con algunas/varias
cuestiones. Por dar un ejemplo en como nombrar las variables (que es un tema del que quiero hablar pronto) y en el 
caso de hoy con las interfaces.

Golang tiene sus particularidades, pero se basa en muchos patrones ya conocidos en la industria. En el ecosistema de Go 
a veces complejizamos algunos patrones y caemos en "anti patrones", en la siguiente publicación vamos a revisar un proyecto
falso y refactorizarlo para hacer un buen uso de las interfaces.

Para simplificar un poco la publicación vamos a acotar el caso de uso y lo reduciremos a la capa de servicio y repositorio.

Imaginemos que en la capa de repositorio nos encontramos con lo siguiente:

```go
package repository

import (
    // pkg imports
)

type Repository interface {
	Save(ctx context.Context, model *beer.Beer) (*beer.Beer, error)
}

type repository struct {
	// repository client and configs go here
}

func NewRepository() Repository {
	return &repository{}
}

func (repo *repository) Save(ctx context.Context, model *beer.Beer) (*beer.Beer, error) {
	// previous logic here
	return repo.toModel(beerEntity), nil
}
```

Y en la capa de servicio lo siguiente:

```go
package service

import (
    // pkg imports
)

type Service interface {
	Create(ctx context.Context, model *Beer) (*Beer, error)
}

type service struct {
	repo Repository
}

func NewService(repo Repository) Service {
	return &service{repo: repo}
}

func (svc *service) Create(ctx context.Context, model *Beer) (*Beer, error) {
	beer, err := svc.repo.Save(ctx, model)
	if err != nil {
		return nil, err
	}

	return beer, err
}
```

El anterior escenario es el que me encuentro comúnmente en los proyectos de Go y quiero comentarles que yo también supe 
cometer el mismo error.

Todo bien luispi, ¿pero cuál es el error?

>_Las interfaces Go generalmente pertenecen al paquete que usa valores del tipo de interfaz, no al paquete que implementa 
esos valores._ :scream_cat: :boom:

## Propuesta/aprendizaje

>_El paquete de implementación debe devolver tipos concretos (generalmente puntero o estructura): de esa manera, se pueden
agregar nuevos métodos a las implementaciones sin requerir una refactorización extensa._

Con esto en mente vayamos a los bifes

En primer lugar ataquemos la capa de repositorio, como bien dice la nota anterior vamos a retornar una estructura y
no la interfaz.

```go
package repository

import (
    // pkg imports
)

type Repository struct{
	// repository client and configs go here
}

func NewRepository() Repository {
	return Repository{}
}

func (repo *Repository) Save(ctx context.Context, model *beer.Beer) (*beer.Beer, error) {
	// previous logic here
	return repo.toModel(beerEntity), nil
}
```

Repasemos el cambio.

En primer lugar, eliminamos la interfaz y ahora la función `NewRepository()` retorna la estructura `Repository`, y en segundo lugar
agregamos a `Repository` el método save.

Todo bien luispi, ¿pero qué ganamos con este cambio?

Como no tenemos que cumplir con ningún contrato de interfaz no estamos atados a tener que implementar todos los métodos
que tenga la misma.

Tenemos que pensar a este paquete como un productor (`producer`) y siempre tengamos como nota mental que los `producer`, 
de nuevo, retornan tipos concretos (generalmente un puntero o una estructura).

Ahora es el turno de editar la capa del consumidor (`consumer`), en este caso el `service`.

```go
package service

import (
    // pkg imports
)

type Repository interface {
	Save(ctx context.Context, beer *Beer) (*Beer, error)
}

type Service struct {
	repo Repository
}

func NewService(repo Repository) Service {
	return Service{repo: repo}
}

func (svc *Service) Create(ctx context.Context, beer *Beer) (*Beer, error) {
	createBeer, err := svc.repo.Save(ctx, beer)
	if err != nil {
		return nil, err
	}

	return createBeer, err
}
```

Repasemos el cambio.

Realizamos varios cambios, en primer lugar declaramos la interfaz `Repository` y en la estructura `Service` inyectamos
la interfaz para que pueda consumirse en los métodos del servicio.

Al igual que con la capa de repositorio nuestro `NewService()` ahora retorna una estructura y no una interfaz.

Por ultimo agregamos el método `Create` a nuestro `Service`.

## Conclusiones

Con estos cambios sutiles pero poderosos nuestros `producer` ahora tienen una enorme flexibilidad.

Por último quiero agradecer a mi amigo y mentor [morenojp](https://www.linkedin.com/in/morenojp/) que me compartió este
anti patrón y me hizo repensar y mejorar, una vez más, en esto del desarrollo de software.

Para no aburrirte y por el momento hagamos una pausa.

Próximamente vamos a seguir con pequeñas publicaciones donde vamos a intentar repensar otros anti patrones.

¡Que pase bien!

---
Fuentes:
- [wiki oficial de Golang](https://github.com/golang/go/wiki/CodeReviewComments#interfaces)
- [go interfaces misuse](https://8thlight.com/blog/go-interface-misuse/)