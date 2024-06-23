---
title: "Interfaces and \"copy paste\""
description: "Preventing the abuse or misuse of interfaces favors code maintainability."
date: 2022-06-20T10:47:02-03:00
tags: ["best-practices", "golang", "go"]
categories: ["stop-copy-paste"]
---
In the following post I want to share a common mistake I used to make in my Golang projects. 
And as the title and description says, today we are going to talk about interfaces.

You will probably come across some references to another post I did on 
[hexagonal architecture](https://luispe.github.io/blog/posts/hexagonal-architecture/), I think part of the misuse 
of interfaces is the consequence of following "medium" publications to the letter and not taking the time 
to understand the underlying concept. 

## Preface

If there is something that we have to admit in the Golang ecosystem, it is the kink that we sometimes give ourselves 
with some/several issues. To give an example on how to name variables (which is a topic I want to talk about soon) 
and in today's case with interfaces.

Golang has its particularities, but it is based on many patterns already known in the industry. 
In the Go ecosystem sometimes we complex some patterns and fall into "anti patterns", 
in the next post we will review a fake project and refactor it to make good use of the interfaces.

To simplify the publication a bit, we will narrow down the use case and reduce it to the service and repository layer.

Let's imagine that in the repository layer we find the following:

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

And in the service layer the following:

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

The above scenario is the one I commonly encounter in Go projects and I want to tell you that 
I also made the same mistake.

All good luispi, but what's the mistake?

>_Go interfaces generally belong to the package that uses values of the interface type, 
> not to the package that implements those values._ 🫠

## Proposal/learning

>_The implementation package must return concrete types (usually pointer or struct): that way, 
> new methods can be added to implementations without requiring extensive refactoring._

With this in mind, let's get to work

First, let's attack the repository layer, as the previous note says, we are going to return a 
structure and not the interface.

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

Let's review the change.

First we remove the interface and now the `NewRepository()` function returns the `Repository` structure, 
and second we add to `Repository` the save method.

All well and good luispi, but what do we gain with this change?

Since we do not have to comply with any interface contract we are not tied to having to implement 
all the methods that the interface has.

We have to think of this package as a producer (`producer`) and always keep in mind that `producers`, again, 
return concrete types (usually a pointer or a structure).

Now it is the turn to edit the consumer layer, in this case the `service`.

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

Let's review the change.

We make several changes, first we declare the `Repository` interface and in the `Service` structure 
we inject the interface so that it can be consumed in the service methods.

As with the repository layer our `NewService()` now returns a structure and not an interface.

Finally, we add the `Create` method to our `Service`.

## Conclusions

With these subtle but powerful changes our `producer` now has enormous flexibility.

Finally, I want to thank my friend and mentor [morenojp](https://www.linkedin.com/in/morenojp/) who shared with me 
this anti pattern and made me rethink and improve, once again, in this software development.

So as not to bore you and for the moment let's take a break.

Soon we will continue with small publications where we will try to rethink other anti-patterns.

Have a good time! 👋🏽

---
Sources:
- [wiki oficial de Golang](https://github.com/golang/go/wiki/CodeReviewComments#interfaces)
- [go interfaces misuse](https://8thlight.com/blog/go-interface-misuse/)
