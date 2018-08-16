### Intro
We said previously that in Go you tipically use interfaces to design your programs. Go programs are usually(but not always) centered around a few interfaces that expose the behaviour of the module you are working on.  
The packages in the standard libraries are no different from this. There are lot of interfaces running around, some of the most popular ones are `io.Writer`, `io.Reader`, `net.Conn` and the one we will be focusing in this section: [http.Handler](https://godoc.org/net/http#Handler).  

### http.Handler
This interface has a single method: `ServeHTTP(ResponseWriter, *Request)`. Whatever implements this method will be an `http.Handler` capable of handling http requests and returning responses. Lets illustrate with an example. Say we are writing a simple hello world http server, but instead of saying hello world we want to say `hello <your name>`. To keep things simple your name will not be entered by the user, we will harcoded it into the code. Lets first define a new type based on the type string: `type Name string`. Now that we have a type we can define methods on it, since we want to handle http requests with this type we'll implement the `ServeHTTP` method:
```go
type Name string

func (n Name) ServeHTTP(w http.ResponseWriter, r *http.Request) {
}
```
We now have an HTTP handler that we could easily embed on an http server using the [http.Handle](https://godoc.org/net/http#Handle) function. This function receives a path and an `http.Handler` that handles a request. Each request will be executed in a new `goroutine`:
```go
func main() {
        n := Name("Globant")
        http.Handle("/", n)
        log.Fatal(http.ListenAndServe(":8080", nil))
}
```
Right now our HTTP handler does nothing, lets add the functionality of replying "Hello Globant" to a GET request:
```go
func (n Name) ServeHTTP(w http.ResponseWriter, r *http.Request) {
        if r.Method != http.MethodGet {
                http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
                return
        }
        if _, err := w.Write([]byte(fmt.Sprintf("Hello %s", n))); err != nil {
                http.Error(w, err.Error(), http.StatusInternalServerError)
        }
}
```
Time to put all the code together:
```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

// Name holds a name and implements http.Handler.
type Name string

func (n Name) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodGet {
		http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
		return
	}
	if _, err := w.Write([]byte(fmt.Sprintf("Hello %s", n))); err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

func main() {
	n := Name("Globant")
	http.Handle("/", n)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

```
Go to [localhost:8080](http://localhost:8080) in your browser or `curl` it. If all went well you should see the message `Hello Globant`. Congrats!!