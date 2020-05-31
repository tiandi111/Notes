Timer vs Ticker 
---

Core difference
- Timer fires only once
- Ticker fires every interval

Some Bad cases
---
Bad case 1
```go
func main() {
	for {
		select {
		case <- time.After(time.Second):
			fmt.Println("hi")
		case <- time.After(time.Second*2):
			fmt.Println("ho")
		}
	}
}
```
"ho" will never be printed because time.After() will create a new time.Timer
every iteration.

Bas case 2
```go
func main() {
	timeout1 := time.After(time.Second)
	timeout2 := time.After(time.Second*2)
	for {
		select {
		case <-timeout1:
			fmt.Println("hi")
		case <-timeout2:
			fmt.Println("ho")
		}
	}
}
```
Now we avoid creating new time.Timer every iteration, but this is still a bad
case because time.Timer will only fire once. After the two time.After() fired,
the main goroutine deadlocked.

Below is the program output:
```go
hi
ho
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select]:
main.main()
        /Users/tiandi03/go/src/awesomeProject/t.go:12 +0xda

Process finished with exit code 2

```