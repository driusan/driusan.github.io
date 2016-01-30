---
layout: default
title: Notes on installing Plan 9
---
I recently decided to give Plan 9 another shot. It's probably been
10 to 15 years since I last tried it, and figured "Go can cross-compile
to it and it doesn't have systemd, so it might be fun."

I don't think anyone will be surprised if I say Plan 9 has a learning
curve if you're used to UNIX, but these are the notes of problems I
had (and resolutions) after about a half day of playing around and trying
to create a new user. May they be useful for someone else trying to get 
started with it.

## Installation:

- When I first installed from the ISO, I got a kernel panic at boot,
  but booting as a live CD worked as expected. Documentation tells you
  not to use VirtualBox but to try a different VM tech. If this isn't
  an option, I had luck using a QCOW instead of VDI for the disk format.
- The default is to assume a 640x480x8bit VGA display driver. Since this
  is 2016, if you want to go higher resolution, use vesa instead of VGA
  (and use a 32 bit display, not an 8 bit.)

## Getting started:

- when it asks for a user on boot, type "glenda". Anything else will drive
  you insane and won't have permission to do anything. Glenda is the 
  default user, and will open a couple windows with (barely) enough
  help to get you started with rio (the windowing system)
- First read the rio help that's open by default to understand the
  windowing system. (up and down arrows will scroll up/down by
  about a half page at a time.) By the time you get to the end,
  you should be able to open a new terminal window or resize a window,
  at least
- Then read the acme help that's open by default in another window 
  to understand Plan 9's ACME editor. "vi" is not an editor, but a command
  completely unrelated to the UNIX "vi", so you'll have to learn to use
  acme to edit any files. (If you're a hardcore unix user and can get by 
  with ed, "ed" does what you expect.)
- Eventually, when googling for how to get started, you'll find the plan9
  wiki telling you to "read 9.ps, names.ps, acme/acme.ps, auth.ps, 
  fossil.ps, and plumb.ps (in roughly that order)". These mostly explain 
  the philosophy and design aspects of Plan9, not how to use it. So even 
  if you manage to read them, they won't tell you, for instance, how 
  view a postscript file on Plan 9.
  You can open them with the "page" command (the plan9 terminals are
  implicitly paged, so there's no need for less or more, but "page" is a 
  pager that understands more formats than plain text.. such as .ps)

## Creating another user:

- /adm/users seems to be the equivalent of /etc/password combined 
  with /etc/group. "man 6 users" to figure out format of /adm/users
- None of the documentation I could find on how to create a new user
  seemed to work. In the end, adding a line to /adm/users and adding
  that user to group "sys" (by adding it to the end of the sys line, not
  by adding sys to the end of the user's line) seemed to work.
- Documentation about creating users tells you to run /sys/lib/newuser.
  That gives an error because /usr/$username doesn't exist. If you manually
  create it, you can run the script. (From what I can tell /usr/ is like a
  hybrid of /home and /usr in Unix. There is no /home.)
- auth/changeuser gives an error about permission denied on
  /mnt/keys/$username. The obvious solution (change the permissions)
  doesn't work, and I haven't figured it out yet.

So now I'm in a state where I have a Plan9 system up and running, no 
network, my own user, but no password, and no way to change it.

In conclusion, I don't think this will be the year of Plan 9 on the
desktop.

