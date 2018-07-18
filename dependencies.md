# How to add dependencies into HTTP handlers?

Given that we have this interface that will be used as a dependency.

```go
type RecipeFinder interface {
    FindByID(id int) *Recipe
}
```

## Using a functional approach

```go
func IndexHandler(finder RecipeFinder) (http.Handler, error) {

    // Defensive validation
    if recipeFinder == nil {
        return nil, errors.New("recipeFinder cannot be nil")
    }

    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Use your dependendy
        finder.FindByID(50)
        // ...
    }), nil
}

finder := NewRecipeFinder()

handler, err := IndexHandler(finder)
if err != nil {
    // Do something with the error
}
http.HandleFunc("/", handler)
```
Or you can remove the defensive validation and make it simpler
```go
func IndexHandler(finder RecipeFinder) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Use your dependendy
        finder.FindByID(50)
        // ...
    })
}

finder := NewRecipeFinder()
http.HandleFunc("/", IndexHandler(finder))
```

### Pros

- Easy to use
- Simple to understand
- Programmers from functional languages will feel at home
- Dependencies are restricted to use in this function

### Cons

- Programers comming from OO languages will need some time to understand

## Using a OO approach

```go
type MyHandler struct {
    recipeFinder RecipeFinder
    // Maybe other dependencies
}

func NewMyHandler(recipeFinder RecipeFinder) (MyHandler, error) {
    // Defensive validation
    if recipeFinder == nil {
        return MyHandler{}, errors.New("recipeFinder cannot be nil")
    }
    return MyHandler{
        recipeFinder: recipeFinder,
    }
}

func (m MyHandler) Index(w http.ResponseWriter, r *http.Request) {
    // Use your dependendy
    m.finder.FindByID(50)
    // ...
}

func (m MyHandler) About(w http.ResponseWriter, r *http.Request) {
    // Use your dependendy
    // as the dependency is bound to the MyHandler struct you are free to use it in any handler
    m.finder.FindByID(50)
    // ...
}
```

### Pros

- Easy to use
- Simple to understand
- Programmers comming from OO languages will feel at home
- You can use the same dependency for more than one Handler

### Cons

- You can use the same dependency for more than one Handler(If you want complete isolation)


## Which one is better?

It is up to you and your team, I prefer to use the functional style when the number of dependencies are smnall and when I do not have many HTTP Handlers.