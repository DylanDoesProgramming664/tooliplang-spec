# ToolipLang Specification

## Reserved Words

```ruby
and break const defer do else
elif end enum false for match
if in infer local next nil not
or proc return static struct
then trait true typedef union
undefined volatile while xor
```

## Operators

```ruby
'+' # plus
'-' # minus
'*' # multiply
'/' # divide
'%' # modulus
'&' # and
'|' # or
'~' # not
'^' # xor
'..' # concat
'...' # range exclusive infix
'...=' # range inclusive infix
'??' # nil coalesce
'**' # power
'//' # floor div
'<<' # bit shift left
'>>' # bit shift right
'!' # error try (suffix)
'?' # option unwrap (suffix)
'++' # increment
'--' # decrement
'==' # equals
'<' # less than
'>' # greater than
'<=' # less than or equal
'>=' # greater than or equal
'~=' # not equal
'=' # assign or reassign
'+=' # reassign plus
'-=' # reassign minus
'*=' # reassign multiply
'/=' # divide assign
'%=' # modulus reassign
'&=' # and reassign
'|=' # or reassign
'^=' # xor reassign
'..=' # concat reassign
'??=' # nil coalesce reassign
'**=' # power reassign
'//=' # floor div reassign
'<<=' # bitshift left reassign
'>>=' # bitshift right reassign
```

## Types

```ruby
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
^type # pointer type 
unique[type] # unique pointer
shared[type] # shared pointer
allocator # allocator
...type # variadic of a type
proc # procedure type
```

## Delimiters
```ruby
_ # unnamed variable; ignores the value given to it
. # separator for fields of a type such as structs. `type.fieldIdent type.fieldProcIdent(args)`
: # separator for a field of an ident of a ProcValue. `var:fieldProcIdent(args) == typeOfVar.fieldProcIdent(^type self, args)`
; #optional early statement terminator
, # separator for compound type values, procedure args, and multiple variables or values on the same line
[ | ] # used for array, compound type and proc return type declaration only
| # used for lambdas
{ and } # used for the values of complex types such as arrays, lists, maps and tuples and struct literals.
( and ) # for procedure declaration args and function call args
```

## Variable Declarations and Procedure definitions
```ruby
uint answer = 42 # explicit typing
infer three = 3 # infered compile-time type
const float64 pi = 3.141592
static []char yoMama = "is fat"
local bool amIToolipYet = false

#Generics with the math.Add trait are being used to allow for int args to result in returning an int, while flt64 args return a flt64.
proc[T] add[T[math.Add]](...T nums) # the generic and its constraining traits are declared between the proc name and parameters
    infer sum = T.default # default propogates the default value of T.
    nums:foreach(|num| out += num)
    return sum
end
```

## Type Aliasing, Structs, Traits, Unions, and Enums
```ruby
typedef intList = list[int]

# traits are ToolipLang's interfaces, inspired by Rust and Go
trait Shape
    proc[flt64] perimeter(^T[Shape] self)
    proc[flt64] area(^T[Shape] self)
end
    
struct Rectangle[Shape]
    flt64 length = 1 # an explicit default value
    flt64 width # has flt64's implicit default value of 0.0. You can prevent defaults by assigning a variable to undefined
    
    proc[flt64] perimeter(^Rectangle self)
        return 2*self.length + 2*self.width
    end
    
    proc[flt64] area(^Rectangle self)
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

enum Days # may specify an enumeration type doing enum[enumType]. defaults to uint8, which starts at 0 by default
    Sunday = 1, # an explicit starting value
    Monday, # implicitly given the value of two, but may be explicit if you need to allow only specific values. Must be all ascending or all descending
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
end

# you can have enum[union,enumType] for tagged unions. enum[union] defaults to uint8 enumeration
enum[union] JSONElement
    Number number,
    []char str,
    bool boolean,
    []^JSONElement array,
    map[string]^JSONElement object,
end
```

## Control Flow and Pattern Matching

```ruby
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

## Core Libraries for ToolipLang
These are libraries and core features and libraries of ToolipLang
```ruby
compiler # includes token.tlp, lexer.tlp, parser.tlp, ast.tlp, types.tlp, ir.tlp, optimizer.tlp, codegen.tlp, mockRepl.tlp, compMacros.tlp, builtins.tlp
asm # includes x86.tlp, arm.tlp and riscV.tlp
mem # basic memory management utilities
endian # stop using big endian, but I'll cover that shit for you because I like you. Baka >_<
```

## Standard Libraries for ToolipLang
Below are libraries that may be used without assigning local variables for an include builtin procedure call.
With that said, Toolip only brings in a library from stdlib if explicitly used in a Toolip file being ran or built, or if an included library uses that global library.
You are only required to explicitly include libraries outside of ToolipLang's standard library
Each and every library in standard will be as independent of one another as possible, mainly relying on the core of the language to flesh out the library features
```ruby
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
net # net utilities, such as TCP, UDP, and sockets
simd # simdeez nuts
```

## Utils Libraries
These are additional libraries for toolip that are not part of the standard library, yet are nice libraries to have in your back pocket.
Unlike standard libraries, these must be brought in through the include builtin.
```ruby
parsers/(json, xml, html, css, csv, toml) # parsers for handling common filetypes
web/(http, wasm, wasi)
runtimes/(gc, arc) # allows you to opt into a default garbage collection or automatic reference counter for specified regions of your source code. Useful for writing higher-level procedureality for binaries, but I strongly suggest not thrusting these memory models upon developers through libraries. It is best if each developer has the final say in how memory is managed in their codebase.
generators/(tree_sitter, antlr, lemon)
embeddedPLs/(lua)
```

## Include Structure
```ruby
local typedef json = include("parsers/json") # is a struct of the public json.tool contents.

local infer alloc = Allocator.init()
defer alloc:deinit()

local string pi_str = json.stringify(3.141592)
local flt64 pi_double = json.parse(flt64, alloc, pi_str)
```

Oh, and types are technically libraries. Or, more accurately, all libraries are structs, and so are your standard primitive types. So, the string, byte, char and bool types all have global procedures to use.

## Type Casting and Transmuting
```ruby
uint64 piEngineer = uint64.cast(flt64, math.pi)! # hahah. funny joke

assert(piEngineer == 3) # assert either crashes or moves on

infer eulerConstStr = fmt.parseFloat(flt64, math.e, 6)! # The 6 tells the formatter to cut the float string short after 6 decimal places.

assert(eulerConstStr == "2.718281")

uint64 piRaw = uint64.transmute(flt64, math.pi) #Turns into its IEEE 754 representation as a uint64

assert(fmt.ParseInt(uint64, piRaw, 16) == "0x400921fb54442d18")
```

## Example HTTP server
```ruby
# Inspired by Crystal's front page example. I <3 Crystal btw. If you're reading this, please get Crystal's tree-sitter in the official nvim-treesitter repo.
local typedef http = include("web/http")

proc[!void] main()
    local infer server = http.Server.init(|context| do
        context:response:contentType = "text/plain"
        context:response:print("Hello Web from Toolip")
    end)

    local infer port = 8080
    local infer addr = server:bindTCP(port)! do #'!' propogates errors, and allows an |errIdent| do code end block after to customize error handling.
        io.write(io.stderr, "Could not bind TCP to port ${port}.\n")
        return
    end
    print("Listening on http://${addr}\n")
    server:listen()! #'!' without a block will just panic if something goes wrong unless the ident or procedure call has custom error handling
end
```

