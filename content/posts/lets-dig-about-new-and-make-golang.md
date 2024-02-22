+++
title = "Understanding the Nuances of new() and make() in Go"
date = 2024-02-22T18:26:57+01:00
draft = false
tags = ["go"]
+++

Go has couple of memory allocation primitives: `new()` and `make()`.

### new(T)

__new(T)__ is allocation primitive which allocates the memory for type `T` but zeros it. Unlike Java or C#, go does not do further initialization. new(T) returns a pointer `*T`.

```go
import "bytes"

type User struct {
	Name string
}

func main() {
	b := new(bytes.Buffer)
	fmt.Println(b.Len())

	n, _ := b.Write([]byte("Hello hello"))
	fmt.Println(n)
	b.WriteTo(os.Stdout)

	user := new(User)
	fmt.Printf("\n%#v", user)
}

// Output:
// 0
// 11
// Hello hello
// &main.User{Name:""}
```

As shown in the output, the variable for buffer **b** is allocated with zeroed memory but the variable is still available for immediate usage to write contents.

__new()__ is specially useful to create types like buffer or mutex whose zero values could be used without further initialization.

The expressions __new(User)__ and __&User{}__ are equivalent.

### make(T, args)

__make()__ is different from __new()__ in the sense that it initializes memory for type __T__ and it takes _length_ and _capacity_ as arguments.

Only channels, slices and maps can be created using __make()__ and these types references to another underlying data structures that must be initialized at the first place in order to be useful.

For example, 

```go 
s := make([]int, 10, 50)
```

__s__ is a slice which references to another underlying array that is initializedwith 50 elements and __s__ points to the first 10 elements in that array.
  
One can initialize a slice using __new()__ but the resulting value is _nil_ and is rarely useful to contain elements.

Another distinction of __make(T, args)__ and __new(T)__ is that __make__ do not return a pointer but returns type __T__.

The following example explains the difference between make and new, 

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

