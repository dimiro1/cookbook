# How to access the database without expsigin the implementation details?

Using repositories

```go
package store

type Recipe struct {
    Name string
    Description string
}
```

```go
package store

import (
    "errors"
    "database/sql"
)

var RecipeDoesNotExists = errors.New("recipe does not exists")

type RecipeRepository interface {
    FindByID(id int) (*Recipe, error)
    All(id int) ([]*Recipe, error)
}

type DBRecipeRepository struct {
    db *sql.DB
}

func NewDBRecipeRepository( db *sql.DB) (*DBRecipeRepository, error) {
    if db == nil {
        return nil, errors.New("db cannot be nil")
    }

    return &DBRecipeRepository{ db: db }, nil
}

func (d *DBRecipeRepository) FindByID(id int) (*Recipe, error) {
    // Implementation here
}

func (d *DBRecipeRepository) All(id int) ([]*Recipe, error) {
    // Implementation here
}
```

In your handler you use the interface, not the concrete implementation

```go
package handlers

type MyHandler struct {
    recipeRepository store.RecipeRepository
}

func NewMyHandler(recipeRepository store.RecipeRepository) (*MyHandler, error) {
    if recipeRepository == nil {
        return nil, errors.New("recipeRepository cannot be nil")
    }

    return &MyHandler{ recipeRepository: recipeRepository }, nil
}

func (h *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Use the repository
    recipe, err := h.recipeRepository.FindByID(50)
    // continue with your logic
}
```

## Why this is good?

- You hide the implementation details from your handler
- If you decide to switch databases you only have to implement a RecipeRepository with the implementation of your new database
- It is easier to test, you can have mocks or fakes that implement this interface
- It is easier to understand.