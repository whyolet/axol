# Axolang

Axolang is a minimalist programming language, named after the axolotl animal for its ability to regrow missing body parts and for being cute.

![Axolotl](axolotl.png)

Please don't confuse Axolang with:
* https://github.com/axolotl-lang/axolotl
* https://github.com/axodotdev

This early draft version 0.0.1 is made of few examples.

## Run Axolang file

```bash
axolang run hello.axo
```

## Produce values from literals

```axolang
"foo"
-3.14
true
null
{}
// The last one is a function.
```

## All values have few shared functions

```axolang
null.is(null)
null.or(42).is(42)
```

## Set name=value

```axolang
set foo="bar"
set baz=foo
```

## Get value:

```axolang
print foo
print "foo = {foo}"
```

## Set function

```axolang
set func={
  print "List of all positional parameters = {p}"
  print "First = {p.0}"
  print "Middle = {p.get(p.len.div(2).round)}"
  print "Last = {p.-1}"

  print "Map of all named parameters = {n}"
  print "Named parameter = {n.name}"
  print "Default value = {n.size.or(42)}"

  "The last value in a function is its result"
}
```

## Call function

```axolang
func

func p0 p1 name=value

func p0 p1
  name=value

func
  p0
  p1
  name=value
```

## Get reference to a function

```axolang
print func
set foo=func
if condition then=func
```

## Call function and get its result

```axolang
print func()
set foo=func(p0 p1 name=value)

set foo=func(
  p0
  p1
  name=value
)

set action={func p0 p1 name=value}
set foo=action()
```

## Flow control

```axolang
if a.is(null)
  then={set foo=bar}
  else={set foo=baz}

set foo=if(a.is(null) then=bar else=baz)

set found=false
while {found.is(false)} do={
  if find() then={set found=true}
}

try {
  if found.is(false) then={cry "bug"}
} catch={
  if n.error.is("bug")
    then={print "bug at line {n.line} will not raise higher"}
    else=n.cry
} finally={
  print "this will always run"
}
```

## Data structures

```axolang
set items=list(
  "of"
  some
  "items"
)

set items=list("of" some "items")
items.add "more" 3.14 "bar"
items.insert 0 "at" "start"
print items.-1
// Prints "bar"

items.each {print "{n.index}={n.value}"}
set foo=items.each({processed n.value})

set bar=map(
  color="violet"
  size=42
)
print bar.color

set bar=map(color="violet" size=42)
bar.set color="green"
bar.each {print "{n.name}={n.value}"}
```

## Objects

```axolang
set cat={map
  legs=4
  say={print "meow"}
}

set alice=cat()
print "alice has {alice.legs} legs and says:"
alice.say

set axolotl={
  set this=cat(legs=n.legs.or(4))

  this.set
    mutate={this.set(
      legs=n.legs.or(42)
      wings=2
    )}

    fly={
      if this.wings.is(null)
        then={cry "no wings yet"}
        else={print "flying..."}
    }

  this
}

set bob=axolotl()
bob.mutate
bob.fly
```

## Import

```axolang
import lib="../../lib.axo"
lib.func p0 p1 name=value

import lib="https://example.com/some/lib.axo"

import py="python3"
py.sort items key={processed p.0}
set os=py.import("os")
os.listdir().each {print n.value}
// That's how small Axolang regrows missing body parts!
```
