---
draft: false
title: "Hexagonal Architecture"
description: "How to decouple our system layers correctly"
date: 2022-06-09T21:53:34-03:00
tags: ["best-practices", "golang", "go"]
categories: ["stop-copy-paste"]
---
This is the first of a series where we will review different development patterns that I made 
mistakes and what is currently and what I understand so far, the best way to apply it.

As the title of the publication says, today I want to talk about hexagonal architecture.

## Preface
I have been noticing a lot of "hype" around the hexagonal architecture and I would like to be clear about it, 
I am not against it, on the contrary, I think it is an excellent pattern.

But I think we fall into the mistake of applying the publishing recipes of "medium" and not only end up 
with all the layers of our system coupled but with package names like "adapters" or "ports" and if there is 
something beautiful in the language of Go(lang) is the intentionality in the name of a package.

>A good name on the package means that almost nothing else is needed to express the intentionality of the package.

I share and recommend you to read the following [official publication](https://go.dev/blog/package-names), 
personal opinion, applies to any language.

## Let's start

>Anti pattern
> 
>Annotation in our domain model e.g `json`.

One mistake I used to make is having `json` or `gorm` annotations or any annotations in the domain model.

Let's imagine that we have our `/user/user.go` model with the following content

```go
type User struct {
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
}
```

Before reading on, I invite you to take a few seconds/minutes to reflect on why this is an anti-pattern.

## Proposal/learning

Our model should be presentation and data layer agnostic, therefore, it should not contain any annotations. 
We should map the data to our model and vice versa, here is an example of what a rest HTTP controller 
would look like e.g.:

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

As we can see in the previous example the private methods in the rest controller map from the `data transfer object` 
to the domain model when we call the service and vice versa.

Finally, we remove the annotations in our `user/user.go` model.

```go
type User struct {
	FirstName string
	LastName  string
}
```

All good luispi, but what gain did we get?

Now our `/user/user.go` not having any annotation no matter how we present or how we obtain/persist the data, 
we will never touch the core/domain of our application, now we could affirm that our core/domain is agnostic 
to the presentation or data layers gaining flexibility, testing, etc.

So as not to bore you and for the moment let's take a break.

Soon we will continue with small publications where we will try to rethink other anti-patterns.

Have a good time! 👋🏽
