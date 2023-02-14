# Reference/Sketch
## ui
* english, standardi(s/z)ed, ~2000, UK keyboard layout

## hardware
* pointers (flat address space)
* addressable element: (0-1)[8] #(byte)
* native numbers make up multiple bytes
* number types:
    * unsigned: u8/u16/u32/u64 (byte[1/2/4/8])
    * signed: i8/i16/i32/i64 (byte[1/2/4/8])
    * floating point: f32/f64 (byte[4/8])
```
named scalar:
.   8       16      32      64      address
u   byte    ushort  uint    ulong   usize
i   sbyte   short   int     long    isize
f/n u8n/i8n         float   double
```
## types
```
templates:
    `<? field ...>*`: pointer (with optional offset)
    `^`: resource / reference counted handle
    `[*]`/`[*: <cx:i>]`: array view
    `[<cx:i/t:enum, ...>]`/`[&/<t:enum>: <cx:i>]`: fixed size array
    `[<, ...>]`/`[: <cx:i>]`: array resource / heap array
    `.<tpl>`: custom template
    `(...)[ -> <t> ](*/^)`: function pointer/object

aliases:
    `<named scalar><2/3/4>`: `<named scalar>[<2/3/4>]`
    `number`: `float`
    `vector<2/3/4>`: `number[<2/3/4>]`
    `char/wchar`: `byte/ushort`
    `string/wstring`: `char[]/wchar[]`

special:
    `bool`: (0/1), doesn't have an address (?)
    `bool[<cx:i>]`: bool array (?), has an address if multiple of 8
    `<t:scalar> bool`: 0/1 (?)
    (field): `name`(constant string), `type`(type), `offset`(constant)

`<t:partial>`: `? [ restricted ] <n:t> <? tpl ...>`
    `? t2[*: ? d]`
    `? t2[&: ? d]` // (?)
    `? t2[: ? d]`
    `restricted`: disables all operators on argument and 
            allows using with private layouts
        
```
## nodes
* name/n
    * not allowed: `!"#$%&'()*+,-./ :;<=>?@ [\]^ {|}~`, backtick, control ch.
    * allowed: a-z, A-Z, 0-9, _, space, any character not listed above
    * multiple spaces count as one
    * variables can start with numbers (?)
* `\`: continue at any (positive) indentation instead of +2 or higher
* if depth is 1 or more then the node is not terminated by newlines
    * (when inside of `()`/`[]`/`{}`)
```
.ld: `[private/protected][ordered][packed]layout[ (<t:base>) ][ => (<x>) ]`

.f: `[ (<? p>) ](<? p, ...>)[ -> <t> ]/[ => <x> ]`
    `p`:
        `[ using ][ <n:1> | ]<n:2>[ ref ][: <x/t> ]`
        `[[ using ][ <n:1> | ]<n:2>[ ref ]: ] <t:partial>`
        `[ !<t> ]`

`<n/ns> [ *]:: [ private ]namespace`
`<n> :: platform[ (<plf:base>) ]"<n>"`
`<n>[: ] :: [ <t:i> ]enum[ (t:base) ]`
`<n>[: ] :: [ <t:u> ]flags[ (t:base) ]`
`[<field:enum> &]/[<n> .]<n>[: ] ::[ ? .]<.ld>`
`<n/"n"/op.> [ *]::[ ? ] <.f>`
`<n/"n">: <x/t>`
`<n> #:[ ? ] <cx>`

`[<n> .]<l> :: !<plf> .<.ld>`
`<f> ::[ !<plf> ] <.f>`
`<c> #:[ !(<plf>) ] <cx>`

`<x>`: expression
`if`, `else if`, `else`
`if& <x> <b.(op/f)>`
`for[ each ]<n>[ at <n> ]in <x:range>/<x:view>/<t>[ ## <n:label> ]`
`while <x>[ ## <n:label> ]`
`repeat <x>[ ## <n:label> ]`

`override (<field>): <x/t>` (?)

`case <x/t:partial, ...>[ => <x> ]`: in selection if
`case <cx, ...> [ => <n & ...>: <(x/t, ...)>]`: in layouts

`<n>[ #: <integer> ] , ...`: in enum and flags
`<n>[ @<integer> ] , ...`: in flags

`<n> #: {...}`: constant constructor (in <l>)
`<n> ':: (...) -> <t>`: named constructor, t is layout's name (in <l>)
`~ ':: (ref ...)`: destructor (in <l>)

`using <...>`: on fields (or on any variable (?) (whichever is easier))

