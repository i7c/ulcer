Nobody wants an ulcer. This should make you think.

It’s basically a primitive script, that spawns multiple processes (sources)
that can produce values on a bus. The bus is consumed to update a hashmap. The
map can be used to fill in the values in a format string.

It might not be tailored to, but could be used to feed something like
https://github.com/LemonBoy/bar


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


Plugin list
-----------
Every known plugin in the list will be activated. Plugins have their own life
cycle and their own rate at which they produce events. They do not affect other
plugins.

Example
-------

ulcer '${.time.pretty}' time

Display the value 'time.pretty' produced by the time plugin.

Examine the state
-----------------
To examine all available values, you can run

ulcer '${.state}' <list of plugins>

The state value is special, as it produces a formatted multi-line dump of the
internal state.
