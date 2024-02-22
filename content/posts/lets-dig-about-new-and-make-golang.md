+++
title = "Let's Think a second about New and Make in Go"
date = 2024-02-22T18:26:57+01:00
draft = true
+++

Go has couple of memory allocation primitives which are `new()` and `make()`.

### New()

`new()` is allocation primitive which initializes the memory of a type `T` with it's zero values. It returns a pointer `*T` and the value is readily available for usage.

```go
import "bytes"

func main() {
	b := new(bytes.Buffer)
}
```
