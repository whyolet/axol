![Axolotl](axolotl.png)

# [axol](#)

* Minimalist programming language simple to read, write, extend.
* Influenced by: [Python](https://www.python.org/), [Bash](https://www.gnu.org/software/bash/), [XL](https://xlr.sourceforge.io/).
* Axol is named after the axolotl animal for its ability to regrow missing body parts and for being cute.
* Version: 0.3.0
* Docs:
{:toc}

## [comment](#comment)

```axol
# Text from `#` to the end of line is a comment,
# unless `#` is inside of a string.
```

## [string](#string)

```axol
"Double-quoted multiline string # here
expands \"escape\tsequences\"
including \\ and no-newline: \

It also expands var {names} and {other.code()}
unless curly braces are escaped like \{these\}."

'Single-quoted multiline raw string
expands \no {special sequences
to avoid escaping of "double quotes" and {code}
and double escaping of \regex.'
```

* String having no `"\{}` chars should be double-quoted to avoid conflicts of personal preferences.

## [value](#value)

```axol
null
false
true
-3.14
"string"
```
* These are examples of simple values: `null`, bools, numbers, strings.
* More values like boxes and actions are described below.

## [placeholders](#placeholders)

* Examples below use placeholders: `aaa`, `bbb`, `ccc`, `ddd`, ..., `yyy`, `zzz`.
* They are simpler to invent and recognize than few well-known [metasyntactic variables](https://en.wikipedia.org/wiki/Metasyntactic_variable) `foo`, `bar`, `baz`, `qux`, ...what's next?
* They avoid confusion with:
  * Words that should be used as is, like `true`, `lib`, `file`.
  * Short words like `a` being an article.
  * Each other, unlike `name1`, `name2`, `name3`, ..., `name25`, `name26`.

## [box-concept](#box-concept)

* Box is a namespace - a space with names, not values.
* Each name points to a value outside the box, e.g:
```
________
|      |
| 0 -----> null
| 1 -----> false
| 2 -----> true         /--\
| 3 -----> -3.14        |  |
| aaa ---> "bbb" _______v_ |
| ccc -------^   |       | |
| ddd ---------->| self ---/
|______|         |_______|
```

## [auto-created](#auto-created)

* Given axol app:
```axol
name="Alice"
action: name=@nick
action(nick="Mad")
```
* the next boxes are created:
```
              /--\
              |  |
  ____________v_ |
  |            | |
  |  $local -----/
  |  $import ----> : ...
  |  name -------> "Alice"
  |  action -----> : name=@nick
  |____________|
   ^
   |          /--\
/--/          |  |
| ____________v_ |
| |            | |
| |  $local -----/
\--- $outer    |
\--- $caller   |           _________
  |  $args --------------->|       |
  |  @nick ------> "Mad" <--- nick |
  |  name ----------^      |_______|
  |____________|
```
* Each time axol app is executed,
  * or a lib is imported,
  * or an action is called,
  * a new local scope is auto-created.
* Local scope is a box with:
  * Custom local names created by the app/lib/action.
  * Auto-created local names.
* Name auto-created in any scope is:
  * `$local` - points to this local scope box itself.
* Name auto-created in scopes of app and lib is:
  * [$import](#import) - action to import other libs.
* Names auto-created in action scope are:
  * `$outer` - points to the scope box where this action was created.
  * `$caller` - points to the scope box of the caller of this action.
  * `$args` - points to a box with arguments passed by the caller of this action.
  * `@0` - shortcut for `$args.0` - the first positional argument.
  * `@1` - shortcut for `$args.1`, and so on.
  * `@nnn` - shortcut for `$args.aaa` - named argument `aaa="bbb"`
* See also: [env](#env), [cli_args](#cli-args), [pargs](#pargs), [nargs](#nargs).
* All names starting with `$` or `@` are reserved for axol to avoid collision with custom names created by user.

## [name=value](#name-value)

```axol
aaa="bbb"
```
* Create or update name `aaa` in local scope.
* Make it point to value `"bbb"`.
* See also: [up](#up) finds and updates name in local or outer scopes.

## [name](#name)

```axol
aaa
```
* Find name `aaa` in local scope.
* If found, then return a value it points to.
* Else try the same in `$outer` scope, else in its `$outer`, and so on.
  * This step is not applied for [auto-created](#auto-created) names starting with `@`, as they are shortcuts to names in `$args` box which is always found in the local scope.
* Else cry "Name `aaa` is not found."

## [action](#action)

```axol
: bbb
```
* Get a new action.
* When this action is [called](#call-concept), it will execute the code `bbb` inside of this action.
* This trivial code `bbb` will return value of name `bbb` [visible](#name) from inside of this action, even if the caller of this action does not [see](#name) this `bbb`.
* Action is known in other languages as closure, lambda, function, procedure, method, etc.

## [name: action](#name-action)

```axol
aaa: bbb
```
* Create or update name `aaa` in local scope.
* Make it point to a new [action](#action).
* This is a [name=value](#name-value) composed with [action](#action), but:
  * Cute `aaa: bbb` syntax should always be used instead of `aaa=: bbb` composed from `aaa=` and `: bbb`

## [multiline-action](#multiline-action)

```axol
:
  # Multiple
  # lines here.
  bbb
```
* Get a new [action](#action) that will execute multiple indented lines and return value of `bbb`.
* One indentation level should contain exactly two spaces, no tabs.

## [name: multiline-action](#name-multiline-action)

```axol
aaa:
  # Multiple
  # lines here.
  bbb
```
* Create or update name `aaa` in local scope.
* Make it point to a new [multiline-action](#multiline-action).
* This is a [name=value](#name-value) composed with [multiline-action](#multiline-action), but:
  * Cute `aaa: multiline_bbb` syntax should always be used instead of `aaa=: multiline_bbb` composed from `aaa=` and `: multiline_bbb`

## [call-concept](#call-concept)

Given `aaa: @0`
```axol
result=aaa("bbb")
```
means:
* Get the [action](#action) `: @0` pointed by name `aaa`.
* Create its [local scope box with few names inside](#auto-created), including:
  * `$args` pointing to a new box with the only name `0` inside, pointing to the value `"bbb"` passed as the only positional argument.
  * `@0` being a shortcut for `$args.0` hence pointing to the same value `"bbb"`.
* Run the code of this trivial action `: @0`
  * Get the value `"bbb"` of the [name](#name) `@0`
  * Return this value to the caller of this action.
* As in [name=value](#name-value), create or update name `result` in local scope of the caller, make this name point to returned value `"bbb"`.
* See also: [inline-call](#inline-call), [newline-call](#newline-call), [multiline-call](#multiline-call), [pipeline-call](#pipeline-call), box [$call](#call).

## [inline-call](#inline-call)

```axol
aaa(bbb "ccc" ddd="eee" fff: ggg)
```
* Get result of action `aaa` [called](#call-concept) with arguments:
  * [name](#name) - `aaa(bbb)`
    * Value of name `bbb` is passed as a positional argument.
  * [value](#value) - `aaa("ccc")`
    * Value `"ccc"` is passed as a positional argument.
  * [action](#action) - `aaa(: hhh)`
    * Action `: hhh` is passed as a positional argument.
    * Supported only when it is the only argument - as otherwise it would be easy to confuse `aaa(bbb : hhh)` with `aaa(bbb: hhh)`
  * [multiline-action](#multiline-action)
    * Not supported, use [multiline-call](#multiline-call).
  * [name=value](#name-value) - `aaa(ddd="eee")`
    * Passed as a named argument `ddd` with value being `"eee"`.
  * [name: action](#name-action) - `aaa(fff: ggg)`
    * Passed as a named argument `fff` with value being an action `: ggg`
  * [name: multiline-action](#name-multiline-action)
    * Not supported, use [multiline-call](#multiline-call).
  * Without arguments - `aaa()`
* If an [inline-call](#inline-call) starts on its own line and has arguments, then it should be converted to a [newline-call](#newline-call) by deleting the parentheses `()` for readability.

## [newline-call](#newline-call)

```axol
aaa bbb "ccc" ddd="eee" fff: ggg
```
* The same as [inline-call](#inline-call), but:
  * It should be on its own line in app/lib/action, as otherwise `aaa` and `bbb` become arguments as in `print aaa bbb "ccc"`
  * Passing [action](#action) without [name: action](#name-action) is not supported, as `aaa : hhh` can be easily confused with `aaa: hhh`
  * "Without arguments" case is not supported, as it would be the same as just getting a value of [name](#name) `aaa`.
  * It can return result only when it is the only or the last line of an action:
```axol
get_result: aaa bbb "ccc" ddd="eee"

get_result:
  # Multiple
  # lines here.
  aaa bbb "ccc" ddd="eee"

result=get_result()
```

## [multiline-call](#multiline-call)

```axol
aaa bbb "ccc" ddd="eee" fff: ggg
  : hhh
  :
    iii
    jjj
  kkk:
    lll
    mmm
  nnn ooo "ppp" qqq="rrr" sss: ttt
    : uuu
```

* The same as [newline-call](#newline-call), but:
  * All "not supported" and "supported only when" restrictions of [inline-call](#inline-call) and [newline-call](#newline-call) (except "Without arguments") are removed in [multiline-call](#multiline-call) for indented arguments, each on its own line(s).
  * One indentation level should contain exactly two spaces, no tabs.
  * A [multiline-call](#multiline-call) with all arguments indented can return result via [name=value](#name-value) e.g:
```axol
result=aaa
  bbb
  "ccc"
  ddd="eee"
  fff: ggg
  : hhh
```

## [pipeline-call](#pipeline-call)

```axol
a0|aaa(a1 a2)

a0|aaa a1 a2

a0|aaa
  a1
  a2
```
* The [inline-call](#inline-call), [newline-call](#newline-call), and [multiline-call](#multiline-call) above have their first positional argument `a0` passed through the pipe `|`
* They all are equal to `aaa(a0 a1 a2)`
* It makes infix actions like `length|minus(1)` more readable than `minus(length 1)`.
* It avoids need in operator precedence that can be confused with different operator precedence from another programming language:
  * What `aaa / bbb % ccc` means?
    * `(aaa / bbb) % ccc` ?
    * `aaa / (bbb % ccc)` ?
    * Are you sure?
  * With axol you are sure:
    * `aaa|div(bbb|mod(ccc))` means `aaa / (bbb % ccc)`
    * `aaa|div(bbb)|mod(ccc)` means `(aaa / bbb) % ccc`
* It enables concise left-to-right pipelines like `items|map(to: "{@name}={@value}")|sorted|join("\n")`
* Note that parentheses `()` are omitted in `|sorted` because it is definitely a [pipeline-call](#pipeline-call), which cannot be confused with just getting a value of [name](#name) `sorted`.

## [$import](#import)

```axol
aaa=$import("bbb ccc=ddd" file="../eee.axol")
```
* If `file` is not passed, then `file` is `.axol/main.axol` - main features of axol standard library.
* If `file` starts with `.axol/`
  * Then find it in:
    * The same directory as the importer.
    * Else in importer's parent directory.
    * Else in one more parent directory, and so on.
    * Else in OS-specific user home directory like `~`
    * Else in OS-specific shared directory like `/usr/local/lib`
    * Else cry "File `.axol/eee.axol` is not found."
  * Else `file` path should start with either `./` or `../` to clearly be relative to importer's directory.

* If the absolute path to the `file` is not in the import cache box yet, then:
  * Add this path to this cache as a name, pointing to a new box.
  * Run the `file` passing this box as its local scope.

* For each positional argument like `"bbb ccc=ddd"` passed:
  * Create or update names `bbb` and `ccc` in local scope of the caller.
  * Make them point to the values which names `bbb` and `ddd` point to in the cached scope box of the `file`:
```axol
# lib.axol
bbb="fff"
ddd="ggg"

# app.axol
$import "bbb ccc=ddd" file="./lib.axol"
print bbb ccc # fff ggg
```

* Return the cached scope box of the `file` to the caller, e.g. `axol=$import()` will create a name `axol` pointing to a box with main features of axol standard library.

* `$import` is one of very few [auto-created](#auto-created) names. All custom names should be either imported from axol standard library or other libs or created by an app/lib/action:
```axol
$import "each print"

py=$import
  file=".axol/python.axol"
  "all any min max sum zip" # SEO
  "sorted" # used in this example

os=py.__import__("os")

os.listdir()|sorted|each do:
  print @value
```
* Lib `.axol/python.axol` contains axol bindings to all names available in python language, e.g. its [built-in functions](https://docs.python.org/3/library/functions.html).
* [`python.__import__`](https://docs.python.org/3/library/functions.html#import__) can import any [installed python packages](https://pypi.org/) and packages from its [standard library](https://docs.python.org/3/library/index.html).
* That's how small axol regrows missing body parts!

## [box](#box)

```axol
$import "box print"

aaa=box(bbb "ccc" ddd="eee" fff: ggg)

print aaa.1 # ccc
print aaa.ddd # eee
aaa.fff()

aaa.hhh="iii"
print aaa.hhh # iii
```
* Get a new box with passed args.
* They can be get and set using either dot `.` or actions - see [get](#get), [set](#set), etc.
* See [box-concept](#box-concept) and [inline-call](#inline-call).
* Implementation:
```axol
box: $args
```

## [print](#print)

```axol
$import "print"

print()
# (newline)

print "hi" # hi
# (newline)

print "yes/no: " end="" # yes/no: (no newline)

print "aaa bbb" "ccc" # aaa bbb ccc

print "aaa bbb" "ccc" sep="" # aaa bbbccc

print null false true "true" -3.14 # null false true true -3.14

print box(null false true "true" -3.14 aaa="bbb" ccc: ddd)
# null
# false
# true
# "true"
# -3.14
# aaa="bbb"
# ccc: ddd

print "failed" to=2 # failed
# (to stderr)

result=print("aaa" "bbb" to=null)
print result # aaa bbb

print aaa bbb sep=" " end="\n" to=1
```
* Join positional args to the resulting string using `sep` separator, default is one space.
  * Printing a `box` prints its items, each on its own line(s).
* Append the `end`, default is a newline.
* If `to` is not `null`: print the resulting string to [file descriptor](https://en.wikipedia.org/wiki/File_descriptor) `to`, default is 1 aka `stdout`.
* Return the resulting string.

## [input](#input)

```axol
$import "input"

aaa=input(end="\n")
```
* Input a string from stdin.
* End the input on EOF or when the `end` is consumed, newline by default.
* Return the result excluding the `end`.

## [env](#env)

* Given command line:
```
AAA=bbb ccc=ddd path/to/app.axol
```
* `env` points to a box with environment variables:
```axol
env=box
  AAA="bbb"
  ccc="ddd"
  # More inherited env vars.
```
* Usage:
```axol
$import "env print"

print env.AAA # bbb
```

## [cli_args](#cli-args)

* Given command line:
```
path/to/app.axol aaa "bbb ccc" -ddd --eee=fff
```
* `cli_args` points to a box with command line arguments:
```axol
cli_args=box
  "path/to/app.axol"
  "aaa"
  "bbb ccc"
  "-ddd"
  "--eee=fff"
  eee="fff"
```
* Usage:
```axol
$import "cli_args print"

print cli_args.0 # path/to/app.axol
print cli_args.1 # aaa
print cli_args.eee # fff
```

## [pargs](#pargs)
## [nargs](#nargs)

```axol
$import "nargs pargs print"

aaa:
  pargs "bbb ccc" ddd="eee" fff="ggg"
  nargs "hhh iii" jjj="kkk" lll="mmm"

  print bbb ccc ddd fff
  print hhh iii jjj lll

aaa
  "nnn" "ooo" "ppp"
  hhh="qqq" iii="rrr" lll="sss"
# nnn ooo ppp ggg
# qqq rrr kkk sss
```
* `pargs`:
  * Set local name `bbb` to point to positional arg `@0`.
  * `ccc` - to `@1`.
  * `ddd` - to `@2`, etc.
* `nargs`:
  * Set local name `hhh` to point to named arg `@hhh`.
  * `iii` - to `@iii`.
  * `jjj` - to `@jjj`, etc.
* Names passed in the positional args are required, e.g. `bbb` and `hhh`.
* Names passed as named args are optional with given default values, e.g. `fff="ggg"`.

## [bool](#bool)

```axol
$import "bool box"

bool(null)
bool(false)
bool(0)
bool("")
bool(box())
```
* All `bool`-s above return `false`.
* `bool` of anything else is `true`.

* Each bool comparison below is `true`:
```axol
$import
  "eq ne lt gt lte gte"
  "not and or xor"

"aaa"|eq("aaa") # EQual
"aaa"|ne("bbb") # Not Equal
"aaa"|lt("bbb") # Less Than
"bbb"|gt("aaa") # Greater Than
false|lte(true) # Less Than or Equal
true|gte(false) # Greater Than or Equal
not(false)|eq(true)
true|and(false)|eq(false)
true|or(false)|eq(true)
true|xor(true)|eq(false)
```

## [math](#math)

* Each line below is `true`:
```axol
$import
  "plus minus mul div"
  "pow mod abs eq"

2|plus(3)|eq(5) # 2 + 3 == 5
5|minus(3)|eq(2) # 5 - 3 == 2
2|mul(3)|eq(6) # 2 * 3 == 6
6|div(3)|eq(2) # 6 / 3 == 2

"2"|plus("3")|eq("23")
"2"|mul(3)|eq("222")

2|pow(8)|eq(256) # POWer, ^, **
-2|abs|eq(2) # ABSolute value, |x|

256|pow(1|div(8))|minus(2)|abs|lt(0.001)
# 256^(1/8) is a float close to 2

5|mod(2)|eq(1) # MODulo, %
```

## [set](#set)

```axol
$import "box print set"
aaa=box()

aaa|set "bbb" "ccc"
print aaa # bbb="ccc"
```
* The same as setting `aaa.bbb="ccc"` [using dot](#box), but `name` is passed as a positional argument `@1` to support dynamic naming, e.g. in a [loop](#loop).

## [get](#get)

```axol
$import "box get print"
aaa=box(bbb="ccc")

print aaa|get("bbb") # ccc

print aaa|get("ddd" default=null) # null
```
* The same as getting `aaa.bbb` [using dot](#box), but:
  * `name` is passed as a positional argument `@1` to support dynamic naming, e.g. in a [loop](#loop).
  * If optional `default=value` is passed, then it is returned instead of crying "Name `ddd` is not found."

* If [bool](#bool) comparisons `lt gt lte gte` are used as named arguments instead of the positional argument `@1`:
  * Then return a new box with positional values (from original box `@0`) having indices satisfying all comparisons:
```axol
$import "box get print"
aaa=box("bbb" "ccc" "ddd" "eee" fff="ggg")

print aaa|get(lt=2)
# "bbb"
# "ccc"

print aaa|get(gte=1 lte=2)
# "ccc"
# "ddd"

print aaa|get(gt=0)
# "ccc"
# "ddd"
# "eee"
```

* Both dot `.` and [get](#get) can be used to get substrings too:
```axol
import "get print"
aaa="string"

print aaa.0 # s
print aaa.1 # t
print aaa|get(2) # r
print aaa|get(gte=2 lte=3) # ri
print aaa|get(gt=3) # ng
```

## [up](#up)

```axol
$import "print up"
aaa="bbb"

ccc:
  aaa="ddd"
  print aaa # ddd
ccc()
print aaa # bbb

ccc:
  up aaa="ddd"
  print name # ddd
ccc()
print aaa # ddd

up eee="fff"
# Error: Name `eee` is not found.

ggg=box(hhh="iii" jjj: kkk)
ggg|up hhh="lll" jjj: mmm
print ggg.hhh # lll

ggg|up aaa="nnn"
# Error: Name `aaa` is not found.
```
* While [name=value](#name-value), [name: action](#name-action), and [name: multiline-action](#name-multiline-action) create or update names in local scope only,
* `up` updates names (never creates) in the next way:
  * If a positional argument `@0` is passed:
    * Then set internal name `where` to this box: `@0`.
    * Else set it to the [local scope of the caller](#auto-created): `$caller`
  * For each [name=value](#name-value) argument passed:
    * Find the name in the `where` box.
    * Else in `where.$outer`, if any.
    * Else in `where.$outer.$outer` and so on.
    * Else cry with the same error as the failed [name](#name) lookup.
    * If found, then update the name where it was found, to point to a new value or action passed.

## [add](#add)

```axol
$import "add box print"
aaa=box("ccc" yyy="zzz")

aaa|add "ddd" "eee"
aaa|add "aaa" "bbb" at=0
print aaa
# "aaa"
# "bbb"
# "ccc"
# "ddd"
# "eee"
# yyy="zzz"

aaa=box("ccc" yyy="zzz")
aaa|add flat=true
  "ddd"
  box("eee" "fff")
  box(ggg="hhh")
print aaa
# "ccc"
# "ddd"
# "eee"
# "fff"
# ggg="hhh"
# yyy="zzz"

aaa|add bbb ccc at=-1 flat=false
```
* Add values passed as positional arguments to the box passed as `@0`:
  * If `at` index is passed:
    * Then insert values at given index.
    * Else append values after existing positional values in the box.
  * If `flat` is passed and is `true`:
    * Then unpack all boxes passed as positional args, adding their contents to the target box.
    * Else add such boxes as is, without unpacking.

## [del](#del)

```axol
$import "box del print"
aaa=box("bbb" ccc="ddd")

print aaa|del(0) # bbb
print aaa
# ccc="ddd"

print aaa|del("ccc") # ddd
print aaa
#

print aaa|del("ccc" default=null) # null
```
* The same as [get](#get), but also deletes the name passed as `@1` from the box passed as `@0`.
* If [bool](#bool) comparisons are used instead of `@1`, then [get](#get) and delete matching indices:
```axol
$import "box del print"
aaa=box("bbb" "ccc" "ddd" "eee" fff="ggg")

print aaa|del(gte=2)
# "ddd"
# "eee"

print aaa
# "bbb"
# "ccc"
# fff="ggg"
```

## [names](#names)

```axol
$import "box names print"
aaa=box("bbb" "ccc" ddd="eee")

print aaa|names
# 0
# 1
# "ddd"

print $local|names
# "$import"
# "$local"
# "aaa"
# "box"
# "names"
# "print"
```
* Get a new box with values being names from the box passed as `@0`

## [values](#values)

```axol
$import "box print values"
aaa=box("bbb" "ccc" ddd="eee")

print aaa|values
# "bbb"
# "ccc"
# "eee"
```
* Get a new box with values from the box passed as `@0`

## [join](#join)

```axol
$import "box join print"
aaa=box("bbb" "ccc" ddd="eee")

print aaa|join(",") # bbb,ccc,eee
```
* Get a string created by joining values from the box passed as `@0` using separator passed as `@1`.

## [split](#split)

```axol
$import "print split"

print "aaa,bbb,ccc"|split(",")
# "aaa"
# "bbb"
# "ccc"
```
* Get a new box with values split from the string passed as `@0` using separator passed as `@1`.

## [length](#length)

```axol
$import "box length print"

print "string"|length # 6

aaa=box("bbb" "ccc" ddd="eee")
print aaa|length # 2, not 3.
```
* If `@0` is a string: return its length in characters.
* Else if `@0` is a box: return the length of its positional side, ignoring `name="value"` pairs.
* Else cry "Value `{@0}` has no length."

## [find](#find)

```axol
$import "box find print"

print "aaabbbccc"|find("bbb") # 3
print "aaabbbccc"|find("ddd") # null

aaa=box("bbb" "ccc" ddd="eee")
print aaa|find("bbb") # 0
print aaa|find("ddd") # null
print aaa|find("eee") # ddd
```
* If the values passed are strings:
  * If `@1` is found in `@0`:
    * Then return the first matching index.
    * Else return `null`.
* Else if `@0` is a box:
  * If `@1` is found in the values of the box `@0`:
    * Then return the first matching name.
    * Else return `null`.
* Else cry "Presence of `{@1}` in `{@0}` cannot be checked."

## [in](#in)

```axol
$import "box in names print"

print "bbb"|in("aaabbbccc") # true
print "ddd"|in("aaabbbccc") # false

aaa=box("bbb" "ccc" ddd="eee")
print "bbb"|in(aaa) # true
print "ddd"|in(aaa) # false
print "ddd"|in(aaa|names) # true
print "eee"|in(aaa) # true
```
* The same as `@1|find(@0)|ne(null)`
* See [find](#find).

## [if](#if)

```axol
$import "if"

if aaa then: bbb
  else: ccc

if aaa
  then:
    bbb
    ddd
  else:
    ccc
    eee

result=if
  aaa
  then: bbb
  else: ccc
```
* If `bool` of condition `aaa` is `true` and `then` action is passed, then call it.
* If `bool` of condition `aaa` is `false` and `else` action is passed, then call it.
* Return result of the action called.

## [then](#then)

```axol
$import "print then"

print aaa|then(bbb else=ccc)
```
* If `bool` of condition `aaa` is `true`:
  * Then return `bbb`.
  * Else return `ccc`.
* Unlike [short-circuit evaluation](https://en.m.wikipedia.org/wiki/Short-circuit_evaluation) of actions in [if](#if), here both branches are evaluated instantly as simple values.

## [elif](#elif)

```axol
$import "elif"

elif
  aaa
  : bbb

  ccc
  : ddd

  eee
  : fff

  else: ggg
```
* The same as few nested [if](#if)-s:
```axol
$import "if"

if aaa
  then: bbb
  else:
    if ccc
      then: ddd
      else:
        if eee
          then: fff
          else: ggg
```

## [case](#case)

```axol
$import "case"

case aaa
  "bbb"
  : ccc

  "ddd"
  : eee

  "fff"
  : ggg

  else: hhh
```
* The same as this [elif](#elif):
```axol
$import "elif"

elif
  aaa|eq("bbb")
  : ccc

  aaa|eq("ddd")
  : eee

  aaa|eq("fff")
  : ggg

  else: hhh
```

## [loop](#loop)

```axol
$import "loop"

loop
  while: aaa
  do: bbb
  until: ccc

loop while: aaa
  do: bbb
```
* Set name `do_result` in the local scope of this `loop` call to point to `null`.
* Repeat the next steps:
  * If `while` action is passed, then:
    * Call it.
    * If `bool` of its result is `false`, then:
      * Break the loop.
  * `up do_result=do()`
    * Call the `do` action.
    * Update `do_result` name to point to the result of `do` action.
  * If `until` action is passed, then:
    * Call it.
    * If `bool` of its result is `true`, then:
      * Break the loop.
* Return the value of `do_result`.

## [each](#each)

```axol
$import "box each print"
aaa=box("bbb" "ccc" ddd="eee")

aaa|each do: print @name @value
# 0 bbb
# 1 ccc
# ddd eee
```
* For each `name=value` in the box passed as `@0`:
  * Call action `do(name=name value=value)`.
* A string is treated as a box with characters:
```axol
$import "each print"

"str"|each do: print @name @value
# 0 s
# 1 t
# 2 r
```

## [map](#map)

```axol
$import "box map print"
aaa=box("bbb" "ccc" ddd="eee")

print aaa|map(to: "{@name}={@value}")
# "0=bbb"
# "1=ccc"
# "ddd=eee"

print aaa|map(
  to: $value
  if: $name|ne(1)
)
# "bbb"
# "eee"

print aaa|map(flat=true to: $item)
# "bbb"
# "ccc"
# ddd="eee"

aaa|map
  to: bbb
  if: ccc
  flat=false
```
* Create `map_result=box()`
* For each `name=value` in the box passed as `@0`:
  * Create `item=box()`
  * `item|set name value`
  * If `if` action is passed, then:
    * Call `if_result=if(name=name value=value item=item)`
    * If `bool` of `if_result` is false, then continue from the next iteration of the loop.
  * Call `to_result=to(name=name value=value item=item)`
  * `map_result|add(to_result flat=flat)`
    * See [add](#add).
* Return `map_result`.
* A string is treated as a box with characters:
```axol
$import "map print"

print "str"|map(flat=true to: $item)
# 0="s'
# 1="t"
# 2="r"
```

## [$call](#call)

```axol
$import "box print"

aaa=box($call: "bbb")

print aaa
# $call: "bbb"

print aaa() # bbb
```
* Name `$call` is not [auto-created](#auto-created), but it has a reserved meaning.
* When a box is [called](#call-concept), its `$call` action is called instead.
* Else cry "Cannot call: neither an action, nor a box with `$call` action."

## [roadmap](#roadmap)

* Work on the [drafts](drafts.md) left.
* Add axol to the next highlighters to simplify the reviews:
  * [Rouge](https://github.com/rouge-ruby/rouge) for our GitHub Pages website.
  * [github-linguist](https://github.com/github-linguist/linguist) for GitHub main UI and for grammars used by popular code editors.
* Reviews and fixes of this draft specification.
* Implement it.

## [that's all!](#thats-all)

Thank you for reading.

Please [post an issue or idea](https://github.com/whyolet/axol/issues) or contribute otherwise:

<img src="GitHub-Mark-32px.png" alt="GitHub icon" align="center" /> [https://github.com/whyolet/axol](https://github.com/whyolet/axol)
