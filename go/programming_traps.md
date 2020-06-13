Some Traps
---
- Trap 1: append elements to a slice in a function

    - The built-in append function will return the updated slice which is supposed to be stored. 
    - The underlying representation of a slice is a struct with three fields, pointer to the underlying array, len and cap.
    - Golang is a passed-by-value language. Struct will be copied to a new one, then pass into functions.
    
    => when append elements to a slice in an application function, the updated slice will not be visible to caller.

Bad case:
```go
func main() {
    x := []int{1, 2, 3}
    Append(x)
    fmt.Println(x)
}

func Append(x []int){
    x = append(x, 4)
}
```

Good one:
```go
func main() {
    x := []int{1, 2, 3}
    Append(&x)
    fmt.Println(x)
}

func Append(x *[]int){
    *x = append(*x, 4)
}
```
