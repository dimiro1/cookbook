# How do I proper start an HTTP Server? 

Go standard `http.ListenAndServe` it is not safe as it does not have any timeouts. This can become a big problem as you can leak connections that were not closed properly.

```go
package main

import (
    "net/http"
    "os"
    "time"
)

func main() {
    server := &http.Server{
		Addr:         ":8080",
		Handler:      nil, // using the standard router
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 5 * time.Second,
		IdleTimeout:  60 * time.Second,
	}

	if err := server.ListenAndServe(); err != nil {
        // log the error and exist with error code 1
        os.Exit(1)
    }
}
```

See for more info: https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/