`<library documentation>` (in global/unordered scope) (?)
    text that doesn't contain `:` unless `\:`
    `\(<any>)`: reference
    `\("<n>")`: external reference (for images)
    applies to the next function/layout/constant
        has to be within two lines away to be associated with a node (?)
    (can be chained)
        (as in you can have blank lines in the text)

//`(ref <x>): <x>` (?)
`ref <x>: <x>` (?)
```
## operators
```
+	-	*	/	%	^	<<	>>	// binary maths
+	-	*	@	^	not			// prefix (`^`: reference copy/alias)
<	>	<=	>=	=	/=	==	/==	// comparisons
`++ --`(?), `[]`(built in indexing starts at 1)
<-+	<'-	<-*	<-/
<-(<<)	<-(>>)	<-(>&)	<-(<&)	// (?)
and	or
is, as, to, contains
`<-`, `<->`
`'s`/`'`/`of`
`?'s`/`?'` (?)
`|`/`&`: binary repeater (on any expression that returns bool)
`&`: flags combine (bitwise or)
`? ||`: inline `if`/`else if`/`else`
`>&`, `<&`
`<x/t> <b.(op/f)> &{<x/t:partial>/default => <x> , ...}`

`<x:flags> <-+/<'- <x:flags>`
`<x:flags> contains <x:flags>`
`<x> to <x>` // range (t[2])
`<x> to <t>` // conversion/copy
`<x> as <t>`
`<x:union> is <t:union>`
`<t> is <t>`

can be overrided: maths, <, >, =, @, [], ^, <-+, <'-, <-*, <-/

`/`: always returns float/double/normalized, not integer
`%`: remainder
`^`: power
`==`: compares bits, can't be overrided
`*`: pointer to, can't be overrided
`@`: dereference, can be overrided
`<-`: destruct + assign (`:`)
assign: unnamed: (move), named: `to`
`<->`: swap
`>&`: max, `<&`: min (uses `>` and `<`)
`as`: reinterpret
`>=`/`<=`/`/=` use `<`/`>`/`=`

(all arrows can be reversed)
```
## subexpressions
```
`<f/x:f>[ ... ]` (function call)
    `(<a, ...>)`, `a`: `[<p[1]/p[2]>: ][ ref ]<x>` (requires ref if reference)
    `<a ...>`, `a`: `[<p[1]> ]<x>` (no ref even if parameter is reference)
    can have both named and unnamed parameters in the same call
    named parameters can be in any order (and have to come after unnamed ones)
    if function pointer: `!` can be used in front of value with pointer (?)
        `function (!object, ...)`: `object's function (object, ...)`
array indexer optional (?)
    `a [i]`: `a i`
    `a i + b (i + 1)`
`external (<p, ...>)[ -> <t> ] <cx:s> in <cx:s> [ on <plf> ]`
    `external <t> ...`
`break [ <label> ]`
`continue` (?)
    or `skip/next/` (?)
initializer
    `{...}`: complete and ordered, no default
    `'{...}`: incomplete or unordered, no default
    `~{...}`: uninitialized, no default
    `"..."`: if char/wchar array ([]/[&]/[*]), default: char[&]
    arrays: can use indices instead of fields for named parameters
        if length is an enum then the values of the enum can be used
`[ <t> ]<initializer>`
`new <t>[[<...>] ][ <initializer> ]`
`new !<t>[ <initializer> ]`
`[]/[&]/[*] [ <initializer> ]`: array with inferred type, default: number
`new[ with (<x:allocator>) ] ...`(?)
`true`/`false`
`null`: default: `unknown*`
`<(p[: <t> ], ...)> => <x>`/`<x*>`: lambda / function object
    `x => x > 5`: `!x > 5`
    typed parameters only allowed with parentheses
`*~struct`: destructor
`*<f>`: function pointer
`*(<f>)`/`*<f> ()`: pointer to value returned by function
`<x> to <x>`: range (`t[2]`)
    `[ first ]1 to 5`: {1, 5}
    `first 5`: {1, 5}
    `last 5`: {-1, -5} // or reverse it (?)
    `last 1 to 5`: {-1, -5} // (?)
`<x> #<(n)>`: tag / inline assignment (to reference)
    (doesn't require parentheses for multiple words)
`(<x>)`

`<number...>[.<number...> ]( [ K/M/G/T][i] )/( <t:iuf> )/U`
    `0i32`, `0U`(usize) (?)
        or no suffix type (?)
    `usize {10Ki}`
`"<? text>"`, `\n`, `\t`, `\0`, `\\`, `\(<x>)`
    string containing `\(<x>)` returns `char[]` instead of `char[*]` (?)
    multiple line strings
        indentation of the first non-tab character is used
            `\t` doesn't count (it's the same as any non-tab character)

