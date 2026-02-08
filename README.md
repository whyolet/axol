![Axolotl](axolotl.png)

# [axol](#)

* Minimalist programming language simple to read, write, extend.
* Axol is named after the axolotl animal for its ability to regrow missing body parts and for being cute.
* Version: 0.4.0

# [draft](#draft)

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

f={b}

print(f)
# {1234567890}
# (function address is shown
# instead of function code
# to be able to compare functions,
# that may have the same code,
# yet they are still not the same function)

print(f)
# {}
# (to keep docs simple,
# let's skip the address...)

print(f)
# f
# (...or just show the function name)

print(f())
# c
# (because f={b} and b="c")

dup={
  val=$.0
  "{val}{val}"
}

print(dup("he"))
# hehe

g=["he"]

print(g)
# ["he"]

print(g.0)
# he

h=["i" "j" key="val" m="n" o="p"]

print(h.1)
# j

print(h.key)
# val

[
  pos=[q r="s" t="u"]
  kv=[key m="v" w="x"]
  rest=[y]
]=h

print(q r t)
# i j u

print(key m w)
# val n x

print(y)
# [o="p"]

print(["a" y...])
# ["a" o="p"]

print(["a" o="p" o="q"])
# ["a" o="q"]

greet={
  [pos=[name] kv=[how="hello"]]=$
  print("{how}, {name}!")
}

greet("Alice")
# hello, Alice!

greet("Bob" how="hi")
# hi, Bob!

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

echo={print($)}

"a"|echo("b" c="d")
# ["a" "b" c="d"] 

then={
  [pos=[cond do]]=$
  if(cond then=do)
}

2|sum(2)|eq(4)|then({
  print("ok")
})
# ok

else={
  [pos=[cond do]]=$
  if(cond else=do)
}

2|sum(2)|eq(4)|else({
  print("why?")
})

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

while={
  [pos=[cond] kv=[do={}]]=$
  cond|is({})|else({
    throw("replace while(condition ...) \
      with while({condition} ...) \
      to check it many times")
  })
  loop({
    if(cond() then=do else=break)
  })
}

i=4
while({i} do={
  i|print
  up(i=i|sub(1))
})  # 4 3 2 1

[kv=[foo]]=import("./lib.axol")
foo()

py=import("python")
# Not relative.
# Finds nearest parent dir with
# `axol/python.axol` file,
# which may be a symlink
# (created by a package manager)
# to the specific version file like
# `axol/python/3.14.2.axol`,
# which has auto-generated bindings to CPython 3.14.2
# including its built-in functions
# https://docs.python.org/3/library/functions.html
# like `__import__("package")`
# https://docs.python.org/3/library/functions.html#import__
# giving access to python standard library
# https://docs.python.org/3/library/index.html
# and installed third-party packages.

dt=py.__import__("datetime")

getNow={
  dt.datetime.now(dt.UTC).isoformat()
}

print(getNow())
# 2026-12-31T23:59:59Z

log=[]
log.levelNames="debug info warn error"|split(" ")
log.levelLetters=log.levelNames\
  |map({$.val.0|upper})|join("")
log.levels=[]
log.levelNames|each({
  log.levels[$.val]=$.key
}]
log.level=log.levels.info
log.print=print
log.do={
  [pos=[level] rest=[vals]]=$
  level|lt(log.level)|then(return)
  letter=log.levelLetters[level]
  log.print("{getNow()} {letter} {vals}")
}
log.levels|each({
  level=$.val
  log[$.key]={log.do(level, $...)}
})

log.debug("invisible")

log.info("seen")
# 2026-12-31T23:59:59Z I ["seen"]

log.error("summary" details=[])
# 2026-12-31T23:59:59Z E ["summary" details=[]]

print={
  [kv=[sep=" " end="""

  """]]=$
  # $|pos|each({...})
}

print("a" "b")
# a b

print("a" "b" sep="" end="")
print("c")
# abc

input("sure? (yes/no): ")|case(
  "yes" {delete(all=true)}
  "no" {print("no problem")}
  else={print("assuming \"no\"")}
)

print(env.HOME)
# /home/me

# ./app.axol a -bc --d=e --foo
print(cli)
# ["./app.axol" "a" "-bc" "--d=e" b=true c=true d="e" foo=true]

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

a=["b" "c" "d" e="f"]
print(a.0 a[1 3] a.e a["e"] a["g" default="h"] end=" ")
# b ["c" "d"] f f h

print(a|del(1 3 default=[]))
# ["c" "d"]
print(a)
# ["b" e="f"]

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

b|add("h" i="j")
print(b)
# ["a" "h" c="d" i="j"]

b|add("k" "l" at=0)
print(b)
# ["k" "l" "a" "h" c="d" i="j"]

c=["d" e="f"]
g=["h" [i="j"] k="l"]
c|add(g flat=true)
print(c)
# ["d" "h" [i="j"] e="f" k="l"]

keys={$.0|map({$.key}))
print(["a" b="c"]|keys)
# [0 "b"]

