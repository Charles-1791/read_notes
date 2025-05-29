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

Obeying the UTF-8 standard, a string may contain characters made up of multiple bytes, so the right approach to iterate the string is to convert it into []rune first, and go through it.

a for-loop iterates a string rune by rune rather than byte by byte.

Converting a string into a []rune allocates new memory. Since rune is fixed-lengthed, after the size of the []rune is always no less than the string.

unsafe.Sizeof(x) can get the size of a variable, but if x is string, it returns its head size, that is 16 on a 64-bit system, we can consider a string in golang having the following structure:
```golang
type string struct {
    Data *byte  // pointer to the actual UTF-8 bytes
    Len  int    // length in bytes
}
```

Converting a string to []byte also necessitates memory allocation, and so does the reverse.

## array and slice

### slice
- unsafe.Sizeof() returns a slice's head size, which is 24 on a 64-bit system. We may consider a slice like this:
```golang
type slice struct {
    Data *T  // pointer to the underlying array
    Len  int // number of elements
    Cap  int // capacity of the array
}
```
- We can use T[]{idx1: ele1, idx2: ele2} to initialize a slice

- cap(slc) and return the capacity of the slice

#### Memory Concern

- Using a subscript operator, that is X[i:j], to generate a sub-slice Y from some slice X actually allocates no memory. Mutation to Y would affect X unless Y points to a new piece of chunk, which happens when appending a new element after the capacity is exhausted. 

```golang
	X := []int{0,1,2,3,4,5,6,7,8}
	fmt.Println(cap(X))
	// output: 9
	Y := X[3:5]
	Y[0] = 300
	Y = append(Y, 500)
	Y = append(Y, 600)
	Y = append(Y, 700)
	Y = append(Y, 800)
	fmt.Printf("X = %v\n", X)
	// output: X = [0 1 2 300 4 500 600 700 800]
	fmt.Printf("Y = %v\n", Y)
	// output: Y = [300 4 500 600 700 800]
	// now, Y points to a different piece of memory
	Y = append(Y, 900)
	Y[1] = 400
	fmt.Printf("X = %v\n", X)
	// output: X = [0 1 2 300 4 500 600 700 800]
	fmt.Printf("Y = %v\n", Y)
	// output: Y = [300 400 500 600 700 800 900]
```

- A slice allocates new memory when an element is appended beyond its current capacity. If the capacity is less than 1024, it typically grows by a factor of 2; for larger slices, the growth factor is around 1.5. However, according to my experiment, it is not the case.

```golang
var test []int
oldCap := cap(test)
fmt.Printf("size = 0, capacity = %d\n", oldCap)

for i := 0; i < 100000000; i++ {
    test = append(test, i)
    if cap(test) != oldCap {
        fmt.Printf("size = %d, capacity = %d, ratio is %.3f\n", i+1, cap(test), 1.0*float64(cap(test))/float64(oldCap))
        oldCap = cap(test)
    }
}
```

```
size = 0, capacity = 0
size = 1, capacity = 1, ratio is +Inf
size = 2, capacity = 2, ratio is 2.000
size = 3, capacity = 4, ratio is 2.000
size = 5, capacity = 8, ratio is 2.000
size = 9, capacity = 16, ratio is 2.000
size = 17, capacity = 32, ratio is 2.000
size = 33, capacity = 64, ratio is 2.000
size = 65, capacity = 128, ratio is 2.000
size = 129, capacity = 256, ratio is 2.000
size = 257, capacity = 512, ratio is 2.000
size = 513, capacity = 848, ratio is 1.656
size = 849, capacity = 1280, ratio is 1.509
size = 1281, capacity = 1792, ratio is 1.400
size = 1793, capacity = 2560, ratio is 1.429
size = 2561, capacity = 3408, ratio is 1.331
size = 3409, capacity = 5120, ratio is 1.502
size = 5121, capacity = 7168, ratio is 1.400
size = 7169, capacity = 9216, ratio is 1.286
size = 9217, capacity = 12288, ratio is 1.333
size = 12289, capacity = 16384, ratio is 1.333
size = 16385, capacity = 21504, ratio is 1.312
size = 21505, capacity = 27648, ratio is 1.286
size = 27649, capacity = 34816, ratio is 1.259
size = 34817, capacity = 44032, ratio is 1.265
size = 44033, capacity = 55296, ratio is 1.256
size = 55297, capacity = 69632, ratio is 1.259
size = 69633, capacity = 88064, ratio is 1.265
size = 88065, capacity = 110592, ratio is 1.256
size = 110593, capacity = 139264, ratio is 1.259
size = 139265, capacity = 175104, ratio is 1.257
size = 175105, capacity = 219136, ratio is 1.251
size = 219137, capacity = 274432, ratio is 1.252
size = 274433, capacity = 344064, ratio is 1.254
size = 344065, capacity = 431104, ratio is 1.253
size = 431105, capacity = 539648, ratio is 1.252
size = 539649, capacity = 674816, ratio is 1.250
size = 674817, capacity = 843776, ratio is 1.250
size = 843777, capacity = 1055744, ratio is 1.251
size = 1055745, capacity = 1319936, ratio is 1.250
```

In C++, vector grows by 2x larger on my computer.

### array
- We can use T[...]{} to initialize an array, the size of which is deduced from the number of elements in curly brace.

- The sha256.Sum256([]byte) can generate a [32]byte array

- We can use == to compare two arrays but not slices

- When an array is used as a function input parameter, the function **deep copy** the original array. So mutation happened on the copied array doesn't go to the original input. Use *[]T to avoid copy.

## Map
- unlike in c++, we cannot take the address of a value inside a map because the map may rehash and invalidate the original pointer. This feature is really similar to the unordered_map in c++, which invalidate all previously returned iterators upon rehashing.

## Struct
- the following kind of initialization required every field to be offered a value. In other words, if some member is not exported, the following strategy cannot work.
```golang
type Point struct{ X, Y int }
p := Point{1, 2}
```

### Anonymous Fields
```golang
type Point struct {
    X, Y int
}

type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}
```
We say that a Point is embedded within Circle, and a Circle is embedded within Wheel.

Anonymous fields can be used to expose functions. For instance, if we define the following two struct in package *tools*, 
```golang
type secret struct {
	Sec int
}

func (s secret) Say() {
	fmt.Printf("secret reveals itself\n")
}

type Public struct {
	secret
}
```
then we cannot create an instance in package *main*. However, we can create an instance of Public, through which the member variable Sec and function Say() are available for use.

### Marshal and Unmarshal
json.Marsh() can convert a struct into a []byte. But a member must meet all of the following requirements to be marshalled:

- it is exported, that is, begin with an upper-cased alphabet
- it does not have a json tag `json:"-"`
- it belongs to: basic types such as string, int, bool, or slices, maps, structs, pointers

Exported anonymous embedded structs are not marshaled.

