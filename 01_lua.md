## Some notes on Lua
**Comments** in Lua start with two dashes `--`. These may appear at the start of
a line or anywhere along a line.  
The **local** keyword is a way to limit the scope of a variable or function.
You can ignore its exact meaning for now, but it may be helpful to realise that
whenever you see `local` it means we're defining a variable.

Lua doesn't have many data types:
1.  **Nil** typically indicates the absence of any useful value. Setting a
variable to equal nil is effectively the same as deleting the variable.
`var = nil`
2.  **Boolean** can have one of two values: true or false. Both nil and false
make a condition false; any other value is 'truth-y' (including the number 0).
`var = true` or `var = false`
3.  **Number** is any integer or real (floating-point) number - Lua does not
differentiate between the two.  
`n = 36`  
`o = 3.14159`
4.  **String** is an immutable collection of 8-bit clean bytes.  
`str = "Hello, world!"`

The only compound data structure in Lua is the **table**. Tables are always a
collection of zero or more key/value pairs. The keys may be specified or
inferred (in which case Lua uses integers starting from 1). The values may be
any value, including nil or other tables.  
Tables are created (initiated) using curly braces:  
`my_table = {}`  
Values can be assigned at the point of creation:
```
my_table = {
    "A string",
    "Tables can also take numbers",
    42,
    {
        "or even",
	"other tables!"
    },
}
```
Since I'm not specifying keys, each field of this table gets an integer index
starting from 1: my_table[3] equals 42.  
`my_table.favourite_animal = "chicken"`  
This assigns the string value 'chicken' to a new field 'favourite_animal' of
our table. Lua doesn't care that the table so far only contained integer
indices - the new key 'favourite_animal' with its value 'chicken' is simply
added to the table.

Thanks to a little syntax magic, *string keys* in a table can be queried using
dot notation as well as 'typical' key indexing. In other words, `table.field`
and `table["field"]` are equivalent.  
Since Lua functions can be passed around just like any other values, this means
you can store a function as a value in a table - and then call the function by
indexing the table. This looks (and works) exactly like calling a method of a
class in many other languages:
```
table = {}
table.greet = function() print("Hello, world!") end
table.greet()  -- prints "Hello, world!"
```
You will see this structure and syntax a lot all throughout rc.lua. Whenever
AwesomeWM needs to call one of its own functions, as opposed to builtin
functions such as `print()` or `require()`, it will be indexing some table.

### Helpful sources
(The Lua 5.3 reference
manual)[https://www.lua.org/manual/5.3/contents.html#contents] - don't worry
if your copy of Awesome is built against a later version of Lua! All of the
manual still applies.  
(Learn Lua in one video)[https://youtu.be/iMacxZQMPXs] by Derek Banas covers
all of Lua's basic functionality in under one hour

Let's go over rc.lua section by section in the next chapter.
