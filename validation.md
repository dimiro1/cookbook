# How do I do struct validations?

```go
package validation

type Error struct {
	Field   string `json:"field"`
	Message string `json:"message"`
	Cause   error  `json:"-"`
}

func (e Error) Error() string {
	return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

type Errors []Error

func (e *Errors) Error() string {
	var errorsAsString []string

	for _, entry := range *e {
		errorsAsString = append(errorsAsString, entry.Error())
	}

	return strings.Join(errorsAsString, ";")
}

func (e *Errors) Add(field, message string, cause error) {
	*e = append(*e, Error{Field: field, Message: message, Cause: cause})
}

func (e *Errors) IsEmpty() bool { return len(*e) == 0 }
```

```go
package models

type Person struct {
    Name string
    Age int
}

func (p Person) Validate() validations.Errors {
    e := validation.Errors{}

    if p.Name == "" {
        // The third parameter is an error object
        e.Add("name", "name cannot be blank", nil)
    }

    if p.Age < 18 {
        e.Add("age", "age must be greater than 18", nil)
    }

    return e
}
```

```go
package jsonvalidation

import (
    "fmt"
    "github.com/xeipuuv/gojsonschema"
)

type JSONSchema struct{
    schema string
}

func (j JSONSchema) Validate(r *http.Request) (validations.Errors, error) {
    e := validation.Errors{}

    loader := gojsonschema.NewStringLoader(j.schema)
    // document := 
    b, err := ioutil.ReadAll(r.Body)
    if err != nil {
        return validation.Errors{}, err
    }

    document := string(b)
    result, err := gojsonschema.Validate(loader, document)
    if err != nil {
        return validation.Errors{}, err
    }

     for _, err := range result.Errors() {
         e.Add(result.Field(), result.Description(), nil)
    }

    return e
}

```

```go
type Handler struct {
    extractor params.Extractor
    schema jsonvalidation.JSONSchema
}

func (h Handler) Index(w http.ResponseWriter, r *http.Request) {
    p := h.extractor.PersonFromRequest(r)

    if validationResult := p.Validate(); !validationResult.IsEmpty() {
        // show some error to the user
        // You can loop in the validationResult to collect all the errors
        return
    }

    // continue with your logic here
}

func (h Handler) FromJSONSchema(w http.ResponseWriter, r *http.Request) {
    validationResult, err := h.schema.Validate(r)
    if err != nil {
        // show some error to the user
        // 500 Error
    }

    if!validationResult.IsEmpty() {
        // show some error to the user
        // You can loop in the validationResult to collect all the errors
        return
    }

    // continue with your logic here
}
```

## Pros

- Easy to use
- We can have a shared validation package between projects
- It is generic
- Can be used for anhything that can be extracted from request