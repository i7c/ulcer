Nobody wants an ulcer. This should make you think.

It’s basically a primitive script, that spawns multiple processes (sources)
that can produce values on a bus. The bus is consumed to update a hashmap. The
map can be used to fill in the values in a format string.

It might not be tailored to, but could be used to feed something like
https://github.com/LemonBoy/bar

Feature list
============

v 0.1
-----
* Built-in support for time, memory, temperatures, battery
* Herbstluftwm: show tag list, active tag and urgent tags and current window
  title
* Event accumulation for fast event sequences: ulcer uses chunks of events to
  update the state in case there are multiple events waiting in the queue. This
  ought to increase processing speed.
* Event sources are completely decoupled; each can have its own event rate
* Moderate memory footprint, no process spamming. Ulcer attempts to run as
  little external commands as possible
* Output conversions: transform values when displaying them

v 0.2
----------------
* Process watchdog: Ulcer now surveils the child processes it spawns and
  restarts failed ones. Poorly written event sources can still provide their
  services more or less reliably.
* Graceful shutdown can be initiated via SIGUSR1
* hlwm plugin learned to respect the quit_panel and reload hooks, for which it
  will shutdown the entire ulcer process
* Improved hlwm support: ulcer now sets the initial state for
  "hlwm.tags.active" correctly
* Config file support: both options (plugin list and template string) that were
  specified via command line args so far, can now also be specified via a
  config file.
* "Section" support in the template string: Instead of composing the entire
  template string in one big literal, the user can now specify sections. See the
  example config below for details. In the config file section "sections", the
  user may define arbitrary keys and assign them parts of a template string.
  Those bits can then be embedded in other bits or in the root template string
  like so: `~[key]~`

  "Embed in other bits" means, that ulcer resolves these references
  recursively.  So you can define keys and use other keys in the string.

  Note, that ulcer don't gives much about endless recursions. The user is
  responsible. If you define endless recursive patterns, ulcer will most likely
  produce a high CPU load and do nothing else.

  If the user wants to use the literal string `~[something]~`, they’re doomed at
  the moment.

Usage
=====

Ulcer takes two arguments.

1. The format string
2. Comma-separated list of plugins to use

Note, that in most shells you should wrap the format string in single quotes.

Format string
-------------
The format string is what defines ulcer’s output. You can put whatever you want.
Currently, the "special" ulcer syntax is comprised of this:

* Value placeholders look like this: ${name}

  Ulcer will replace such placeholders with their most recent value from the
  state. If the value is undefined, ulcer replaces it with the empty string.

  Note, that names can be hierarchical. The time plugin for example produces the
  values "pretty" and "seconds" located under the parent "time". Ulcer flattens
  the structure so they become two dot separated names pointing to scalar
  values. Ulcer uses a prefix to flatten structures recursively. Currently, the
  top-level prefix is the empty string, which is why all names start with a .

* Value placeholders with conversions look like this:
  ```
    ${name|`conversion`}
  ```

  Sometimes the source provides values in a different format as you would want
  to display them. In such cases, you can specify a conversion. A conversion is
  specified by adding a pipe followed by the conversion code enclosed in
  backticks. The syntax between backticks is the regular perl syntax. You can
  access the original value through the `$val` variable. It is also writeable
  and you can safely modify it, if you so wish.

  Example: Displaying the free memory in MiB:

  ```
    ${.mem.realfree|`int($val / 1024)`}
  ```

Plugin list
-----------
Every known plugin in the list will be activated. Plugins have their own life
cycle and their own rate at which they produce events. They do not affect other
plugins.

To see a list of known plugins, examine the state (as described below).

Example
-------

ulcer '${.time.pretty}' time

Display the value 'time.pretty' produced by the time plugin.

Examine the state
-----------------
To examine all available values, you can run

ulcer '${.state}' list,of,plugins

The state value is special, as it produces a formatted multi-line dump of the
internal state.
}

Example config for herbstluftwm with lemonbar (`$HOME/.config/ulcer/config`)
---------------------------------------------

```
[ulcer]
plugins=hlwm,time,acpi,mem
template='~[left]~~[right]~'

[sections]
left = '%{l} ~[title]~'
right = '%{r}~[hlwm]~ ~[bat]~ ~[temp]~ ~[mem]~ ~[time]~ '

bat = '${B-}%{F#aafaaf}${.bat.BAT0.capacity}% (${.bat.BAT0.status})%{B-}%{F-}'
hlwm = '%{B#f92672}%{F#000}${.hlwm.tags.urgent|`length($val) > 0 ? " $val " : ""`}%{F-}%{B-} %{B#fd971f}%{F#000} ${.hlwm.tags.active} %{B-}%{F-}'
mem = '%{B-}%{F#f92672}${.mem.realfree|`int($val / 1024)`} MiB%{B-}%{F-}'
temp = '%{B-}%{F#fd971f}${.temp.thermal_zone0.temp}°C ${.temp.thermal_zone1.temp}°C ${.temp.thermal_zone2.temp}°C ${.temp.thermal_zone3.temp}°C%{B-}%{F-}'
time = '%{B-}${.time.pretty}%{B-}%{F-}'
title = '${.hlwm.window.title}'
```
