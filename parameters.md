# How to extract parameters from URL?

## The simplest way

```go
// /hello?name=Claudemiro
func IndexHandler(w http.ResponseWriter, r *http.Request) {
    name := r.URL.Query().Get("name")
    // ...
}
```

```go
// /hello?name=Claudemiro&age=30
func IndexHandler(w http.ResponseWriter, r *http.Request) {
	name := r.URL.Query().Get("name")
	ageString := r.URL.Query().Get("age") // always return a string
	age, err := strconv.Atoi(ageString)
	if err != nil {
		// return bad request
	}
	// ...
}
```

As you can see, when you need to do type conversion, things become complicated.

## Using a Extractor

This one is more complex, but have many benefits.

- Extractor is an interface, you can mock it or create a fake for test purposes
- You can share this extractor between packages and other projects
- It is type safe
- Can deal with complex convertions
- You can use the same approach to extract complex objects from the request body

```go
package params

type Extractor interface {
    NameFromQuery(r *http.Request) string
    AgeFromQuery(r *http.Request) (int, error)
    DateFromQuery(r *http.Request) (time.Time, error)
}

// Implements Extractor
func MyExtractor struct {}

func (MyExtractor) NameFromQuery(r *http.Request) string {
    return r.URL.Query().Get("name")
}

func (MyExtractor) AgeFromQuery(r *http.Request) (int, error) {
    age, err := strconv.Atoi(r.URL.Query().Get("age"))
	if err != nil {
		return 0, errors.New("age must be integer")
    }
    return age
}

func (MyExtractor) DateFromQuery(r *http.Request) (time.Time, error) {
	dateStr := fmt.Sprintf(
		"%s-%s-%s",
		r.URL.Query().Get("year"),
		r.URL.Query().Get("month"),
		r.URL.Query().Get("day"),
	)

	date, err := time.Parse("2006-01-02", dateStr)
	if err != nil {
		return time.Time{}, fmt.Errorf("invalid date %s", dateStr)
	}

	return date, nil
}
```

```go
package handlers

type Handler struct {
    extractor params.Extractor
}

func NewHandler(extractor params.Extractor) (Handler, error) {
    // Defensive validation
    if extractor == nil {
        return Handler{}, errors.New("extractor cannot be nil")
    }

    return Handler{
        extractor: extractor,
    }, nil
}

// /hello?name=Claudemiro&age=30&year=2018&month=07&day=18
func (h Handler) Index(w http.ResponseWriter, r *http.Request) {
    name := h.extractor.NameFromQuery(r)
    age, err := h.extractor.AgeFromQuery(r)
    if err != nil {
		// return bad request
    }
    
    // Extracting a complex object from the query parameters
    date, err := h.extractor.DateFromQuery(r)
    if err != nil {
		// return bad request
	}
}
```