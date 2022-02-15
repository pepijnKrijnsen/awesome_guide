#The Awesome Guide

The default rc.lua file is well commented, but AwesomeWM has some
inconsistencies that may make it difficult to read for new users. Learning the
basics is a great way of understanding more complicated concepts later.
I think it will be helpful to briefly highlight some common Lua patters found in
rc.lua, for those unfamiliar with the language. So let's start with that!

##Some notes on Lua
**Comments** in Lua start with two dashes `--`. These may appear at the start of
a line or anywhere along a line.
The **local** keyword is a way to limit the scope of a variable or function.
You can ignore its exact meaning for now, but it may be helpful to realise that
whenever you see `local` it means we're defining a variable.
Lua doesn't have manu data types:
1.  **Nil** typically indicates the absence of any useful value. Setting a
variable to equal nil is effectively the same as deleting the variable.
`var = nil`
2.  **Boolean** can have one of two values: true or false. Both nil and false
make a condition false; any other value is 'truth-y' (including the number 0).
`var = true` or `var = false`
3.  **Number** is any integer or real (floating-point) number - Lua does not
differentiate between the two.
n = 36
o = 3.14159
4.  **String** is an immutable collection of 8-bit clean bytes.
str = "Hello, world!"
The only compound data structure in Lua is the **table**. Tables are always a
collection of zero or more key/value pairs. The keys may be specified or
inferred (in which case Lua uses integers starting from 1). The values may be
any value, including nil or other tables.
Tables are created (initiated) using curly braces:
`my_table = {}`
Values can be assigned at the point of creation, or afterwards, like this:
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

Now, let's go over rc.lua section by section!

##Loading the standard library
```
-- If LuaRocks is installed, make sure that packages installed through it are
-- found (e.g. lgi). If LuaRocks is not installed, do nothing.
pcall(require, "luarocks.loader")

-- Standard awesome library
local gears = require("gears")
local awful = require("awful")
require("awful.autofocus")
-- Widget and layout library
local wibox = require("wibox")
-- Theme handling library
local beautiful = require("beautiful")
-- Notification library
local naughty = require("naughty")
local menubar = require("menubar")
local hotkeys_popup = require("awful.hotkeys_popup")
-- Enable hotkeys help widget for VIM and other apps
-- when client with a matching name is opened:
require("awful.hotkeys_popup.keys")
```
The `require()` function in Lua loads a module from a different file into the
current file. The `pcall()` function is like a try/catch block in other
languages: it allows the code to continue executing should the function being called
inside `pcall()` return an error.
In the very first line (aside from the comments), Lua tries to `require()` (load
the module) `luarocks.loader`. Since this call is wrapped inside `pcall()`, if
the call to `require()` fails, rc.lua will continue executing as if nothing is
wrong.
Libraries in Lua have funny names. 'gears' makes some sense - it's a module full
of helper functions, like the gears in a clock making sure everything works -
but what would something called 'awful' add to the program?
All that these lines of code do, is assign the contents of modules (or rather,
the return values of the result of executing those modules) to a variable inside
rc.lua.
The only exception is the very last line, where `awful.hotkeys_popup.keys` is
required but not assigned to any variable. This is because 'keys' is a table
with references to some apps that get special hotkeys assigned to them in
AwesomeWM. The table with those references can be made available directly,
instead of assigning it to a variable.

##Error handling
```
-- {{{ Error handling
-- Check if awesome encountered an error during startup and fell back to
-- another config (This code will only ever execute for the fallback config)
if awesome.startup_errors then
    naughty.notify({ preset = naughty.config.presets.critical,
                     title = "Oops, there were errors during startup!",
                     text = awesome.startup_errors })
end

-- Handle runtime errors after startup
do
    local in_error = false
    awesome.connect_signal("debug::error", function (err)
        -- Make sure we don't go into an endless error loop
        if in_error then return end
        in_error = true

        naughty.notify({ preset = naughty.config.presets.critical,
                         title = "Oops, an error happened!",
                         text = tostring(err) })
        in_error = false
    end)
end
-- }}}
```
When AwesomeWM loads, it tries to load user-specific configuration. At that
point, one of three things may happen:
1. AwesomeWM encounters errors during startup, significant enough to prevent
it from loading normally. It populates the `startup_errors` field of the
`awesome` table, then loads the default config (usually from
`/etc/xdg/awesome/rc.lua`) and tells the user what the errors were.
2. AwesomeWM starts up normally, but encounters errors during runtime because of
issues with the loaded config. It populates a notification to present the error
to the user.
3. AwesomeWM does not encounter any errors - `awesome.startup_errors` remains
empty and `debug::error` does not trigger. Happy days!
Take one more look at the `awesome.connect_signal()` function. This is an
example of rc.lua calling a function by indexing a table field, but it has one
more party trick. The first argument passed to this function is just a string;
but the second argument is another function, defined on-the-fly while calling
`connect_signal()`!
It's not necessary to understand exactly what is going on there - we would need
to dissect the `awesome.connect_signal()` function to find out, and there's
really no need to customise AwesomeWM's error notification logic. I just wanted
to highlight this beautiful feature of the language.

