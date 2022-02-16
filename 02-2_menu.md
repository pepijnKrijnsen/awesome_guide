## Menu
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
The submenu is a table of tables assigned to the 'myawesomemenu' name. Each
inner table contains two fields: a string representing the name of the menu
item, and a command or function to execute when that option is clicked.  
The variable `mymainmenu` is then instantiated to contain the return value of a
call to `awful.menu()`. What are we passing to `awful.menu`? It looks
complicated, but in the end we're passing a single table which has a single
element: another table called 'items', which has as ITS elements the menu that
we created previously and an option to open the terminal.  
On the last line, the variable `mylauncher` is instantiated to contain the
return value of the `awful.widget.launcher()` function. All we're passing to
this function is the icon we want the launcher to have, and the menu that
should be opened when the launcher is clicked. (Adding the launcher to the bar
at the top of the screen is taken care of at a later stage.)
