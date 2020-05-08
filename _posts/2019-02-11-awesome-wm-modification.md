---
layout: post
author: Grant Nichol
color:  green
title:  "Awesome WM Modifications"
date:   2019-02-11 20:50:00 -0600
categories: ["Notes"]
---

I'm currently using Awesome as my window manager. One of the main reasons I switched to it from i3
was the built in capacity for modification. I'm finding that anything I want to tweak, I can.

Two of the main modifications I've made so far are:
   * Sane icon scaling in built in notifications
   * Ability to rename tags in place

\*Note I've adapted code from Stack Overflow questions to fit how I like, most of the work is not my own

## Icon Scaling

Turns out as of `v4.3` there is a `beautiful.notification_icon_size` so I could use that instead.

With `v4.2` and earlier I am able to use `naughty.config.notify_callback` to look for notifications with
icons and set the desired size. My final function in `rc.lua` is:

```lua
naughty.config.notify_callback = function(args)
	if args.icon then
		args.icon_size = 50
	end	
	return args
end
```
One issue I have found is that if a notification tries to use a built in icon that doesn't exist, it
still reserves space for an icon of size 50. I have not yet figured out a way to get around that.

## Tag Renaming

This is just a simple function that calls `awful.prompt.run` and asks the user for a new name.
The benefit of this method is that I can rename tags as I leave windows places.

```lua
awful.key({ modkey, "Shift",  }, "r",
   function ()
      awful.prompt.run {
		   prompt       = "rename current tag: ",
         text         = awful.tag.selected().name,
		   textbox      = awful.screen.focused().mypromptbox.widget,
         exe_callback = function (s) awful.tag.selected().name = s end,
      }
   end,
		{description = "rename tag", group = "awesome"}),
```



