## The default rc.lua
Understanding your default config is a great first step towards Awesome
competence. I will be going over each section in turn, not only explaining the
*end result* of each line but also very explicitly stating *what is happening*
to produce that end result.

Thanks to the self-explanatory nature of the default rc.lua, it's not difficult
to understand the purpose and the end result of the following section:
```
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
```
Just by reading the lines `"hotkeys", function() hotkeys_popup.show_help...`
and `"manual", terminal .. " -e man awesome"...` we can understand that here
we're defining some sort of menu. This becomes especially obvious when we click
the Awesome menu icon in the top right and see the same options in the first
submenu there. However, this doesn't mean that we can now go in and edit the
menu to our liking. What if you want to expand the main menu, rather than the
submenu? Where do you go? How do you understand the syntax error you get when
you forget to deliminate table entries correctly? *What even is a table!?*

We won't be diving into the bowels of the Lua API for this guide - we're
limiting ourselves to only the rc.lua for now - but we will be looking very
closely at what exactly is being defined or called, and how.

Let's get started!
