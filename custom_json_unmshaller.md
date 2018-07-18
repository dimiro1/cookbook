# How can I create a custom UnmarshalJSON

```go
package main

import (
	"encoding/json"
	"fmt"
	"strings"
)

type MyType struct {
	ID   int
	Body []byte
}

func (m *MyType) UnmarshalJSON(b []byte) error {
	data := struct {
		ID int `json:"id"`
	}{}

	err := json.Unmarshal(b, &data)
	if err != nil {
		return err
	}

	m.ID = data.ID
	m.Body = b
	return nil
}

func main() {
	m := MyType{}
	err := json.NewDecoder(strings.NewReader(`{ "id": 10, "field_a": "a", "field_b": "b" }`)).Decode(&m)
	if err != nil {
		panic(err)
	}
	fmt.Println(m)
}

```