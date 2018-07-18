# How do I create fakes of interfaces for testing?

```go
package calculator

type Calculator interface {
    SumInts(a, b int) int
}

type ExternalCalculator struct {}

func (ExternalCalculator) SumInts(a, b int) int {
    // Here some very complex implementation
    // Accessing external services...
}

// a Calculator fake
type CalculatorFaker struct {
    SumIntsCount int
    SumIntsFn func(a, b int) int
}

func (c *CalculatorFaker) SumInts(a, b int) int {
    c.SumIntsCount++
    return c.SumIntsFn(a, b)
}
```

In this example, we have this function SomeFunction that accepts a calculator.Calculator and perform the calculation

```go
package somepackage

func SomeFunction(c calculator.Calculator, a, b int) int {
    return c.SumInts(a, b)
}
```

On your tests

```go
package somepackage_test

import "testing"

func TestSomeFunction(t *testing.T) {
    myFakeCalculator := &calculator.CalculatorFaker{
        SumIntsFn func(a, b int) int {
            return a + b // The implementation of my function
        },
    }

    s := somepackage.SomeFunction(myFakeCalculator, 10, 20)

    if s != 30 {
        t.Error("s != 30")
    }
}
```

## Why is it good?

- Your tests does not have to depend on slow external dependencies
- You are testing the logic of your functions, how they behaves in different situations
- You can test edge cases that were difficult to test if you were using a real service