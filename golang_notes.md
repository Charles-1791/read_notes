# Golang Notes

## compilation
- "go run xxx.go" compiles a single input and execute it

- "go build xxx.go" creates an executable binary

- func init() {} is a special function automatically called when the program are executed. It can not be called in the code.

## syntax
- a++ is a statement, not a expression, so b = a++ is invalid

## Golang type categories

### basic types
- int, int8, int16, int32, int64
- uint, uint8, uint16, uint32, uint64
- float32, float64 (there is no float)
- complex64, complex128
- bool
- string

### aggregate types
- array
- struct

### reference types

consider them as pointers 
- pointer
- slice
- map
- function
- channel 

### interface types
- interface

## string, []byte and []rune

String is immutable. To change a character, we need to convert string into []rune on which we apply the modification and then convert it back.

Obeying the UTF-8 standard, a string may contain characters made up of multiple bytes, so the right approach to iterate the string is to convert it into []rune first, and go through it

Converting a string into a []rune allocates new memory. Since rune is fixed-lengthed, after the size of the []rune is always no less than the string.

unsafe.Sizeof(x) can get the size of a variable, but if x is string, it returns its head size, that is 16 on a 64-bit system, we can consider a string in golang having the following structure:
```
type string struct {
    Data *byte  // pointer to the actual UTF-8 bytes
    Len  int    // length in bytes
}
```

Converting a string to []byte also necessitates memory allocation, and so does the reverse.

## array and slice

### slice
- unsafe.Sizeof() returns a slice's head size, which is 24 on a 64-bit system. We may consider a slice like this:
```
type slice struct {
    Data *T  // pointer to the underlying array
    Len  int // number of elements
    Cap  int // capacity of the array
}
```
- We can use T[]{idx1: ele1, idx2: ele2} to initialize a slice

- cap(slc) and return the capacity of the slice

### array
- We can use T[...]{} to initialize an array, the size of which is deduced from the number of elements in curly brace.

- The sha256.Sum256([]byte) can generate a [32]byte array

- We can use == to compare two arrays but not slices

- When an array is used as a function input parameter, the function **deep copy** the original array. So mutation happened on the copied array doesn't go to the original input. Use *[]T to avoid copy.