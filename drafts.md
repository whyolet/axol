# axol drafts

These are the "work in progress" drafts on their way to [axol](https://github.com/whyolet/axol#axol)

## [$type](#type)
## [$parent](#parent)

```axol
$import "box eq Null Bool Number String Action Box"

null.$type|eq(Null)
false.$type|eq(Bool)
true.$type|eq(Bool)
-3.14.$type|eq(Number)
"string".$type|eq(String)
aaa: "bbb"
aaa.$type|eq(Action)
box(aaa="bbb").$type|eq(Box)
```
* Every [value](#value) has a type, reserved name `$type` points to.
* Names of types are TitleCased (unlike snake_cased other names) to allow `aaa = Aaa()`
* Each type is a box with at least `$parent` name pointing to a parent type:
```axol
$import "box Error"

Aaa=box($parent=Error)
```
* Standard library of axol contains the next tree of types:
  * `Object`
    * `Null`
    * `Bool`
    * `Number`
    * `String`
    * `Action`
    * `Box`
    * `Error`
      * `NameNotFound`
      * `FileNotFound`
      * `NoLength`
      * `NotFindable`
      * `NotCallable`
      * etc.
* Root of this tree is looped as `Object.$parent|eq(Object)`
* If a type box also has a [$call](#call) name, then a call of this type should produce objects of this type, usually from the arguments passed:
```axol
$import "box eq print Action Number"

print Number("101010" base=2) # 42

aaa=Action("print 42")
aaa() # 42

DoubleAction=box
  $parent=Action
  $call:
    code=@0
    result=Action("{code}\n{code}")
    result.$type=DoubleAction
    result

bbb=DoubleAction("print 42")
bbb()
# 42
# 42

print bbb.$type|eq(DoubleAction) # true
```

## [is](#is)

Given the last example in [$type](#type) section:
```axol
print bbb|is(DoubleAction) # true
print bbb|is(Action) # true
print bbb|is(Object) # true
print bbb|is(Number) # false

print DoubleAction|is(Action) # true
print DoubleAction|is(Object) # true
print DoubleAction|is(Number) # false
```
* Set `left=@0` and `right=@1`
* If `left` is not a type:
  * Set `left=left.$type`
* Loop:
  * If `left|eq(right)` then return `true`.
  * Set `left=left.$parent`
  * If `left|eq(Object)` then return `right|eq(Object)`

## [type](#type)
## [$this](#this)

```axol
$import "box is join nam_args mul print Error"

type
  Animal:
    nam_args "word" times=1
    $parent
      word=word
      times=times
      _sep="-"

  talk:
    print box($this.word)
      |mul($this.times)
      |join($this._sep)

type of=Animal
  Cat:
    nam_args times=2
    $parent word="meow" times=times
  
  smile:
    print "=^..^= {$this.word}"

alice=Cat()
alice.talk() # meow-meow
alice.smile() # =^..^= meow

alice|is(Cat)
alice|is(Animal)
Cat|is(Animal)

type of=Error "NotFound Invalid"
```
* `type` calls above are the shortcuts for:
```axol
Animal=box
  $parent=Object
  $call:
    nam_args "word" times=1
    $this=Animal.$parent
      word=word
      times=times
      _sep="-"
    $this.$type=Animal
    this.talk:
      print box($this.word)
        |mul($this.times)
        |join($this._sep)
    $this

Cat=box
  $parent=Animal
  $call:
    nam_args times=2
    $this=Cat.$parent
      word="meow"
      times=times
    $this.$type=Cat
    $this.smile:
      print "=^..^= {$this.word}"
    $this

NotFound=box
  $parent=Error
  $call:
    NotFound.$parent($args...)

Invalid=box
  $parent=Error
  $call:
    Invalid.$parent($args...)
```

## [try](#try)
## [Error](#Error)

```axol
$import "box pos_args try Error"

try
  do: Error("aaa").cry()
  catch: print("Caught: @0")
  finally: print("Always")

type of=Error "NotFound"

found=false
try
  do:
    if not(found)
      then: NotFound("Value of 'found' is not true").cry()
    print "This will never run"
  catch:
    pos_args "err"
    if err|is(NotFound)
      then:
        print "{err.$type} at {err.file} L{err.line}:C{err.column}\n{err.args}"
       else: err.cry()
  finally:
    print "This will always run"

```
* Run the `do` action.
    * When an object of `Error` is created, it stores current `file, line, column`, can store args.
    * To raise this error, call its `.cry()` action.
* If the error is raised, run the `catch` action, passing this error as the only positional argument.
    * The error can be reraised by calling its `.cry()` action.
* Uncaught error is printed to stderr, app exits with code 1.

## TODO

* how an object of error gets its class name?
* make links to [is](#is) from `length`, `in`, etc.
* rename `bool` to `Bool`
* add sections for `Number` etc
* `mul` and `plus` of box
* `aaa($args...)`
* `$str: "..."`

## raw

```
$code string
$line number
$column number
$stack box with boxes being $args:

aaa(bbb(ccc ddd) eee)
012345678901234567890
          1         2
          
$column=4
$stack=box
  box()
  box(ccc ddd)

$column=17
$stack=box
  box(fff eee)

what points to scopes?
calls|set $id call

call is not a box like $local

call is a coroutine, paused execution.
it contains:
$local, 
instruction pointer,
and all nameless values, e.g:
foo=bar()|plus(baz())
if result of bar() is already known,
where it is stored?
on the stack...

pass call to v as a0|foo

return
break
continue
pause
resume

return/break -> end
continue/resume -> next

# Find if the object has any of values passed,
# e.g. ("foo" "bar" "baz"):hasAny("bar" "qux")
set hasAny={
  set func=here
  set object=a.0 values=a:get(from=1)
  values:each {
    if a.value:in(object) {
      func.return true
    }
  }
  false
}

while {true} {
  set loop1=here
  foo():each {
    set loop2=here
    if bar(a.value) then=loop2.continue
    if baz(a.value) then=loop1.break
  }
}

break, continue, return, pause, resume -
are implemented as cry
caught by function and loops if scope matches:

set each={
  set object=a.0 action=a.1
  set func=here s=(index=0, scope=null)
  set length=object:len
  while {length:eq(null):or(s.index:lt(length))} {
    set value=if(length:eq(null)
      then=try(
        object.resume
        catch={
          set err=a
          if err.a.type:eq("stop") then=func.return
          err.cry()
        }
      )
      else={object:get(s.index)}
    )
    try {
      s:set scope=here
      action name=s.index value=value
    } catch={
      set err=a
      if err.scope.outside:eq(s.scope) {
        if err.a.type:eq("break") then=func.return
        if not(err.a.type:eq("continue") then=err.cry
      } else=err.cry
    }
    s:set index=s.index:plus(1)
  }
}

set range={
  set func=here
  set s=(index=a.from:or(0))
  set to=a.to:or(a.0):or(-1)
  func.pause func
  while {s.index < to} {
    func.pause s.index
    s:set index=s.index:plus(1)
  }
}  

range(from=4 to=10:pow(6)):each {print a.value}

set foo={
  print 2 a.0
  print 4 here.pause(here)
  print 7 here.pause(5)
}
set bar=foo(1)
# Prints: 2 1
set baz=bar.resume(3)
# Prints: 4 3
print baz
# Prints: 5
bar.resume 6
# Prints: 7 6

set foo={
  print 2 a.0
  print 4 here.pass(here)
  print 7 here.pass(5)
  here.pass 8
}
set bar=foo(1) # 2 1
print bar.pass(3) # 4 3, 5
print bar.pass(6) # 7 6, 8

set counter={
  set s=(i=0)
  return (next={
    print a.0 # foo, bar
    s:set i=s.i:plus(1)
    return s.i
  })
}
set c=counter()
print c.next("foo") # 1
print c.next("bar") # 2

can we use this simple approach instead of pause/resume?

async/await

with
open
decorator
```