vals={$.0|map({$.val}))
print(["a" b="c"]|vals)
# ["a" "c"]

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

e.$call={print("ok")}
e()
# ok

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

print(err|trace([]))
# [
#   [path="/path/app.axol" line=42 col=1 at="e=catch({"]
#   [path="/path/app.axol" line=44 col=3 at="throw(\"a\" b=\"c\")"]
# ]

print(err|trace(""))
# /path/app.axol L42 C1
#   e=catch({
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
# Error: one string only

throw("anything" else="here")
# Error: ["anything" else="here"]

break={throw(break $...)}
continue={throw(continue $...)}
return={throw(return $...)}

# loop={
# each={
# map={
#   ...
#   err=catch(do)
#   err|then({
#     err.0|case(
#       break {...}
#       continue {...}
#       return {...}
#       else={throw(err)}

here={
  line=input()
  line|else(break)
  line|eq("skip")|then(continue)
  line|eq("ret")|then({
    return("ok" from=here)
  })
  print(line)
}
loop(here)|print
# (input "ret")
# ok

time=py.__import__("time")

# decorator
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

a={$}
print(a("b" c="d"))
# ["b" c="d"]

e=[a={$}]
print(e.a("b" c="d"))
# ["b" c="d" $me=e]

rootType={[]}

type={
  [
    kv=[of=rootType]
    pos=[make=rootType]
  ]=$
  $type=[of... $of=of]
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
    $.$me.isAwake|then(return)
    throw("too sleepy")
  }
]})

Animal.getTypes={
  item=$.$me.$type|or($.$me)
  items=[]
  while({item|and(
    item|ne(rootType)
  )} do={
    items|add(item)
    up(item=item.$of)
  })
  items
}

Cat=type(of=Animal {
  [kv=[mood=2]]=$
  [
    mood=mood
    talk={
      $.$me.$parent.talk(
        $...
        $me=$.$me
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
#   getTypes={}
#   isAwake=true
#   mood=2
#   talk={}
#   $type=Cat
#   $parent=[
#     getTypes={}
#     isAwake=true
#     talk={}
#     $type=Animal
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

print(bob|is(Cat))
# true

print(bob|is(Animal))
# true

print(Cat|is(typeOf=Animal))
# true

Lion=type(of=Cat)
simba=Lion(mood=1)

print(simba.getTypes())
# [Lion Cat Animal]

print(Lion.getTypes())
# [Lion Cat Animal]

File=type({
  [pos=[path] kv=[mode="r"]]=$
  file=[]
  file.handle=fs.open(path mode)
  ["write" "read" "seek" "truncate"]|each({
    action=$.val
    file[action]={
      fs[action]($.$me.handle $...)
    }
  })
  file.$close={
    fs.close($.$me.handle)
  }
  file
})

with={
  [pos=[do] rest=[items]]=$
  result=null
  err=catch({
    up(result=do(items...))
  })
  items|each({$.val.$close()})
  err|then({err|throw})
  result
}

with(
  input=File("input.txt" mode="r")
  result=File("result.txt" mode="w")
{
  $.input.read()|$.result.write
  throw("test err")
})
# (both files are auto-closed)
# Error: test err

seq={
  [pos=[start stop] kv=[step=1]]=$
  i=start
  cmp=if(step|gte(0)
    then={lte}
    else={gte}
  )
  while({i|cmp(stop)} do={
    msg=pause(i)
    msg.restart|ne(null)|then({
      up(i=msg.restart)
      continue()
    })
    up(i=i|sum(step))
  })
}

a=seq(1 3)
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
  # ...
}

seq(1 100)|each({print($.val)})
# 1
# 2
# ...
# 100

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

# This is the main loop,
# which executes all tasks,
# including `mainTask` auto-created
# from the main code of the app.
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

task=Task(seq 1 3)

print(task)
# [
#   done=false
#   result=[$next={}]
#   err=null
#   $type=Task $parent=[]]

pause()
# (mainTask.result.$next() returns
# to mainLoop, which checks other tasks,
# and then calls mainTask.result.$next() again,
# so this pause() returns here)

print(task)
# [done=true result=4 err=null
# $type=Task $parent=[]]

f={
  print($.0)
  42
}
instant=Task(f "ok")
# ok

print(instant)
# [done=true result=42 err=null
# $type=Task $parent=[]]

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

one=Task(seq 1 10)
another=Task(seq 2 20)
await(one another)

print(one.done another.done)
# true true

print(one.result another.result)
# 11 21

result=await(Task(seq 3 30))
print(result)
# 31

bad=Task(seq 4 40 step={})
print(bad)
# [
#   done=false
#   result=[$next={}]
#   err=null
#   $type=Task $parent=[]]

await(bad)
# (trace to seq)
#   up(i=i|sum(step))
# Error: cannot sum(4 {})

async={
  func=$.0
  {Task(func $...)}
}

aseq=async(seq)

print(await(aseq(5 50)))
# 51

afoo=async({
  # foo
})
```
