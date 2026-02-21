# axol

![axolotl](axolotl.png)

* Axolotl is a cute amphibian which can [regrow](https://en.wikipedia.org/wiki/Axolotl#Regeneration) missing body parts.
* Axol is a minimalist programming language simple to read, write, extend.
* It aims for a great user experience with as few core elements as possible, just as the vast diversity of atoms arises from only three particles: protons, neutrons, and electrons.
* Core elements of axol: `"strings"`, `[boxes]`, and `{functions}`.

axol version 0.4.16

# core

## string

```
print("hello world")
# hello world

a=1
print(a)
# 1

b="c"
print("{a}d{b}")
# 1dc

print("e\"f\{g\}h\\i\
  j
  k"
)
# e"f{g}h\ij
#   k

print("""
  e"f{g}h\ij
    k
""")
# e"f{g}h\ij
#   k

print(
  """
    l
    """
    m
  """
)
# l
# """
# m

print(
  "
    n
    \"
    o
  "
)
#
#   n
#   "
#   o
#

print(
  "p
    {b}
  q"
)
# p
#   c
# q
```

## box

```
b=["c" "d" key="val" m="n" o="p"]

print(b)
# ["c" "d" key="val" m="n" o="p"]

print(b.0)
# c

print(b.key)
# val

[
  pos=[q r="s" t="u"]
  kv=[key m="v" w="x"]
  rest=[y]
]=b

print(q r t)
# c d u

print(key m w)
# val n x

print(y)
# [o="p"]

print(["a" y...])
# ["a" o="p"]

print(["a" o="p" o="q"])
# ["a" o="q"]

[kv=[a b="c"]]=[]
# Error: `a` is required
```

See also: [box functions](#box-functions).

## function

```
f={a}

a="b"

print(f())
# b

print(f)
# {$key="f"}

print([f=f])
# [f={}]
```

Function [$key](#key) is printed instead of a usually long function code.

`f={$key="f"}` is printed as `f={}` for brevity.

### $

`$` is a box with the arguments the function was called with.

```
dup={
  val=$.0
  "{val}{val}"
}

print(dup("he"))
# hehe

greet={
  [pos=[name] kv=[how="hello"]]=$
  print("{how}, {name}!")
}

greet("Alice")
# hello, Alice!

greet("Bob" how="hi")
# hi, Bob!

echo={print($)}

echo("a" "b" c="d")
# ["a" "b" c="d"]

"a"|echo("b" c="d")
# ["a" "b" c="d"]

"a"|echo
# ["a"]
```

While valid, one-liner functions are less readable:

```
{val=$.0 "{val}{val}"}("no")
# nono
```

### $key

`$key` is the first key the function was set to.

```
f={print($key)}
f()
# f

g={
  [pos=[h]]=$
  h()
}

g(f)
# f

g({
  print($key)
})
# h

{
  print($key)
}()
# null
```

### $me

`$me` is the last box the function was got from.

```
f={print($me)}

f()
# null

a=["b" f=f]
c=["d" f=f]

a.f()
# ["b" f={}]

c.f()
# ["d" f={}]

f2=a|get("f")
f2()
# ["b" f={}]
```

`$me` can be overridden, it is not kept in the `$` box:

```
f($me=c)
# ["d" f={}]

e=["g" f={
  print($ $me)
}]

e.f(h="i")
# [h="i"] ["g" f={}]

e.f(j="k" $me=c)
# [j="k"] ["d" f={}]
```

See also: [oop](#oop).

### $here

`$here` is a box with keys and values set in the function.

```
a="b"

f={
  a="c"
  print($here)
}

f()
# [a="c"]

print($here)
# [a="b" f={}]
```

### $outer

`$outer` is a [$here](#here) box of a function that created the current function.

```
a="b"
f={print($outer)}
true|then({
  a="c"
  f()
})
# [a="b" f={}]
```

See also: [up](#up).

### $caller

`$caller` is a box the function was called from.

```
a="b"
f={print($caller)}
true|then({
  a="c"
  f()
})
# [a="c"]
```

See also: [up](#up), [throw](#throw).

# stdlib

## flow

### te

`te` reads as "then/else" or "ternary".

```
te={
  [pos=[cond thenVal elseVal]]=$
  if(cond
    then={thenVal}
    else={elseVal}
  )
}

print("foo"|te("yes" "no"))
# yes

print([]|te("yes" "no"))
# no
```

### if

```
if={
  [
    pos=[cond]
    kv=[then={} elif=[] else={}]
  ]=$
  native.ifThen(cond|bool {
    return(then() from=if)
  })
  native.ifThen(elif|bool {
    # This extra `ifThen` avoids eternal loop between this `if` and another `if` inside the `while`.
    while({elif} do={
      [pos=[cond then]]=elif|del(0 len=2)
      native.ifThen(cond|bool {
        return(then() from=if)
      })
    })
  })
  else()
}

if("cond1" then={
  print(1)
} elif=[{"cond2"} {
  print(2)
} {"cond3"} {
  print(3)
} {"cond4"} {
  print(4)
}] else={
  print(5)
})
# 1
# (try replacing "cond1" with "")

if(eq(sum(2 2) 4)
  then={print("ok")}
  else={print("why?")}
)
# ok

if(2|sum(2)|eq(4)
  then={"ok"}
  else={"why?"}
)|print
# ok

2|sum(2)|eq(4)|te("ok" "why?")|print
# ok
```

See also [from](#from) and [Error](#Error) for real `elif` examples.

### then

```
then={
  [pos=[cond do]]=$
  if(cond then=do else={cond})
}

2|sum(2)|eq(4)|then({
  print("ok")
})
# ok

print("a"|then({"b"}))
# b

print([]|then({"c"}))
# []
```

See also: [and](#bool).

### else

```
else={
  [pos=[cond do]]=$
  if(cond then={cond} else=do)
}

2|sum(2)|eq(4)|else({
  print("why?")
})

print("a"|else({"b"}))
# a

print([]|else({"c"}))
# c
```

See also: [or](#bool).

### case

```
case={
  [kv=[else={}]]=$
  items=$|pos
  expected=items|del(0)
  while({items} do={
    [pos=[val action]]=items|del(0 len=2)
    val|eq(expected)|else(continue)
    result=action()
    return(result from=case)
  })
  else()
}

input("sure? (yes/no): ")|case(
  "yes" {db.delete(all=true)}
  "no" {print("no problem")}
  else={print("assuming \"no\"")}
)
```

See also: [input](#input).

### loop

```
loop={
  do=$.0
  native.repeat({
    err=catch(do)
    err|then({
      [
        pos=[head=null val=null]
        kv=[from=do]
      ]=err
      from|eq(do)|else({err|throw})
      case(head
        break {return(val from=loop)}
        continue {}
        # `return` is handled natively by each `{}`, not just by `loop`
        else={err|throw}
      )
    })
  })
  null
}

loop({
  print("Press Ctrl+C to stop this")
})
```

See also: [break](#break), [continue](#continue), [return](#return).

### while

```
while={
  [pos=[cond] kv=[do=pause]]=$
  cond|is({})|else({
    throw("""
      replace while(condition)
      with while({condition})
      to check condition many times
    """)
  })
  loop({
    if(cond() then=do else=break)
  })
}

i=4
while({i} do={
  i|print(end=" ")
  up(i=i|sub(1))
})
# 4 3 2 1
```

### each

```
each={
  [pos=[items do]]=$
  loop({
    key=null
    [
      pos=[found key val]
    ]=native.getNextItem(items key)
    found|else(break)
    do(key=key val=val)
  })
}

["a" "b" c="d"]|each({
  print($.key $.val)
})
# 0 a
# 1 b
# c d
```

See also `each` definition in [pause](#pause).

### map

```
map={
  [pos=[items do={$.val}]]=$
  results=[]
  $|each({
    results|add(do($...))
  })
  results
}

print("hey"|map({
  "{$.key}={$.val}"
}))
# ["0=h" "1=e" "2=y"]

print(["a" b="c"].map({
  $.val|upper
})|join("")
# AC
```

### bool
### math

```
[false null 0 "" [] {}]|each({
  print(bool($.val) not($.val))
})
# false true
# (6 times)

print(bool("anything else"))
# true

[eq ne lt gt lte gte and or]|each({
  print($.val(2 3) end=" ")
})
# false true true false true false 3 2

[sum sub mul div mod pow]|each({
  print($.val(3 2) end=" ")
})
# 5 1 6 1.5 1 9
```

### throw
### catch
### trace

```
throw={
  err=$
  err.$trace|else({
    err.$trace=native.getTraceId()
  })

  caller=$caller
  while({caller} do={
    caller.$isCatching|then({
      return(err from=catch)
    })
    up(caller=caller.$caller)
  })

  print(err|trace("") file=os.stderr)
  [pos=[val=null] kv=[$trace] rest=[rest]]=err
  msg=if(rest
    then={[val rest...]}
    else={val}
  )
  print("Error: {msg}" file=os.stderr)
  os.exit(1)
}

catch={
  do=$.0
  $isCatching=true
  do()
  return(null)
}

err=catch({
  print(1)
  throw("a" b="c")
  print(2)
})
# 1
print("finally")
# finally

print(err)
# ["a" b="c" $trace=12345]

trace={
  [pos=[err format=""]]=$
  traceId=err.$trace
  items=native.getTraceItems(traceId)
  case(format
    [] {items}
    "" {
      items|map({
        [kv=[path line col at]]=$.val
        "{path} L{line} C{col}
          {at}"
      })|join("
      ")
    }
    else={throw("""
      format should be [] or ""
    """)}
  )
}

print(err|trace([]))
# [
#   [path="/path/app.axol" line=42 col=1 at="err=catch({"]
#   [path="/path/app.axol" line=44 col=3 at="throw(\"a\" b=\"c\")"]
# ]

print(err|trace(""))
# /path/app.axol L42 C1
#   err=catch({
# /path/app.axol L44 C3
#   throw("a" b="c")

err2=catch({
  throw(err)
})
print(err2|eq(err))
# true

throw(err)
# /path/app.axol L42 C1
#   e=catch({
# /path/app.axol L44 C3
#   throw("a" b="c")
# Error: ["a" b="c" $trace=12345]

throw("one string only")
# (trace)
# Error: one string only

throw("anything" else="here")
# (trace)
# Error: ["anything" else="here"]
```

### break
### continue
### return

```
break={throw(break $...)}
continue={throw(continue $...)}
return={throw(return $...)}

here={
  line=input()
  line|else(break)
  line|case(
    "skip" continue
    "ret" {return("ok" from=here)}
  )
  print(line)
}
loop(here)|print
# (input "ret")
# ok
```

See also: [loop](#loop)

## box functions

See also: [box](#box), [each](#each), [map](#map).

### add

```
add={
  [
    pos=[box]
    kv=[at=-1 flat=false]
    rest=[rest]
  ]=$
  vals=rest|pos
  rest|kv|then({
    throw("box|add(key=val) should be either:
      box.key=val
      box|up(key=val)
      box|add([key=val] flat=true)
    ")
  })
  native.add(box at vals flat)
}

b=["a" c="d"]

b|add("h")
print(b)
# ["a" "h" c="d"]

b|add("k" "l" at=0)
print(b)
# ["k" "l" "a" "h" c="d"]

b=["a" c="d"]
b|add([
  "h" [i="j"]
  c="k" l="m"
] flat=true)
print(b)
# ["a" "h" [i="j"] c="k" l="m"]
```

### get

```
get={
  [
    pos=[box key]
    kv=[len=null default=null]
  ]=$
  hasDefault="default"|in($|keys)
  native.get(box key len hasDefault default)
}

a=["b" "c" "d" e="f"]

print(a|get(0))
# b

print(a|get(0 len=1))
# ["b"]

print(a|get(1 len=2))
# ["c" "d"]

print(a|get("E"|lower))
# f

print(a|get("g"))
# Error: `g` is not found

print(a|get("g" default="h"))
# h
```

### set

```
set={
  [
    pos=[box key val]
    kv=[len=null]
    rest=[rest]
  ]=$
  rest|then({
    throw("add(vals...), set(key val)")
  })
  native.set(box key len val)
}

b=["a" "c" d="e"]

b|set("d" "f")
print(b)
# ["a" "c" d="f"]

b|set(0 len=2 ["g"])
print(b)
# ["g" d="f"]
```

### up

```
up={
  [pos=[box=null]]=$
  items=$|kv

  box|then({
    items|each({
      [kv=[key]]=$
      key|in(box|keys)|else({
        throw(KeyError(key))
      })
    })
    items|each({
      [kv=[key val]]=$
      box|set(key val)
    })
    return
  })

  plan=[]
  callerOfUp=$caller
  items|each({
    [kv=[key val]]=$
    cur=[box=callerOfUp]
    while({not(
      key|in(cur.box|keys)
    )} do={
      outerBox=cur.box.$outer
      outerBox.else({
        throw(KeyError(key))
      })
      cur.box=outerBox
    })
    plan|add([
      box=cur.box
      key=key
      val=val
    ])
  })

  plan|each({
    [kv=[box key val]]=$.val
    box|set(key val)
  })
}

a=1
{
  a=2
  print(a)
}()
# 2
print(a)
# 1

a=1
{
  up(a=2)
  print(a)
}()
# 2
print(a)
# 2

up(never=3)
# Error: `never` is not found

b=["a" c="d"]
b|up(c="e" f="g")
# Error: `f` is not found
```

See also: [$outer](#outer), [$caller](caller).

### del

```
del={
  [
    pos=[box key]
    kv=[len=null default=null]
  ]=$
  hasDefault="default"|in($|keys)
  native.del(box key len hasDefault default)
}

a=["b" "c" "d" e="f"]

print(a|del(1 len=2))
# ["c" "d"]

print(a)
# ["b" e="f"]

print(a|del("e"))
# f

print(a)
# ["b"]

print(a|del("e"))
# Error: `e` is not found

print(a|del("e" default="g"))
# g
```

### keys

```
keys={$.0|map({$.key})}

print(["a" b="c"]|keys)
# [0 "b"]
```

### vals

```
vals={$.0|map({$.val})}

print(["a" b="c"]|vals)
# ["a" "c"]
```

### pos
### kv
### len

```
pos={native.pos($.0)}
kv={native.kv($.0)}
len={native.len($.0)}

h=["a" "b" "c" d="e" f="g"]

print(h|pos)
# ["a" "b" "c"]

print(h|kv)
# [d="e" f="g"]

print(
  h|len
  h|pos|len
  h|kv|len
)
# 5 3 2
```

### find
### in

```
find={
  [pos=[where what]]=$
  where|each({
    $.val|eq(what)|else(continue)
    return($.key from=find)
  })
  null
}

print("abc"|find("c"))  # 2
print("abc"|find("d"))  # null

e=["a" "b" c="d"]
print(e|find("a"))  # 0
print(e|find("f"))  # null
print(e|find("d"))  # "c"

in={$.0|find($.1)|ne(null)}
print("a"|in(e))  # true
print("c"|in(e))  # false
print("c"|in(e|keys))  # true
```

### $call

```
a=["b" c="d" $call={
  print($me.c)
}]

a()
# d
```

### $str

```
b=[name="Bob" $str={
  "{$me.name} in the box"
}]

print(b)
# Bob in the box

b.name="Joe"
print("and {b} too")
# and Joe in the box too
```

See also: [Error](#Error)

## files

### import

Relative:
```
[kv=[foo bar]]=import("./baz.axol")

foo(bar)
```

Installed:
```
py=import("python")
```

* It finds the nearest file:
  * `./axol/lib/python.axol`
  * `../axol/lib/python.axol`
  * `../../axol/lib/python.axol`
  * `../../../axol/lib/python.axol`
  * ...
  * `~/.axol/lib/python.axol`
  * `/opt/axol/lib/python.axol`
* Such file may be a symlink (created by a package manager using config file) to the specific version in a shared cache, e.g. `~/.axol/0.42.1/lib/python/3.14.2.axol`.
* It contains auto-generated bindings from axol 0.42.1 to CPython 3.14.2 including its [built-in functions](https://docs.python.org/3/library/functions.html) like [`__import__`](https://docs.python.org/3/library/functions.html#import__) giving access to [Python standard library](https://docs.python.org/3/library/index.html) and installed [third-party packages](https://pypi.org/).

```
dt=py.__import__("datetime")

getNow={
  dt.datetime.now(dt.UTC).isoformat()
}

print(getNow())
# 2026-12-31T23:59:59Z
```

### print

```
print={
  [kv=[file=os.stdout sep=" " end="
  "]]=$
  file.write(
    $|pos|join(sep)|sum(end)
  )
}

print("a" "b")
# a b

print("a" "b" sep="" end="")
print("c")
# abc
```

### input

```
input={
  [kv=[
    end=""
    file=os.stdin
  ] rest=[prompt]]=$
  rest|then({
    print(prompt... end=end)
  })
  file.readLine()
}

answer=input("question: ")
print(answer)
# (the line you've entered)
```

See also: [case](#case).

### cli
### env

```
env=os.getEnv()
cli=os.getCli()

# KEY=VAL ./app.axol a -bc --d=e --foo

print(cli|pos)
# ["./app.axol" "a" "-bc" "--d=e" "--foo"]

print(cli|kv)
# [b=true c=true d="e" foo=true]

print(cli.d)
# e

print(env.KEY)
# VAL

print(env.HOME)
# /home/me
```

## singleton
### log

```
log=[]
log.names="debug info warn error crit fatal"|split(" ")
log.letters=log.names|map({
  $.val.0|upper
})
log.levels=[]  # debug=0 info=1 ...
log.names|each({
  log.levels|set($.val $.key)
}]
log.level=log.levels.info
log.print=print
log.do={
  [pos=[level] rest=[vals]]=$
  level|lt(log.level)|then(return)
  letter=log.letters|get(level)
  log.print("{getNow()} {letter} {vals}")
}
log.levels|each({
  level=$.val
  log|set($.key {
    log.do(level $...)
  })
})

log.debug("invisible")

log.info("seen")
# 2026-12-31T23:59:59Z I ["seen"]

log.error("summary" details=[])
# 2026-12-31T23:59:59Z E ["summary" details=[]]
```

## decorator
### logged

```
time=py.__import__("time")

logged={
  [pos=[func] kv=[
    level=log.levels.debug
  ]]=$

  {
    args=$
    result=null

    start=time.time()
    err=catch({
      up(result=func(args...))
    })
    stop=time.time()

    log.do(level
      func=func
      args=args
      time=stop|sub(start)
      result=result
      err=err
    )

    err|then({err|throw})
    result
  }
}

foo=logged({
  something("slow" $...)
})

foo("bar")
# (logs the details)
```

## context manager
### with
### $without
### File

See also: [oop](#oop).

```
File=type({
  [pos=[path] kv=[mode="r"]]=$
  file=[]
  file.handle=os.open(path mode)
  ["write" "read" "seek" "truncate" "close"]|each({
    actionKey=$.val
    action=os|get(actionKey)
    file|set(actionKey {
      action($me.handle $...)
    })
  })
  file.$without={$me.close()}
  file
})

with={
  [kv=[do] rest=[items]]=$
  result=null
  err=catch({
    up(result=do(items...))
  })
  items|each({
    catch({
      $.val.$without()
    })
  })
  err|then({err|throw})
  result
}

with(
  input=File("input.txt" mode="r")
  result=File("result.txt" mode="w")
  do={
    $.input.read()|$.result.write
    throw("test err")
  }
)
# (both files are auto-closed)
# Error: test err

with(mutex.read() do={
  # throw or not
  if(needed then={
    with(mutex.write() do={
      # throw or not
    })
    # (write-lock is auto-released)
  })
  # throw or not
})
# (read-lock is auto-released)
```

## oop

See also: [$me](#me).

### type
### $type
### $parent

```
rootType={[]}

type={
  [
    kv=[of=rootType]
    pos=[make=rootType]
  ]=$
  $type=[
    of...
    $of=of
    $key=$key
  ]
  $type.$call={
    parent=of($...)
    onlyMe=make($...)
    me=[of...]
    me|del("$call")
    me|add([
      parent...
      onlyMe...
      $parent=parent
      $type=$type
    ] flat=true)
    me
  }
  $type
}

Animal=type({[
  isAwake=true
  talk={
    $me.isAwake|then(return)
    throw("too sleepy")
  }
]})

Animal.getTaxonomy={
  cur=[type=$me.$type|or($me)]
  taxonomy=[]
  while({cur.type|and(
    cur.type|ne(rootType)
  )} do={
    taxonomy|add(cur.type.$key)
    cur.type=cur.type.$of
  })
  taxonomy
}

Cat=type(of=Animal {
  [kv=[mood=2]]=$
  [
    mood=mood
    talk={
      $me.$parent.talk(
        $...
        $me=$me
      )
      print(["meow"]\
        |mul($me.mood)\
        |join(" ")
      )
    }
  ]
})

bob=Cat()

print(bob)
# [
#   getTaxonomy={}
#   isAwake=true
#   mood=2
#   talk={}
#   $type={$key="Cat"}
#   $parent=[
#     getTaxonomy={}
#     isAwake=true
#     talk={}
#     $type={$key="Animal"}
#     $parent=[]
#   ]
# ]

bob.talk()
# meow meow

bob.isAwake=false
bob.talk()
# Error: too sleepy

print(bob.$type|eq(Cat))
# true

print(bob.$parent.$type|eq(Animal))
# true
```

### is

```
is={
  [
    pos=[item expected=null]
    kv=[typeOf=null]
  ]=$
  itemType=item.$type
  typeOf|then({
    while({itemType} do={
      itemType|eq(typeOf)|then({
        return(true from=is)
      })
      up(itemType=itemType.$of)
    })
    return(false from=is)
  })
  expectedType=expected.$type|or(
    expected  # it is type already
  )
  itemType|eq(expectedType)
}

print({42}|is({}))
# true

print({42}|is(""))
# false

print("42"|is(""))
# true

{
  print($|is([]))
}()
# true

print(bob|is(Cat))
# true

print(bob|is(Animal))
# true

print(Cat|is(typeOf=Animal))
# true

Lion=type(of=Cat)
simba=Lion(mood=1)

print(simba.getTaxonomy())
# ["Lion" "Cat" "Animal"]

print(Lion.getTaxonomy())
# ["Lion" "Cat" "Animal"]
```

### Error
### KeyError

```
Error=type()

KeyError=type(of=Error {
  [pos=[key]]=$
  [
    key=key
    $str={"`{$me.key}` is not found"}
  ]
})

err=KeyError("foo")

print(err.$key err.key)
# KeyError foo

throw(err)
# Error: `foo` is not found

if(err|is(KeyError) {
  print("ok")
} elif=[{err|is(Error)} {
  print("it is, but matched above")
}] else={err|throw})
# ok
```

## concurrency

### pause
### from
### $next

```
pause={native.pause($.0)}

from={
  [pos=[i] kv=[to=null step=1]]=$

  cmp=if(to|eq(null) then={
    {true}
  } elif=[{step|gte(0)} {
    lte
  }] else={
    gte
  })

  while({i|cmp(to)} do={
    msg=pause(i)
    msg.restart|ne(null)|then({
      up(i=msg.restart)
      continue()
    })
    up(i=i|sum(step))
  })
  return(i)
}

a=from(1 to=3)
print(a)
# [$next={}]

print(a.$next())
# 1

print(a.$next())
# 2

print(a.$next(restart=2))
# 2

print(a.$next())
# 3

err=catch({
  print(a.$next())
})

print(err.0|eq(pause))
# true

print(err.result)
# 4

each={
  [pos=[items do]]=$
  items.$next|then({
    loop({
      item=null
      err=catch({
        up(item=items.$next())
      })
      err|then({
        err.0|eq(pause)|then(break)
        throw(err)
      })
      do(val=item)
    })
    return(from=each)
  })
  # (main `each` code here)
}

from(1 to=100)|each({print($.val)})
# 1
# 2
# ...
# 100

from(0 step=-2)|each({print($.val)})
# 0
# -2
# -4
# (and so on until Ctrl+C)

print(from("no" to="noooo" step="o")|map...)
# no noo nooo noooo
```

### Task

```
tasks=[]

Task=type({
  [pos=[func] rest=[args]]=$
  task=[result=null]
  task.err=catch({
    task.result=func(args...)
  })
  task.done=bool(task.err)|or(not(
    task.result.$next
  ))
  task.done|else({tasks|add(task)})
  task
})
```

See [mainLoop](#mainLoop).

### mainLoop

This is the main loop which executes all tasks,
including the `mainTask` auto-created from the main code of the app.

```
mainLoop={
  while({tasks} do={
    tasks|each({
      key=$.key
      task=$.val
      task.done|then({
        tasks|del(key)
        continue()
      })
      err=catch({
        task.result.$next()
      })
      err|else(continue)
      if(err.0|eq(pause)
        then={
          task.result=err.result
          if(task.result.$next
            then=continue
          )
        }
        else={
          task.result=null
          task.err=err
        }
      )
      task.done=true
      tasks|del(key)
    })
  })
}

task=Task(from 1 to=3)

print(task)
# [
#   done=false
#   result=[$next={}]
#   err=null
#   $type={$key="Task"}
#   $parent=[]
# ]

pause()
```

* The `mainTask.result.$next()` returns in the `mainLoop`.
* The `mainLoop` checks other tasks,
* and then calls `mainTask.result.$next()` again,
* so this `pause()` returns here in the `mainTask`.

```
print(task)
# [done=true result=4 err=null
# $type={$key="Task"} $parent=[]]

f={
  print($.0)
  42
}
instant=Task(f "ok")
# ok

print(instant)
# [done=true result=42 err=null
# $type={$key="Task"} $parent=[]]
```

### await

```
await={
  [kv=[done=null err=1 ok=null]]=$
  need=[done=done err=err ok=ok]
  have=[done=0 err=0 ok=0]

  tasks=[]
  $|pos|each({
    task=$.val
    task|is(Task)|else({
      throw("not Task: {task}")
    })
    tasks|add(task)
  }]

  do={
    tasks|each({
      key=$.key
      task=$.val

      task.done|then({
        have.done=have.done|sum(1)
        if(
          need.done|ne(null)|and(
          have.done|gte(need.done))
          then={break(from=do)}
        )

        if(task.err
          then={
            have.err=have.err|sum(1)
            if(
              need.err|ne(null)|and(
              have.err|gte(need.err))
              then={task.err|throw}
            )
          }

          else={
            have.ok=have.ok|sum(1)
            if(
              need.ok|ne(null)|and(
              have.ok|gte(need.ok))
              then={break(from=do)}
            )
          }
        )

        tasks|del(key)
        continue()
      })
      pause()  # let mainLoop work
    })
  }
  while({tasks} do=do)

  if($|len|eq(1)
    then={$.0.result}
    else={null}
  )
}

one=Task(from 1 to=10)
another=Task(from 2 to=20)
await(one another)

print(one.done another.done)
# true true

print(one.result another.result)
# 11 21

result=await(Task(from 3 to=30))
print(result)
# 31

bad=Task(from 4 to=40 step={})
print(bad)
# [
#   done=false
#   result=[$next={}]
#   err=null
#   $type={$key="Task"}
#   $parent=[]
# ]

await(bad)
# (trace to `from()`)
#   up(i=i|sum(step))
# Error: cannot sum(4 {$key="step"})
```

### async

```
async={func=$.0 {
  Task(func $...)
}}

afrom=async(from)

print(await(afrom(5 to=50)))
# 51

afoo=async({
  # foo
})
```
