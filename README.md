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
f/n u8n/i8n         float   double  fsize
```
## types
```
templates:
    `<? field ...>*`: pointer (with optional offset)
    `[^]`/`[^: <cx:i>]`: array view
    `[<cx:i/t:enum, ...>]`/`[<? t:enum>: <cx:i>]`: fixed size array
    `.<tpl>`/`<tpl>`: custom template
    `.vector<2/3/4>`/`vector<2/3/4>` (?)
    `(<p...>)[ -> <t> ](*)`: function pointer

aliases:
    `number`: `fsize`
    `vector<2/3/4>`: `fsize .vector<2/3/4>` (?)
    `<named scalar><2/3/4>`: `<named scalar> .vector<2/3/4>`
    `grapheme`: `uint`

special:
    `bool`: (0/1), doesn't have an address (?)
    `<t:scalar> bool`: 0/1 (?)
    `textblock`: `byte[^] of`, `size of`, `start of`, `end of`, `length of`
        can't assign directly to local/global
        start/end are for graphemes
    function: `byte[^] of`, `size of`, `*`(function pointer)
        can't assign directly to local/global
        can't be passed as value (only as function pointer)
    type: `size of`, `name of`
    field: `name of`, `type of`, `offset of`
    charset: `length of`

partial:
    `(? t[: <t/tr.>, ...])`
    `[: ? d]`
    `[^: ? d]`
    `[? length]` (?)
    traits:
        scalar <- integer <- unsigned <- comparable <- equatable <- solid
        (no custom traits)

```
## nodes
* name/n
    * not allowed: `!"#$%&'()*+,-./ :;<=>?@ [\]^ {|}~`, backtick, control ch.
    * allowed: a-z, A-Z, 0-9, _, space, any character not listed above
    * multiple spaces count as one
    * variables can start with numbers (?)
* `\`: continue at any (positive) indentation instead of +2 or higher
* node isn't terminated by newline when inside of () [] {}
```
parameter/p
    `[ <n> | ]<n>:[ ref/move ]<t/x>`
    `[ <n> | ]<n>: *` (?)
    `[[ <n> | ]<n>: ] ? [ platform ][ ref/move ] <n> <tpl...>`
    `!<t>`

.ld: `[ ? .][private][ordered][packed]layout[ (<l:base>) ][ => <...> ]`

`<n> [ *]:: [ <charset> ]textblock`
`<n> [ *]:: charset[ (<base>) ]`
`<n> :: platform[ (<plf:base>) ][ "<n>" ]`
`<n> :: [ ? .]constant (<cx>)`
`<n/ns> [ *]:: namespace`
`[<n> .]/[<field>/unknown &]<n> [ *]:: <.ld>`: l/tpl
`<n>[: <? tpl...>] :: <.ld>`
`<n>[: <? tpl...>][ *]:: [ <t:integer> ]enum`
`<n/op/"n"> [ *]:: [ ? ][ (<p>) ](<p...>)[ -> <t> ]/[ => <x> ]`: f
`<n>:[ ref/move ]<x/t>`: local/global
`[using ]<n>[: <x/t>] , ...`: field(s)
`override (<field>): <x/t>` (?)
`"<n>": <x/t>`
`<variable/field>: atomic <t>` (?)

`<f> :: !<plf> [ (<p>) ](<p...>)[ -> <t> ]`
`<l> :: !<plf> .layout[ (<l:base>) ]`
`<c> :: !<plf> .constant (<cx>)`

`~ :: (<n>: ref <t>)`: destructor
`( <n>: <cx> ), ...`

