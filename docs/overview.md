# ToolipLang Specification

## Reserved Words

```crystal
and break const defer do else
elif end enum false for match
if in infer local next nil not
or proc return static struct
then trait true typedef union
undefined with while xor
```

## Operators

```crystal
+ # plus
- # minus
* # mult or pointer
/ # divide
% # modulus
& # and
| # or
~ # not
^ # xor
.. # concat
... # range exclusive infix
...= # range inclusive infix
?? # nil coalesce
** # power
// # floor div
<< # bit shift left
>> # bit shift right
! # error try (suffix)
? # option unwrap (suffix)
++ # increment
-- # decrement
== # equals
< # less than
> # greater than
<= # less than or equal
>= # greater than or equal
~= # not equal
= # assign or reassign
+= # reassign plus
-= # reassign minus
*= # reassign multiply
/= # divide assign
%= # modulus reassign
&= # and reassign
|= # or reassign
^= # xor reassign
..= # concat reassign
??= # nil coalesce reassign
**= # power reassign
//= # floor div reassign
<<= # bitshift left reassign
>>= # bitshift right reassign
```

## Types

```crystal
int # isize in Zig and Rust
uint # usize in Zig and Rust
flt32 # f32 in Zig and Rust
flt64 # f64 in Zig and Rust
flt128 # f128 in Zig and Rust
uint8
uint16
uint32
uint64
uint128 # u128 in Zig or Rust
uintXX # XX is a placeholder for arbitrary bitwidth unsigned integers like Zig
int8
int16
int32
int64
int128 # i128 in Zig or Rust
intXX # XX is a placeholder for arbitrary bitwidth signed integers like Zig
bool # boolean
char # unicode codepoint. Equivalent to rune in Go or char in Rust
byte # ascii codepoint. Is really just a uint8
[]type # a static array of a specified type
list[type] # a dynamic array of a specified type
map[keyType]valType # a hashed associative array of a specified keyType and valType
set[keyType] # hashset
tuple[type1, type2, ...] # tuples can store a fixed number of typed values specified or infered at declaration
# tuples are really just anonymous structs with numbered keys
void # has no value
error # an error type
struct
enum
union
?type # nilable type
!type # error union type; stderr by default, but can be specified with custom error sets
ptr[type] # pointer type 
unique[type] # unique pointer
shared[type] # shared pointer
allocator # allocator
...type # variadic of a type
proc # procedure type
```

## Delimiters
```crystal
_ # unnamed variable; ignores the value given to it
. # separator for fields of a type such as structs. `type.fieldIdent type.fieldProcIdent(args)`
: # separator for a field of an ident of a ProcValue. `var:fieldProcIdent(args) == typeOfVar.fieldProcIdent(ptr[type] self, args)`
; #optional early statement terminator
, # separator for compound type values, function args, and multiple variables or values on the same line
[ and ] # used for array, compound type and proc return type declaration only
{ and } # used for the values of complex types such as arrays, lists, maps and tuples and struct literals.
( and ) # for function declaration args and function call args
```

## Variable Declarations and Procedure definitions
```crystal
uint answer = 42 # explicit typing
infer three = 3 # infered compile-time type
const float64 pi = 3.141592
static []char yoMama = "is fat"
local bool amIToolipYet = false

#Generics with the math.Add trait are being used to allow for int args to result in returning an int, while flt64 args return a flt64.
proc[T[math.Add]] addNums(...T[math.Add] nums)
    infer sum = T.default # default propogates the default value of T.
    nums:foreach(|num| out += num)
    return sum
end
```

## Include Structure
```crystal
local typedef json = include("parsers/json") # is a struct of the public json.tool contents.

json.stringify(flt64, 3.141592)
json.parse("myJSON.json")
```

## Type Aliasing, Structs, Traits, Unions, and Enums
```crystal
typedef intList = list[int]

# traits are ToolipLang's interfaces, inspired by Rust and Go
trait Shape
    proc[flt64] perimeter(ptr[T[Shape]] self)
    proc[flt64] area(ptr[T[Shape]] self)
end
    
struct Rectangle[Shape]
    flt64 length = 1 # an explicit default value
    flt64 width # has flt64's implicit default value of 0.0. You can prevent defaults by assigning a variable to undefined
    
    proc[flt64] perimeter(ptr[Rectangle] self)
        return 2*self.length + 2*self.width
    end
    
    proc[flt64] area(ptr[Rectangle] self)
        return self.length * self.width
    end
end

local infer rect = Rectangle { length = 4, width = 5 }
local infer perim = rect:perimeter()
local infer area = rect:area()

union Number
    int64 integer
    flt64 double
end

local infer funny_num = Number.integer(69420)

enum Days # may specify an enumeration type doing enum[type]. defaults to uint8
    Monday = 1, # an explicit starting value
    Tuesday, # implicitly given the value of two, but may be explicit if you need to allow only specific values. Must be all ascending or all descending.
    # the rest
end

# you can have enum[union] for tagged enums.
```

## Control Flow and Pattern Matching

```crystal
if condition1 then
    # do something
elif condition2 then
    # do something else
else
    # catchall
end

for int i in 1...=10 do
    print("${i}\n")
end

for int i = 1, i <= 10, i++ do
    print("${i}\n")
end

int i = 1
while (i <= 10) do
    print("{i}\n")
    i++
end

int i = 1
do
    print("{i}\n")
    i++
while (i <= 10)

match funny_num
    .integer {x} => print("integer = ${x}\n"),
    .double {x} => print("double = ${x}\n"),
end
```

## Global libraries for ToolipLang
Below are libraries that may be used without assigning local variables for an include builtin function call. With that said, Toolip only brings in a global library if explicitly used in a Toolip file being ran or built, or if an included library uses that global library.
```crystal
io # input/output utilities and types
fmt # formatting utilities and types
fs # filesystem utilities and types
os # os-specific utilities and types
process # os-agnostic utilities and types
math # math utilities, types and traits
ascii # ascii utilities
utf8 # utf8 utilities
debug # utilities that only run in debug mode
testing # test utilities
```

Oh, and types are technically libraries. Or, more accurately, all libraries are structs, and so are your standard primitive types. so, the string, byte, char and bool types all have global procedures to use.

## Type Conversion
```crystal
uint64 piEngineer = uint64.cast(flt64, math.pi)! # hahah. funny joke

assert(piEngineer == 3) # assert either crashes or moves on

infer eulerConstStr = fmt.parseFloat(flt64, math.e, 6)! # The 6 tells the formatter to cut the float string short after 6 decimal places.

assert(eulerConstStr == "2.718281")
```

## Example HTTP server
```crystal
# Inspired by Crystal's front page example. I <3 Crystal btw. If you're reading this, please get Crystal's tree-sitter in the official nvim-treesitter repo.
local typedef http = include("web/http")


proc[void] main()
    local infer server = http.Server.init(|context| do
        context:response:contentType = "text/plain"
        context:response:print("Hello Web from Toolip")
    end)

    local infer addr = server:bindTCP(8080)
    print("Listening on http://${addr}\n")
    server:listen()
end
```

