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

v 0.2 (upcoming)
----------------
* Process watchdog: Ulcer now surveils the child processes it spawns and
  restarts failed ones. Poorly written event sources can still provide their
  services more or less reliably.

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

Example config for herbstluftwm with lemonbar
---------------------------------------------

```
ulcer '%{l} ${.hlwm.window.title}%{r} %{F#f92672}${.hlwm.tags.urgent}%{F-} %{B#fd971f}%{F#000} ${.hlwm.tags.active} %{B-}%{F-} %{F#aafaaf}${.bat.BAT0.capacity}% (${.bat.BAT0.status}) %{F#fd971f}${.temp.thermal_zone0.temp}°C %{F#f92672}mem:${.mem.realfreeper}% %{F-}${.time.pretty} ' \
    'time,mem,acpi,hlwm' \
    | lemonbar -B '#272822' -F '#f8f8f2' &
```
