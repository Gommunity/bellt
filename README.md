<p align="center">
    <img width="200" src="./logo.png">
</p>


# Bellt  
> Simple Golang HTTP router

## Setup

To get Bellt

##### > Go CLI
```sh
go get github.com/GuilhermeCaruso/bellt
```
##### > Go DEP
```sh
dep ensure -add github.com/GuilhermeCaruso/bellt
```
##### > Govendor
```sh
govendor fetch github.com/GuilhermeCaruso/bellt
```

# Guide

## Router

To initialize our router
```go
    var router = bellt.NewRouter()
```

## Router Methods

- HandleFunc   

HandleFunc function responsible for initializing a common route or built through the Router. All non-grouped routes must be initialized by this method.

```go
    /*
        path - Endpoint string
        handlerFunc - function that will be called on the request
        methods - Slice for endpoint methods ("GET", "POST", "PUT", "DELETE")
    */

    router.HandleFunc(path, handlerFunc, methods)
    
```
- HandleFunc with Middlewares
```go
    /*
        path - Endpoint string
        handlerFunc - function that will be called on the request
        methods - Slice for endpoint methods ("GET", "POST", "PUT", "DELETE")
    */

    router.HandleFunc(path, bellt.Use(
        handlerFunc,
        middlewareOne,
        middlewareTwo,
    ), methods)
   
```

- HandleGroup && SubHandleFunc    

HandleGroup used to create and define a group of sub-routes.

SubHandleFunc is responsible for initializing a common or built route. Its use must be made within the scope of the HandleGroup() method, where the main path will be declared.
```go

    /*
        mainPath - String route grouper
        path - Endpoint string
        handlerFunc - function that will be called on the request
        methods - Slice for endpoint methods ("GET", "POST", "PUT", "DELETE")
    */

    router.HandleGroup(mainPath,
        router.SubHandleFunc(path, handlerFunc, methods),
        router.SubHandleFunc(path, bellt.Use(
            handlerFunc,
            middlewareOne,
            middlewareTwo,
        ), methods),
    )
    
```

## Route Params

To use parameters in routes we must group the variables in braces.
```go
    router.HandleFunc("/user/{id}", handlerFunc, methods)
    router.HandleGroup("api/",
        router.SubHandleFunc("product/{id}", handlerFunc, methods),
        router.SubHandleFunc("product/{id}/{categorie}", bellt.Use(
            handlerFunc,
            middlewareOne,
            middlewareTwo,
        ), methods),
    )
```
## Get Route Params
```go
    router.HandleFunc("/user/{id}/{user}", exampleFunction, "GET")

    func exampleFunction(w http.ResponseWriter, r *http.Request) {
        rv := bellt.RouteVariables(r)

        fmt.Println(rv.GetVar("user"))
        fmt.Println(rv.GetVar("id"))
    }
    // Guilherme
    // 123

```
    

# Examples

Let's start our simple router application.

```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/GuilhermeCaruso/bellt"
)

func main() {

	router := bellt.NewRouter()

	router.HandleFunc("/healt", healthApplication , "GET", "PUT")
	log.Fatal(http.ListenAndServe(":8080", nil))
}

```

## Full Example
```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/GuilhermeCaruso/bellt"
)

func main() {

	router := bellt.NewRouter()

	router.HandleFunc("/contact/{id}/{user}", bellt.Use(
		exampleHandler,
		middlewareOne,
		middlewareTwo,
	), "GET", "PUT")

	router.HandleFunc("/contact", bellt.Use(
		exampleNewHandler,
		middlewareOne,
		middlewareTwo,
	), "GET", "PUT")

	router.HandleGroup("/api",
		router.SubHandleFunc("/check", bellt.Use(
			exampleNewHandler,
			middlewareOne,
			middlewareTwo,
		), "GET", "PUT"),
		router.SubHandleFunc("/check/{id}/{user}", bellt.Use(
			exampleHandler,
			middlewareOne,
			middlewareTwo,
		), "GET", "PUT"),
	)

	log.Fatal(http.ListenAndServe(":8080", nil))
}

func exampleHandler(w http.ResponseWriter, r *http.Request) {
	rv := bellt.RouteVariables(r)
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(200)
	w.Write([]byte(fmt.Sprintf(`{"id": %v, "user": %v}`, rv.GetVar("user"), rv.GetVar("id"))))
}

func exampleNewHandler(w http.ResponseWriter, r *http.Request) {
	rv := bellt.RouteVariables(r)
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(200)
	w.Write([]byte(`{"msg": "Works"}`))
}

func middlewareOne(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("Step One")

		next.ServeHTTP(w, r)
	}
}

func middlewareTwo(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("Step Two")

		next.ServeHTTP(w, r)
	}
}
```

# Author
Guilherme Caruso  [@guicaruso_](https://twitter.com/guicaruso_) on twitter

# License
BSD licensed. See the LICENSE file for details.