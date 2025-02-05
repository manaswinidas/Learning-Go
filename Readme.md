<p align="center">
  <img  src="https://user-images.githubusercontent.com/33368759/80697133-2a085b00-8af6-11ea-9506-903486db94c1.png">
</p>
A simple list of concepts and code snippets that would help in learning Golang and applying in Web Development :tada:. I tried to jot down when I was learning. It might be helpful for beginners who want to learn Go for web development.
<br />
<br />

> ✌️ Hope you find something useful. If you like it please give it a 🌟.


## Contents
* [Installation](#Installation)
* [Concepts to learn before diving into Web](#Initial-Concepts-to-Study-before-diving-deep)
* [Basic Hello World](#Basic-Hello-World)
* [Adding static assets](#Adding-static-asset)
* [Creating Routes](#Adding-Routes)
* [Adding Forms](#Adding-Forms)
* [Adding MiddleWare](#Adding-MiddleWare)
* [Sessions Management](#Sessions-Management)
* [Adding Database](#Adding-Database)
* [Writing Unit Test](#Writing-Unit-Test)

## Installation

> Follow the [official doc](https://golang.org/doc/install) and setup Go depending on your OS (ie. Windows , Linux, OS X)

## Initial Concepts to Study before diving deep

* Basic Understanding of
    * Variables
    * Constants
    * Packages and import/export
    * Functions
    * Pointers
    * Mutability
 * Types
    * Type Conversion 
    * Type assertion**
    * Structs
    * Composition
 * Collection Types
    * Arrays
    * Slicing
    * Range & Maps
 * Control Flow
    * If, For, Switch statement
 * Methods
 * Interfaces
 * Concurrency
    * Goroutines
    * Channels





## Basic Hello World

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World !")
    })

    http.ListenAndServe(":8000", nil)
}
```
[Go back to top &#8593;](#Contents)

## Adding static asset

When we want to serve static files like CSS, JavaScript or images to Web.

```go
func main() {
    http.HandleFunc("/", func (w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello World!")
    })
   
    fs := http.FileServer(http.Dir("./static"))
    http.Handle("/static/", http.StripPrefix("/static/", fs))

    http.ListenAndServe(":8000", nil)
}
```
[Go back to top &#8593;](#Contents)

## Adding Routes

```go
package main

import (
    "fmt"
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
)


type Tasks struct {
    ID 		string  	`json:"id,omitempty"`
    TASKNAME 	string 		`json:"task,omitempty"`
}

var task []Tasks

func getAllTask(w http.ResponseWriter, r *http.Request) {
	json.NewEncoder(w).Encode(task)
}


func getTask(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	for _,item := range task {
		if item.ID == params["id"] {
			json.NewEncoder(w).Encode(item)
			return
		}
	}
	json.NewEncoder(w).Encode(&Tasks{})
}


func main() {
	router := mux.NewRouter()
	router.HandleFunc("/task", getAllTask).Methods("GET")
	router.HandleFunc("/task/{id}", getTask).Methods("GET")
	
	http.ListenAndServe(":8000", router)
}

```

[Go back to top &#8593;](#Contents)

## Adding Forms

Considering the form has 2 fields `Email` and `Message`.

```go

package main

import (
	"log"
	"fmt"
	"net/http"
)

type Details struct {
    Email   string
    Message string
}

func messageHandle(w http.ResponseWriter, r *http.Request) {
	if err := r.ParseForm(); err != nil {
		fmt.Fprintf(w, "ParseForm() err: %v", err)
		return
	}
	
	data := Details{
		Email:   r.FormValue("email"),
		Message: r.FormValue("message"),
        }

        // do something with the data
}

func main() {
    http.HandleFunc("/", messageHandle)
    
    if err := http.ListenAndServe(":8080", nil); err != nil {
        log.Fatal(err)
    }
}


```

[Go back to top &#8593;](#Contents)

## Adding MiddleWare

Here, the `Middlware` function allows adding more than one layer of middleware and handle them appropriately.
`SomeMiddleware` is the middleware function which gets called before the route handler function `getAllTask`

```go

package main

import (
    "encoding/json"
    "log"
    "net/http"
    "github.com/gorilla/mux"
)

func getAllTask(w http.ResponseWriter, r *http.Request) {
	// ... do something inside this route
}


// Function allows adding more than one layer of middleware and handle them appropriately

func Middleware(h http.Handler, middleware ...func(http.Handler) http.Handler) http.Handler {
    for _, mw := range middleware {
        h = mw(h)
    }
    return h
}

// Middlware function

func SomeMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		
		 // ... do middleware things

        next.ServeHTTP(w, r)
    })
}

func main() {

	router := mux.NewRouter()
	router.Handle("/task", Middleware(
        http.HandlerFunc(getAllTask),
        SomeMiddleware,
    ))
    log.Fatal(http.ListenAndServe(":8000", router))
}


```

[Go back to top &#8593;](#Contents)

## Sessions Management

```go

import (
    "os"
    "log"
    "net/http"
    "github.com/gorilla/sessions"
)

var store = sessions.NewCookieStore([]byte(os.Getenv("SESSION_KEY")))

func Handlerfunction(w http.ResponseWriter, r *http.Request) {
	session, err := store.Get(r, "session-name")
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	
	// Set some session values.
    	session.Values["hello"] = "world"
    	// Save it 
    	session.Save(r, w)
}


func main() {
    http.HandleFunc("/", Handlerfunction)
    
    if err := http.ListenAndServe(":8080", nil); err != nil {
        log.Fatal(err)
    }
}

```

[Go back to top &#8593;](#Contents)


## Adding Database

Here in this example we have connected MongoDB with our applicationa and saved sample data into the collection 

```go

package main

import (
"context"
"fmt"
"os"
"time"
"go.mongodb.org/mongo-driver/mongo"
"go.mongodb.org/mongo-driver/mongo/options"
)

type Data struct {
	ID  int `json:"Field Int"`
	Task string `json:"Field Str"`
}

func main() {
	
	clientOptions := options.Client().ApplyURI("<MONGO_URI>") 

	// Connect to the MongoDB
	client, err := mongo.Connect(context.TODO(), clientOptions)
	
	if err != nil {
		fmt.Println("mongo.Connect() ERROR:", err)
		os.Exit(1)
	}

	// To manage multiple API requests
	ctx, _ := context.WithTimeout(context.Background(), 15*time.Second)

	// Access a MongoDB collection through a database
	col := client.Database("DATABASE_NAME").Collection("COLLECTION_NAME")

	// Declare a MongoDB struct instance for the document's fields and data
	newData := Data{
		ID: 12,
		Task: "Learn Go",
	}

	result, err := col.InsertOne(ctx, newData)
	if err != nil {
		fmt.Println("ERROR:", err)
		os.Exit(1)

	} else {
	fmt.Println("Result:", result)
	}
}

```

[Go back to top &#8593;](#Contents)


## Writing Unit Test

Consider the [Adding Routes](#Adding-Routes) section for testing. The below test case is for the `/task` route which returns an array of tasks created by users.

```go

package main

import (
	"net/http"
	"testing"
	"net/http/httptest"
	"strings"
)

func TestGetAllTask(t *testing.T) {
	req, err := http.NewRequest("GET", "http://localhost:8000/task", nil)
	if err != nil {
		t.Fatal(err)
	}
	res := httptest.NewRecorder()
	handler := http.HandlerFunc(getAllTask)
	handler.ServeHTTP(res, req)
	
	if status := res.Code; status != http.StatusOK {
		t.Errorf("Wrong Status Code: got %v want %v",
			status, http.StatusOK)
	}

	// Check the response body is what we expect.
	expected := `[{"id":"1","task":"Hello"},{"id":"2","task":"World"},{"id":"3","task":"yeah"}]`

	if strings.TrimRight(res.Body.String(),"\n") != expected {
		t.Errorf("ERROR: got %v want %v",
			res.Body.String(), expected)
	}
}

```
Remember Test file should be name of original file + test like: `base.go` - `base_test.go`.(good practice)

After running the above test case by `go test -v` command, the following output will appear

```bash
F:\Go\src\Rest_API>go test -v
=== RUN   TestGetAllTask
--- PASS: TestGetAllTask (0.00s)
PASS
ok      Rest_API        0.454s
```

[Go back to top &#8593;](#Contents)

## Licence

MIT © [Debasish Sahoo](https://github.com/debck/Learning-Go/blob/master/LICENSE)

