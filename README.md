thunner
=======

A curses Google Play Music client

'thunner' - thunder (Doric Scots)

Requirements
------------

* Python 2
* Simon Weber's Unofficial Google Music API
* mplayer
* Some songs on Google Play Music
* One or more internets

Keys
----

Arrow keys : menu navigation
enter : play
. : next
, : previous
spacebar : pause 
q : quit

Configuration
-------------

Configuration of thunner is via `~/.thunnerrc`. An example file is as follows:

	      email <your e-mail address here>
	      pass <your password here>
	      color mycolor blue default
	      assign mycolor header-text

`email` and `pass` are required (yes, plaintext password is not exactly secure). Spaces are allowed in `pass`. Any number of `color` and `assign` lines are allowed. 

`color` takes three arguements: a name (which the user chooses), a foreground colour and a background colour. Foreground and background colours can be of the following values:

* `default`
* `black`
* `red`
* `green`
* `yellow`
* `blue`
* `magenta`
* `cyan`
* `white`

`default` picks up the foreground/background colours of the terminal. I believe `black` to `white` are colours 0-7 as per your X resources (e.g. `*color0: #575757`).

`assign` takes two arguements a named colour pair (as defined via `color`) and a destination. The following destinations are defined:

* `thunner` - the colour of "thunner" (top left)
* `header-text` - colour of text in the header (location in menu tree)
* `header-sep` - colour of separators between header text
* `header-border` - colour of border below header
* `footer-border` - colour of border above footer
* `status` - colour of status (playing/paused/stopped)
* `currently-playing` - colour of currently playing item (in menu) [NOT SUPPORTED YET]
* `current-item` - colour of highlighted row in menu
* `text` - other rows in menu

Where the above are not defined in `~/.thunnerrc` they default to `default` which in turn is defined as:

      color default default default

For consitency's sake, in the code I've spelt "colour" as "color".

TO DO
-----

* Add a random mode
* Check compatibility with other OSes
* Add more customisations
* Add RGB definition to configuration file for terminals that support `init_color`
* Add support for pass
* Hide lag from API calls

Known Issues
------------

* API calls can be slow causing lag between input and playback. Avoid switching songs too quickly (with 'next', 'previous', etc.)