references
    starts searching in current node and goes up the hierarchy
    conjugation
        function and variable names
        `is/has/was/will/can / s/ies/(ss/sh/ch/o/x es)`
            `isn't`/`is not`, ..., `doesn't -s/(-ies+y)/-es`/`does not ...`, ...
        functions that end with `of` are used like fields
        fields that start with `is/...` are used like functions
        flags with `using` automatically have `is` added at the start (?)
            (when accessed through parent layout)
            (unless it can already be conjugated (?))
```
## rules/behaviour
```
`<x>`: `[ <prefix> ]<value>[ <suffix> ][ <binary> ... ]`
multiple identifiers in a row:
    picks the one with the most matching words from the start
```
* operators allowed on type parameters (when they're not private)
    * default construct, swap/move, destruct, copy/convert (using `to`), 
        `*x`, `type of x`, `x as <t>`
* `t* + t*`: invalid (?)
* `t* + int`: does *not* automatically multiply by the size of t
* namespaces in names are optional, only used for clarification
    * (`using` can't be used on namespaces or outside of layouts and parameters)
* untyped values in parameters and layouts are `number` by default
* global initializers can't reference other globals (variables or functions)
        that end up referencing back to itself
* can't get pointer to local function
    * (but can convert to function objects)
* function arguments aren't copied
    * (or they might be but passing a value doesn't call `to`)
* functions parameters are shallow immutable
    * can't get pointer to shallow immutable values
* tags are references, identifiers using `<n>:` are values
        (uses `to`/move when declaring)
* struct returned by `return` resides at pointer passed by caller
    * either named or temporary value
* function call parameter mapping: 1d array views are expanded (?)
* when overriding a field the type has to have the same size
    * so both layouts have to be `ordered` (?)
* types of casts can be inferred
    * (using `to` or just assigning)
* flat scopes (`*::`) have to be at the top (excluding comments)
* `<n>: <t> <- <x>` is *not* valid
* can't create template instance when the argument contains a private layout
* imperative functions at the start of a line are always evaluated last
    * except for assigning (right arrow) (?)
* argument to prefix declarative with single parameter can't be followed
        by a binary operator if it's at the start of a line

## unsorted
```
can't initialize values of type with `^` by type (?)
    have to add null
        value: int[null]
        value: int null^
        value: int null .container

    array: int[] // invalid if global/local (?)
    array: int[null] // always valid

value: (...) :: (...) -> <t> // compound declare and call/assign

`parameter: ? restricted ref t^` instead of `parameter: ref ? restricted t^` (?)

`current platform`
`<plf> is <plf>`

parentheses around expression instead of platform for constants (?)
    or don't have platform specific constants (?)


`;`: multiple expressions on the same line

for (x, y) in positions

array initialization can combine arrays (?)
    float4 {float3 {1, 2, 3}, 1}

enum values are automatically used when assigning to an enum
    some color <- red // same as `some color <- color red`
    also constructors