##Variable definitions
```
-- {{{ Variable definitions
-- Themes define colours, icons, font and wallpapers.
beautiful.init(gears.filesystem.get_themes_dir() .. "default/theme.lua")
```
The aptly named `beautiful` library is all about themes. Its `init` method loads
the information from the `theme.lua` file, making the theme definitions
available to any module that has `require("beautiful")` - provided that they
access `beautiful`'s fields after the `beautiful.init()` call.
```
-- This is used later as the default terminal and editor to run.
terminal = "xterm"
editor = os.getenv("EDITOR") or "nano"
editor_cmd = terminal .. " -e " .. editor
```
The preferred terminal is set to xterm by default, for no reason other than it
is almost guaranteed to exist on the system. The config file tries to get the
editor from the environment variable `EDITOR` or, if that doesn't exist, sets it
to nano - a popular choice on many distros.
Finally, we define an editor_cmd which concatenates the terminal, the `-e` flag,
and the editor. In the default case, this command would be `xterm -e nano`.
```
-- Default modkey.
-- Usually, Mod4 is the key with a logo between Control and Alt.
-- If you do not like this or do not have such a key,
-- I suggest you to remap Mod4 to another key using xmodmap or other tools.
-- However, you can use another modifier like Mod1, but it may interact with others.
modkey = "Mod4"
```
The comments really say it all!
```
-- Table of layouts to cover with awful.layout.inc, order matters.
awful.layout.layouts = {
    awful.layout.suit.floating,
    awful.layout.suit.tile,
    awful.layout.suit.tile.left,
    awful.layout.suit.tile.bottom,
    awful.layout.suit.tile.top,
    awful.layout.suit.fair,
    awful.layout.suit.fair.horizontal,
    awful.layout.suit.spiral,
    awful.layout.suit.spiral.dwindle,
    awful.layout.suit.max,
    awful.layout.suit.max.fullscreen,
    awful.layout.suit.magnifier,
    awful.layout.suit.corner.nw,
    -- awful.layout.suit.corner.ne,
    -- awful.layout.suit.corner.sw,
    -- awful.layout.suit.corner.se,
}
-- }}}
```
Now we're getting close to the meat and potatoes of rc.lua. Remember the start
of the file, where we called `require()` a bunch of times to pull in various
modules? One of those was 'awful', and here we're accessing some information
from the module. `awful.layout.suit` contains references to various layouts,
some of which can be further specified (such as the four shapes of 'tile'
defined here).
Note the first line of this section after the comment. The open curly brace
indicates that we're defining a table - the elements of the table are references
to the layouts. The order in which these layouts are added to the table defines
their order in Awesome.
The easiest way to understand how each layout works is by simply spawning a few
terminal windows (hit `super` + `enter` five times) and then cycling through the
layouts. The default hotkey is `super` + `space` (add `shift` to cycle
backwards), or you can click the layout icon in the top right corner
(right-click to cycle backwards).
You can comment out any layouts that you don't intend on using, possibly after
using Awesome for a while and finding out which you prefer.

##Menu
```
-- {{{ Menu
-- Create a launcher widget and a main menu
myawesomemenu = {
   { "hotkeys", function() hotkeys_popup.show_help(nil, awful.screen.focused()) end },
   { "manual", terminal .. " -e man awesome" },
   { "edit config", editor_cmd .. " " .. awesome.conffile },
   { "restart", awesome.restart },
   { "quit", function() awesome.quit() end },
}

mymainmenu = awful.menu({ items = { { "awesome", myawesomemenu, beautiful.awesome_icon },
                                    { "open terminal", terminal }
                                  }
                        })

mylauncher = awful.widget.launcher({ image = beautiful.awesome_icon,
                                     menu = mymainmenu })
```
The comment is the wrong way around - first we create a submenu, then a main
menu, and finally the launcher widget.
The submenu is a table of tables assigned to the 'myawesomemenu' name.
-- Menubar configuration
menubar.utils.terminal = terminal -- Set the terminal for applications that require it
-- }}}
```

