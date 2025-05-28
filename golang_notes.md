Golang Notes

- "go run xxx.go" compiles a single input and execute it

- "go build xxx.go" creates an executable binary

- a++ is a statement, not a expression, so b = a++ is invalid

- func init() {} is a special function automatically called when the program are executed. It can not be called in the code.

- A string may contains characters made up of multiple bytes, so the right approach to iterate the string is to convert it into []rune first, and go through it

- unsafe.Sizeof(x) can get the size of a variable, but if x is string, it returns its head size, that is 16 on a 64-bit system, you can consider a string in golang
```
type string struct {
    Data *byte  // pointer to the actual UTF-8 bytes
    Len  int    // length in bytes
}
```
- If x is a slice, it returns its head size, that is 24 on a 64-bit system. You may consider a slice like this:
```
type slice struct {
    Data *T  // pointer to the underlying array
    Len  int // number of elements
    Cap  int // capacity of the array
}
```
- Converting a string to []byte necessitate memory allocation, and so does the reverse because of the mutability difference

- We can use T[]{idx1: ele1, idx2: ele2} to initialize an array or slice

- The sha256.Sum256([]byte) can generate a [32]byte array

- We can use == to compare two arrays but not slices