waves
    `[<x:view>]`, `%%`, `==>`, `at`, `%_`: + * and or (?)
    `all/either/sum/product of` (?)
    `for item in [list]`
    `for i in [1 to {1920, 1080}]
    `$[...]`: serial / inline for
`%% :: (<wave>) (function: (a: ref ? t, b: t)(^)) -> t`
`%% :: (<$wave>) (first: (a: ? t) -> ? t2 (^), other: (a: ref t2, b: t)(^)) -> t2`
`[100 from source] ==> dest`
`[{10, 10} to {500, 500} from texture] ==> dest at {200, 200}`
`[100 from pointer] <- 1`
`if all of [list] = 0` (?)
`if ([list] = 0)%and` (?)
`[1 to {10, 10} from u8n4[*: 2] {pointer, {256, 256}, 256}] ...`
`[callbacks] {1} ()` // 1 lane per thread
`[first] = [second]`
`int ([first] and [second])` bitwise and

lock, atomic, threads (?)
    referencing an atomic value locks it until the end of the expression
    `lock <x:atomic>`: manual lock (for an extended duration)
    `<n>: atomic <t>`: any type, layout that it's inside of can't be ordered
    built in functions for threads (?)

    thread :: ? private layout // (?)
    start thread :: ? (f: (p: usize)(*), arg: usize, stack size: usize) -> thread^
        or take function object (?)
    wait for any thread :: ? (list: thread^[*])
    is done :: ? (thread: thread^) -> bool
    max current threads #: ? 0
```
## ?
```

commas always optional when the next character is a newline (?)

unnamed structs (?)
    `a: {...}`

C for loop (?)
    for i: dictionary's start, i /= dictionary's end, i++


index types / context accessor / index association (?)

    structure :: layout
        values: int[]
        items: [] :: layout
            value index: 0 ## values [!index] // has to end with index (?)

    struct: deserialize (structure, read bytes from "file.bin")

    for item in struct's items
        a: value // int
```
## built in
```
main :: ()

t[&] + t[&] -> t[&] // returns same length array
t[]/t[*] + t[]/t[*] -> t[] // glues/combines arrays (not zip)

t[&] + t[]/t[*] -> t[]
t[]/t[*] + t[&] -> t[]

(native) to bool -> bool
bool to (native) -> (native (0/1))
t[]/t[&] to t[*] -> t[*] // any dimensions


rounded up/down
truncated

to :: (a: int[? d]) (b: int[d]) -> int[d][2]
from :: (range: int[? d][2]) (list: ? t[*: d]) -> t[*: d]
from :: (count: int) (list: ? t[*]) -> t[*]
where :: (view: t[*]) (condition: (a: t) -> bool (^)) -> t[]

data pointer of :: () (array: ? t[: ? d]) => header's view's pointer
contains :: (array: ? t[*: ? d]) (item: t) -> bool
<-+ :: (array: ? t[]) (item: t[*])
<-+ :: (array: ? t[]) (item: t) -> ref t
<'- :: (array: ? t[]) (item: t) -> ref t


length of :: () (view: ? t[*: 1]) -> ref usize
area of :: () (view: ? t[*: 2]) -> ref usize2

length of :: () (view: ? t[: 1]) -> ref usize
area of :: () (view: ? t[: 2]) -> ref usize2

reallocate :: ? (address: unknown*, size: usize) -> unknown*
exit :: ? ()
assert :: ? (value: bool, message: char[*]) // (?)
```
## layouts
```
t[*: ? d] :: layout
    pointer: t*
    volume: usize[d]
    step: usize[d - 1]

allocator :: layout ((address: unknown*, size: usize) -> unknown* (^))

t^ :: protected layout => (header's data)
    header :: layout
        reallocate: allocator
        destruct: (a: ref t)(*)
        reference count: usize
        data: t*
    header: header data*
    ~ ':: (...)

t[: ? d] :: protected layout
    header :: layout
        reallocate header: allocator
        reallocate data: allocator
        destruct: (a: ref t)(*)
        element size: usize // (?)
        reference count: usize
        capacity: usize
        view: t[*: d]
    header: header view*
    ~ ':: (...)

(...)(^) :: protected layout
    pointer: (..., object: (...)(^)* )(*)
    data: usize[2]
    extra data: unknown^

```