`<library documentation>` (in global/unordered scope)
    text that doesn't contain `:` unless `\:`
    `\(<any>)`: reference
    `\("<n>")`: external reference (for images)
    applies to the next function/layout/constant
        (when there's 1 or 2 lines in between (?))

```
### inside functions
```
if <x>, else if <x>, else <x>
`if& <x/t/plf/charset> <binary f/op>`
`case <x/(t:partial)/plf/charset>[ => <x> ]`
while <x>[ ## <n:label> ]
repeat <x>[ ## <n:label> ]
for <n> [at <n>] in <x:view>/<x:range>/<x:iterable>/<t>[ ## <n:label> ]
    `for {a, b} at [x, y, z] in <x:view>`
for <n>: <x>, <x:bool>, <x>[ ## <n:label> ]
`using[* ] <type/namespace/platform/charset/x>`
`ref <x>: <x>`: assign to uninitialized variable at address
```
## operators / functions
### order
```
(function/parameter)
^
<< >>
* / %
+ -
>&, <&
as, to, (named: non-bool (?))

<  >  >=  <=
=  /=  ==  /==
is, (named: bool (?))

|, &
and
or
? ||

<-, <->, <-+/..., <==

suffix:
first: `[...]`, `(...)`*, `{...}`, `'s`
last: `++`, `--`, (named)

prefix:
+ - not @ * ^ of, (named)

`'s` and `of` also include properties

special:
`break[ <label> ]`
`continue/skip/next` (?)
```
###
```
`<x> to/as <t>`
`<x:union> is <t:union>`
`<t> is <t>`
`<wave> ==> <x:indexable>`

can be overrided: maths, <, >, =, @, [], ^, <-+, <'-, <-*, <-/

`/`: always returns float/double/normalized, not integer
`%`: remainder
`^`: power (binary), default view of (prefix)
`==`: compares bits, can't be overrided
`*`: pointer to, can't be overrided
`@`: dereference, can be overrided
`<-`: destruct + assign (`:`)
assign: unnamed: (move), named: `to`
`<->`: swap
`>&`: max, `<&`: min (uses `>` and `<`)
`as`: reinterpret
`>=`/`<=`/`/=` uses `<`/`>`/`=`

(all arrows can be reversed)

only one arrow per subexpression (?)

`[...]`/`(...)`/`{...}`: explicit function call
    *: if the next thing is not a parameter
    `<f/x:f> (...)`
    `<x> [...]`
    `<t> (<x:wave>)`: settle
    `<t> {...}`: constructor
        `{...}`: complete and ordered
        `'{...}`: incomplete or unordered
        `~{...}`: uninitialized
        arrays: can use indices instead of fields for named parameters
            if length is an enum then the values of the enum can be used
        `<t:scalar> {<x/cx>}`

`<f/x:f>[ ... ]` (function call)
    `(<a, ...>)`, `a`: `[<p[1]/p[2]>: ][ ref ]<x>` (requires ref if reference)
    `<a ...>`, `a`: `[<p[1]> ]<x>` (no ref even if parameter is reference)
    can have both named and unnamed parameters in the same call
    named parameters can be in any order (and have to come after unnamed ones)
array indexer optional (?)
    `a [i]`: `a i`
    `a i + b (i + 1)`

```
## values
```
`$<f>`: function
`~<t>`: destructor
`<global variable>`
`<local variable>`
`<platform>`
`<charset>`
`<type>`
`current platform`
`current charset`
`external (...)[ -> <t> ] <(cx:s)> in <cx:s> [ on <plf> ]`
`external <t> <(cx:s)> in <cx:s> [ on <plf> ]`
[<x:integer>] // bool wave / bitwave
[<x:view>/<x:range>]
(<x>) // subexpression
`<x/t/plf> <binary f/op> &{<x/(t:partial)/plf> => <x> , ...}` (?)
`[ G]"<...>"`: `ref textblock`/`grapheme`
    `\n`, `\t`, `\0`, `\\`
    string iterpolation: `\(<x>)`, uses stack
    `G` requires the charset is known in the current context
        either using `using` or `if&`
`<charset> to <charset>` can be casted to u8/u16/u32 array view (?)
`true`/`false`
`null`: default: `unknown*`

`[-]<digits...>[.<digits...>][K/M/G/T]/[e/E[-/+]<digits...>]`: cx
`0x<digits...>`: cx

references
    starts searching in current node and goes up the hierarchy
    conjugation
        function and variable names
        `is/has/was/will/can / s/ies/(ss/sh/ch/o/x es)`
            `isn't`/`is not`, ..., `doesn't -s/(-ies+y)/-es`/`does not ...`, ...
        also `items contains ...`: `items contain`/`items do not contain`
        functions that end with `of` are used like fields
        fields that start with `is/...` are used like functions
```
## rules/behaviour
* multiple words: picks the identifier with the most matching words
* operators allowed on type parameters (when they're not restricted)
    * default construct, swap/move, destruct, copy/convert (using `to`), 
        `*x`, `type of x`, `x as <t>`
* `t* + t*`: invalid
* `t* + int`: does *not* automatically multiply by the size of t
* namespaces in names are optional, only used for clarification
* untyped values in parameters and layouts are `number` by default
* global initializers can't reference other globals (variables or functions)
        that end up referencing back to itself
* can only get pointer to local function if it doesn't reference any locals
        outside of itself
* functions parameters are shallow immutable
    * can't get pointer to shallow immutable values
* arguments: shallow immutable, passed without calling copy constructors
* tags are references, identifiers using `<n>:` are values
        (uses `to`/move when declaring)
* struct returned by `return` resides at pointer passed by caller
    * return value is either named or temporary
* function call parameter mapping: 1d array views are expanded (?)
* when overriding a field the type has to have the same size
    * so both layouts have to be `ordered` (?)
* values can be automatically casted using `to`/`from`/`copy of` (?)
* flat scopes (`*::`) have to be at the top (excluding comments) (?)
* `<n>: <t> <- <x>` is *not* valid
* can't create template instance when the argument contains a restricted layout
    * platform specific layouts are not restricted when using a certain platform
        * `using` or `if&`
* imperative functions at the start of a line are evaluated last (?)
    * (functions that don't return anything (?))
    * `print "aaa" + "aaa"`
* argument to prefix declarative with single parameter can't be followed
        by a binary operator if it's at the start of a line (?)
* 0 size arrays aren't included in initializers (?)
    * but can still be accessed in templates (?)
* enum values are automatically used when assigning to an enum
    * `some color <- red`: same as `some color <- color red`
* combined characters are drawn as separate code points in charset
        and whole characters in literals (?)
## built in
```
t[^: ? d] :: layout
    data: t*
    volume: usize[d]
    step: usize[d - 1]

exit :: ? ()
assert :: ? (value: bool, message: textblock) // (?)
```
## textblock encoding (variable length)
| 1        | 2        | 3        | 4        | code points           |
|----------|----------|----------|----------|-----------------------|
| 0xxxxxxx |          |          |          | 128 (2^7)             |
| 110xxxxx | 10xxxxxx |          |          | 1920 (2^11 - 2^7)     |
| 1110xxxx | 10xxxxxx | 10xxxxxx |          | 63488 (2^16 - 2^11)	|
| 11110xxx | 10xxxxxx | 10xxxxxx | 10xxxxxx | 2031616 (2^21 - 2^16) |
## unsorted
```
tags
    `<x> # <n>`
    same as `<n>: ref <x>` but can be inside expressions

`;`: multiple expressions on the same line (?)

charset:
    `\\`, `\t`, `\ `, `\n`, `\:`, `\{<n>}`
    `-- <offset/unknown>[: <...> ]`
    starts at 1
    if less than 256 characters then it's fixed u8

`scope` (?)

calling without parentheses automatically uses `ref`/`^` (but not `*`)
    and `move`
        and `copy of` if move and not temporary (?)
function and () not followed by a parameter means function call
    (the parentheses belong to the call, they're not a subexpression)

passing a named value to `move` without parentheses automatically copies it
    `list <-+ <x:temporary>`: move
    `list <-+ <x:named>`: `copy of` + move
    `list <-+ (move <x:named>)`: move
    (?)

iterable: has start and end properties that return iterators
iterator: has `@`, `=`, `++`


function :: (view: (? t: u8 | u16 | u32)[^])
    if no restrictions or one of the restrictions is a trait then \
    you need a default case when checking type in `if&`
    functions that can be used with all types can be used outside `if&`
        (if they return the same type)

`to` and `copy of` are automatic
    `to` can't return same type
    `copy of` has to return same type
    (?)

same line `if` expression (?)
    `if <x> <x>` (second expression can't start with +/-/*/^)
```
## ?
```
`~layout`
    required to be able to declare destructor for type (?)

`<platform> <t>` (?)
    `file as windows file`

`case <x> => <n> & ...: (<t>, ...)`

```


# examples
## `move`
```
<-+ :: (list: (? t: solid) list) (item: move t)
    ...
    ref location: move item
```
## flags
```
options :: enum
    first: 1
    second: 2
    third: 4

function (options ([first] or [second] or [third]))
```
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
sin :: (a) -> number
print :: (text: string)

sin 4 // valid
sin 4 + 4 // invalid
print 4 + 4 // valid (same as `print (4 + 4)`)
sin (4 + 4) // valid

sin 4 -> other

```
## platform specific
```
file :: ? .layout

open file :: ? (name: string) -> file handle


file :: !windows .layout
    ...

open file :: !windows (name: string) -> file handle
    ...
```
## unions/layouts
```
t &struct :: layout
    t: :: enum
        first, second, third // first is 1, unknown is 0
    common value: u32
    case first
        first specific: 1
    case second => a & b: (u16, u32) // (?)

structure: first struct
structure's first specific <- 2

---
structure: unknown struct {common value: 10}

other: structure to first struct
other's first specific <- 2

```
## serialize, views, type selection if, partial types
```
serialize :: (value: ref (? t: solid), output: ref byte list)
    for field in t
        if& field is
            case (? t2) list
                ...
            case (? t2)[: ? d]
                for each element in field
                    serialize (ref element, output)
            case (? t2)[^: ? d] // don't include
            case (? t2)* // don't include
            default
                output <-+ byte[^] {
                        *value + field's offset, size of field's type}
```
## tags, error handling
```
if get info (0, '{} # output) succeeds
    ...
    print output's text
else
    assert false
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
`parameter: (? t)[: ? d][^: ? d2]`
`text: ref "aaa"`
`"aaa" # text`
```
# scrapped
## index types
```
node :: layout
    ...
    children: u8[2] // range (?)

nodes: node list ## [{children: [& => nodes]}]
list: node& list ## [& => nodes]
current: node& => nodes
```
## misc
```
`function (!object, ...)`: `object's function (object, ...)`
```

