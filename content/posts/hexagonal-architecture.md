---
title: "Hexagonal Architecture"
date: 2022-06-09T21:53:34-03:00
draft: false
tags: ["best-practices", "golang", "go"]
categories: ["stop-copy-paste"]
author: "luispi"
---
Esta es la primera de una serie donde vamos a revisar diferentes patrones de desarrollo que errores cometí y cuál es
en la actualidad y lo que entiendo hasta el momento, la mejor forma de aplicarlo.

Como bien dice el título de la publicación hoy quiero hablar sobre arquitectura hexagonal.

---
## Preámbulo
Hace tiempo noto mucho "hype" en torno a la arquitectura hexagonal y me gustaría ser claro al respecto, no estoy en contra, 
mas bien todo lo contrario, me parece un excelente patrón.

Pero creo que caemos en el error de aplicar la recetas de publicaciones de "medium" y no solo terminamos con todas las 
capas de nuestro sistema acopladas sino que con nombres de packages al estilo "adapters" o "ports" y si hay algo hermoso 
en el lenguaje de Go(lang) es la intencionalidad en el nombre de un package.

>Un buen nombre en el package hace que casi ni sea necesario nada más para expresar la intencionalidad del mismo.

Te comparto y recomiendo leer la siguiente [publicación oficial](https://go.dev/blog/package-names), opinión personal,
aplica para cualquier lenguaje.

## Comencemos

>Anti patrón
> 
>Anotación en nuestro modelo de dominio e.g `json`

Un error que solía cometer es tener las anotaciones `json` o `validate` o cualquier anotación en el modelo de dominio.

Imaginemos que tenemos nuestro modelo `/user/user.go` con el siguiente contenido

```go
type User struct {
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
}
```

Antes de continuar leyendo te invito a que por unos segundos/minutos hagamos una reflexión de porque es un anti patrón.

## Propuesta/aprendizaje

Nuestro modelo debería ser agnóstico a la capa de presentación y a la de datos, por lo tanto, no debería contener ninguna
anotación. Deberíamos mapear los datos hacia nuestro modelo y viceversa, a continuación un ejemplo de como quedaría
un controlador rest HTTP e.g.:

`user/rest_controller.go`
```go
type UserDTO struct {
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
}

func (ctrl *controller) toUserModel(userDTO *UserDTO) *User {
	return &User{
		FirstName:	userDTO.FirstName,
		LastName:	userDTO.LastName,
	}
}

func (ctrl *controller) toUserDTO(user *User) *UserDTO {
	return &UserDTO{
		FirstName:	user.FirstName,
		LastName:	user.LastName,
	}
}
```

Como podemos ver en el ejemplo anterior los metodos privados en el controlador rest mapean desde el `data transfer object`
hacia el modelo de dominio cuando llamemos al servicio y viceversa.

Por último quitamos las anotaciones en nuestro modelo `user/user.go`

```go
type User struct {
	FirstName string
	LastName  string
}
```

Todo bien luispi, ¿pero qué ganancia obtuvimos?

Ahora nuestro `/user/user.go` al no tener ninguna anotación no importa como presentemos o como obtengamos/persistamos
los datos, nunca vamos a tocar el core/dominio de nuestra aplicación, ahora si podríamos afirmar que nuestro core/dominio
es agnóstico a las capas de presentación o datos ganando flexibilidad, testabilidad, etc.

Para no aburrirte y por el momento hagamos una pausa.

Próximamente vamos a seguir con pequeñas publicaciones donde vamos a intentar repensar otros anti patrones.

¡Que pase bien!