# examples
## drawing
```
some image: load image "image.png"

main :: ()
    window: open window // (has other parameters that are set to default)
    while window's close button hasn't been pressed
        clear {0.25, 0.25, 0.25, 1}
        draw image some image at {10, 10} multiplied by red
        present // vsync enabled by default

// opening a window when there isn't already one open sets the current surface
```
## imperative vs prefix declarative (?)
```
sin :: () (a) -> number
print :: (text: string)

sin 4 // valid
sin 4 + 4 // invalid
print 4 + 4 // valid (same as `print (4 + 4)`)
sin (4 + 4) // valid

sin 4 -> other

```
## platform specific
```
file :: ? .private layout

open file :: ? (name: string) -> file^


file :: !windows .private layout
    ...

open file :: !windows (name: string) -> file^
    ...
```
## unions/layouts
```
t &struct :: layout
    t: :: enum
        first, second, third // first is 1, unknown is 0
    common value: u32
    case first
        constant subsubtype #: {common value: 1, first specific: -1}
        subsubtype ':: (parameter) -> struct
        first specific: 1
    case second => a & b: (u16, u32)

structure: first struct
structure's first specific <- 2


structure: unknown struct {common value: 10}

other: structure to first struct
other's first specific <- 2

```
## serialize, views, type selection if, partial types
```
serialize :: (value: ? ref t, output: byte[])
    for field in t
        if& field is
            case ? t2[: ? d]
                ...
            case ? t2[&: ? d]
                for each element in field
                    serialize (ref element, output)
            case ? t2[*: ? d] // don't include
            case ? t2* // don't include
            default
                output <-+ byte[*] {
                        *value + field's offset, size of field's type}
```
## reference copy
```
file: open file ("file.bin") // `file` is `file^`

some global variable <- ^file // valid

//some global variable <- file 
// invalid, `t^` doesn't have a copy/`to` operator (with private specified)
```
## field using, override, function pointer call
```
interface :: layout
    using vtable: null // unknown*


struct vtable: :: layout
    set value: (in | object: interface*, to | value: int) -> float (*)

struct :: layout (interface)
    override vtable: *struct vtable


get interface: *external (...) -> struct^ "get interface" in "library"

main :: ()
    ...
    object <- get interface (...)

    result: set value (!object, 10)

    result: object's vtable's set value (object, 10)

    result: set value to 10 in !object
```
## tags, error handling
```
if get info (0, '{} # output) succeeds
    ...
    print output's text
else
    assert false
```
## flags
```
struct :: layout
    using flags: :: u8 flags
        active @1, ...

object: new struct

if object is active

object's flags' active // (?)

...
    state: :: uint flags
        first @1
        second @2
        combined #: first & second
        third @3

object's state <- first & third
```
## flat scope (`*::`)
```
// start of file

a *:: namespace
b *:: namespace // can also combine into `a b *:: namespace`
main *:: ()

print "e" // no indentation
```
## misc.
```
print 10 + 10 // same as `print (10 + 10)`
if do something 10 + 10 // same as `(do something (10)) + 10
text: value is in range &{1 to 10 => "1 to 10", default => "??"}
`parameter: ? t[: ? d][*: ? d2]`
```

# scrapped
## built in serialization, modifiers in structs
```
operators: `<-$`, `$`, `${}`

...
    ...
    relocations: [$u16] :: layout
        local address RVA: u32
        target: t& target :: layout
            t: :: enum $u1
                local #: 0, external #: 1
            case local => local RVA: u32 $u31
            case external => link & fragment: (u16 $u15, u16)
    sections: [$u8] :: layout
        name: char[$u8]
        properties: :: u8 flags
            initialized #: 1, shared #: 2, write protected #: 3
        page aligned dest RVA: u32
        size: u32 $if not initialized
        data: byte[$u32] $if initialized
```
## traits
```
scalar : integer : comparable : equatable

wbuffer :: trait
    wbuffer <-+ byte[*] // (?)
    write bytes (from | source: byte[*], to | destination: wbuffer)

t? .[indexable: d #?] :: trait
    indexable[usize[d]] -> ref t
    ...

t? .iterable :: trait
    ...

sort :: (value: t? (.indexable: t2? comparable))
sort :: (value: t[] where t? comparable)
<-$ :: (dest: t? wbuffer) (source: t? any) // built in

wbuffer <-$ struct

```
## dictionaries (both key and value types), custom nd arrays/views
```
`.[<tpl>: <t>]`: custom/named dictionary
`.[<tpl>]`/`.[<tpl>: <cx:i>]`: custom/named multiple dimension container

dictionary: int[: string] // string to int
custom dictionary: int .[custom: string]
custom array: int .[custom: 3]
```
## captures
```
global: `&(...)`
    allows referencing other globals inside global initializers
local: `+(...)` (before global capture)
    allows getting pointer to local function
```
## $ scopes
```
$ instead of {}

object: $
    field: $
        a: 0
        b: 10

() => $
struct $
function $
char[&] $

identity #: $
    {1, 0, 0, 0}
    {0, 1, 0, 0}
    {0, 0, 1, 0}
    {0, 0, 0, 1}

value: $
    field: 10
    second: 20
    function: $ // function object
        image: load image "sprite.png"
        do something
        ...
        return image
    other: $ // string
        some text
        askfejsek fjskela fj
        as fkljsa efjsa kjlfsaekfj 
        fsaklefjesaf
```
## platform agnostic number declarations
```
(nothing final)

number :: +-1K % 0.0001 to 1e30, +-10M

u8 :: 0 to 255
u16 :: 0 to 2^16 - 1
u32 :: 0 to 2^32 - 1
u64 :: 0 to 2^64 - 1

float :: +-~1e30 (1e-6)
coordinate :: +-100M (0.00001)

```
## splitting up arguments
```
function :: (x, y, ...)

function (%position, ...)
function ((x, y): %position, ...)
x, y: %position
```
## `using` anywhere
```
using object
    set value (!, 10) // inferred function pointer location

using* something
...
```
## misc.
```
`if 5 < a < 10`
`embed <t>[&] "<n>"`
```

