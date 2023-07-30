![Axolotl](axolotl.png)

# [axol](#)

* Minimalist programming language simple to read, write, extend.
* Named after the axolotl animal for its ability to regrow missing body parts and for being cute.
* Version: 0.2.2
* Docs:
{:toc}

## [Simple literals](#simple-literals)

```axol
null
true
false
-3.14
```

## [Comment](#comment)

```axol
# Text from # to the end of line is a comment
# unless # is inside of a string literal
```

## [String literals](#string-literals)

```axol
"Double-quoted multiline string # here
expands \"escape\tsequences\" including \\.

It also expands var {names} and {other.code()}
unless curly braces are escaped like \{these\}."

'Single-quoted multiline raw string
expands \no {special sequences
to avoid double escaping of \regex and {other code}.'
```

## [Object literal](#object-literal)

```axol
(null true false -3.14 "string" name="value" more=true)
```

## [Function literal](#function-literal)

```axol
# No-op nameless function:
{}

# Named function which prints args it got and returns result:
set func={
  print "Object with all args:" a

  print "Number of positional args:" a:len
  print "First positional args:" a.0 a.1 a.2
  print "Last positional arg:" a.-1
  print "Which is equal to:" a:get(a:len:minus(1))

  print "Value of named arg:" a.name
  print "Result of its call:" a.name()
  print "Default value:" a.answer:or(42)
  # So a part of an object can be accessed with `.`
  # Another function can be applied with `:`

  # Set local variables from args:
  set foo=a.0 bar=a.1 name=a.name answer=a.answer:or(42)
  print foo bar name answer

  set result="Calculated from {foo} and other args"
  return result
  # If no `return` was called, then result of the last line is returned.
}
```

## [Function call](#function-call)

Equal effects are grouped together.

```axol
func(a0 a1 name=value)
func a0 a1 name=value
a0:func(a1 name=value)
a0:func a1 name=value

set result=a0:func()
set result=a0:func

set result=func(a0 a1 name=value)
set result=a0:func(a1 name=value)
# `()` are required to avoid confusion with the case below.

set result=a0:func name=value
# This sets `result=func(a0)`
# and also sets `name=value`.

set action={func a0 a1 name=value}
set result=action()

{print a.0} "hello world"

{print a} a0 a1 name=value

set result={a}(a0 a1 name=value)
set result=(a0 a1 name=value)
```

## [Builtin functions](#builtin-functions)

### [set](#set)

```axol
set name="value" more=true
# Sets in current local scope.

set foo "bar" "baz" name="value"
foo:set "bar" "baz" name="value"
# Sets in `foo` object.
```

### [get](#get)

```axol
print name more
print get("name") get("more")
# Gets from current scope, else from outer scopes.

print foo.0 foo.1 foo.name
print foo:get(0) foo:get(-1) foo:get("name")
# Gets from `foo` object.
```

### [len](#len)

Length of positional side of an object.

```axol
set foo=("bar" "baz" name="value")
print foo:len
# Prints 2, not 3.
```

### [each](#each)

```axol
set foo=("bar" "baz" name="value")
foo:each {print a.name a.value}
# Prints:
# 0 bar
# 1 baz
# name value

("foo" "bar" "baz"):each {print a.value}
# Prints:
# foo
# bar
# baz
```

### [map](#map)

```axol
set foo=("bar" "baz" name="value")
print foo:map({"=":join((a.name a.value))})
# Prints: 0=bar 1=baz name=value

# You can also call `join` in `a0:func a1` form:
print foo:map({"=":join (a.name a.value)})

# Or in `func a0 a1` form:
print foo:map({join "=" (a.name a.value)})
```

### [join](#join)

```axol
set foo=("bar" "baz" "qux")
print ",":join(foo)
# Prints: bar,baz,qux

print ",":join(("bar" "baz" "qux"))
# Prints: bar,baz,qux
```

### [add](#add)

```axol
set foo=("c" "d" "e")
foo:add "f" "g"
print foo
# Prints: c d e f g
```

### [ins](#ins)

```axol
set foo=("c" "d" "e")
foo:ins "a" "b" at=0
print foo
# Prints: a b c d e
```

### [del](#del)

```axol
set foo=("c" "d" "e" name="value" more=true)

print foo:del(1 "name")
# Prints: d value

print foo
# Prints: c e more=true
```

### [if](#if)

```axol
if condition then=action1 else=action2

set state=(foo=null)
if name:eq("value") then={
  state:set foo=bar
  # Just `set foo=bar` would set `foo` local to `then` scope.
} else={
  state:set foo=baz
}

set foo=if(name:eq("value") then={bar} else={baz})
# Lazy eval of `then`/`else` values.

set foo=iif(name:eq("value") then=bar else=baz)
# Instant eval of `then`/`else` values.
```

### [while](#while)

```axol
while getCondition do=action
# Note `if condition` vs `while getCondition`,
# as `while` needs to get condition multiple times.

set state=(found=false)
while {not state.found} do={
  if find() then={
    state:set found=true
  }
}
```

### [try](#try)

```axol
try {cry} catch={}
# All caught!

try {
  if not(found) then={
    cry type="bug" why="not found"
  }
  print "this is run if found only"
} catch={
  set err=a
  if err.a.type:eq("bug") then={
    print "bug \"{err.a.why}\"
      at line {err.line}
      of file {err.file}
      will not raise higher"
  } else=err.cry
  # Else cry again about the same error.
  # Default handler will print original error details to stderr and exit with code 1.
} finally={
  print "this will always run"
}
```

### [Operators](#operators)

```axol
2:plus(2):eq(4):eq(true)
not(true):eq(false)
null:or(42):eq(42)
42:and("more"):eq("more")
```

## [Types](#types)

```axol
set cat={(
  legs=4
  say={print "meow"}
)}

set alice=cat()
print "alice has {alice.legs} legs and says:"
alice.say

set axolotl={
  set this=cat(legs=a.legs:or(4))
  this:set type=axolotl

  this:set mutate={
    this:set(
      legs=a.legs:or(42)
      wings=2
    )
  }

  this:set fly={
    if not(this.wings) then={
      cry why="no wings yet"
    }
    print "flying..."
  }

  this
}
set axolotl.parent=cat

set bob=axolotl()
bob.mutate
bob.fly
```

## [Import](#import)

```axol
import lib="../../lib.axol"
lib.func a0 a1 name=value

import lib="https://example.com/some/lib.axol"
set result=a0:lib.func(a1 name=value)

import py="python3"
set items=("b" "a" "c" "d")
print items:py.sort
print items:py.sort(key={lib.func name=a.0})
# https://docs.python.org/3/library/stdtypes.html#list.sort

set os=py.import("os")
os.listdir():each {print a.value}
# That's how small axol regrows missing body parts!
```

## [Roadmap](#roadmap)

* Add axol to [github/linguist](https://github.com/github/linguist/blob/master/lib/linguist/languages.yml) and [Rouge](https://github.com/rouge-ruby/rouge) highlighters to simplify the reviews.
* Reviews and fixes of this draft specification.
* Implement it.

## [That's all!](#thats-all)

Thank you for reading.

Please [post an issue or idea](https://github.com/whyolet/axol/issues) or contribute otherwise:

<img src="GitHub-Mark-32px.png" alt="GitHub icon" align="center" /> [https://github.com/whyolet/axol](https://github.com/whyolet/axol)
