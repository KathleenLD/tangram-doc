**YAML** is an object-based data format with a flexible structure. Its specification includes a huge array of crazy abilities, but we only use a few of them.

Here are the most important YAML features to know about when writing Tangram scene files.

## object types

YAML's structure is made up of "mappings," known elsewhere as "key/value pairs" – we usually call these _parameters_.

```yaml
parameter: value
```

When nested, YAML calls this a "collection" – we usually call it an _element_:
```yaml
element:
    parameter1: value
    parameter2: value
```

At a given level, key names must be unique.

```yaml
element:
    parameter1: value
    parameter1: value # not allowed, parameter name is repeated
```

A parameter can't also be an element:
```yaml
# THIS WON'T WORK
parameter: value
    subparameter: value
```

In this documentation, we refer to both parameters and elements as "objects".

## object syntax

YAML supports two kinds of syntax when writing nested objects: _whitespace syntax_ and _bracket syntax_.

####whitespace syntax

_Whitespace syntax_ requires each level of an object to be indented with spaces – any number of spaces or tabs is allowed, as long as it's consistent throughout the file. It is relatively easy to read, though it tends to result in longer files.

```yaml
element:
    subelement:
        parameter1: value1
        parameter2: value2
    subelement2:
        parameter1: value1
        parameter2: value2
```

_Note:_ if you mix spaces and tabs, the parser will throw an error like this one:
```
YAMLException {name: "YAMLException", reason: "bad indentation of a mapping entry"}
``` 
#### bracket syntax
Here is the same object as the first example above, written in the more compact _bracket syntax_:
```yaml
element1: { subelement1: { parameter1: value1, parameter2: value2 }, subelement2: { parameter1: value1, parameter2: value2 } }
```

In some cases, this syntax may be harder to read, but it's usually fine for shorter objects, or patterns which are repeated frequently:

```yaml
roads:
    data: { source: local, filter: roads }
```

#### Syntax examples
For further examples, check out our many fine [demos](https://github.com/tangrams?query=demo)!

## lists

Lists are written differently in each of the above syntax styles.

#### whitespace lists
```yaml
element:
    parameter:
        - item 1
        - item 2
        - item 3
```

####bracket lists
```yaml
element: { parameter: [ item1, item2, item3 ] }
```
## syntax mixing

_Whitespace syntax_ can enclose _bracket syntax_, but not the other way around – once you start an object in _bracket syntax_, you have to finish it before you can move back into _whitespace syntax_.

```yaml
element:
    parameter: { [ item1, item2, item3 ] }

element:
    parameter:
        - item1
        - item2: { parameter1: value1, parameter2: value2 }
        - item3

# THIS WILL NOT WORK
element: { parameter:
    - item1
    - item2
    - item3
```

## data types

Tangram's data types are based on YAML's functionality, but we've extended them a bit in certain contexts.

#### duck typing

YAML interprets data types automatically:

```yaml
5 # that's an int
5.2 # that's a float
duck # obviously a string
5.2 ducks # mixed-type => string
[1.0, .5, .75] # array of floats (aka a parade)
```

#### stops

Our "stops" data structure is a way to define a relationship between two ranges of values. It is defined as an array of two-item arrays, like so:

`[[12, 3], [14, 6], [16, 9]]`

The first value in each pair is always a _zoom level_. The second value in each pair is interpreted contextually, with all of the constraints of the particular parameter. At other zoom levels, values will be interpolated linearly.

For instance, in a `width` block, if no units are specified, each pair is interpreted as `[zoom, meters]`. The above example will define a value of 3m at zoom 12, 6m at zoom 14, and 9m at zoom 16. At zoom 13, the value will be 4.5m.

Stops may be used in `color`, `width`, `focal_length`, and `fov`.

```yaml
color: [[10, [0.3, 0.4, 0.3]], [14, [0.5, 0.825, 0.5]]]
width: [[13, 0px], [14, 3px], [16, 5px], [18, 10px]]
```

Note that stops define settings to be used when tile geometry is built. Typically, this only happens when the tile is loaded, at a tile integer change (or sometimes halfway between integer zooms) – so incremental zooming won't cause style changes until the next tile is loaded.

## reserved keywords

Our YAML parser detects certain keywords contextually based on the element or parameter in which they are used.

Additionally, all of the named scene file elements are officially _reserved keywords_ and can't be used as element names.

#### `function`

Strings starting with `function` will be passed to the style builder as JavaScript in certain contexts: `color`, `width`, `order`, `interactive`, `visible`, `filter`, `size`, `fill`, and `stroke`.

```yaml
# Single-line Javascript example:
width: function () { return 2.5 * Math.log(zoom); }
```

#### `$` keywords

There are two keywords with a `$` prefix, available for use in [[filters]].

The `$zoom` keyword may be used to define filters with optional `min` and `max` parameters.

```yaml
outline:
   filter: { $zoom: { min: 15, max: 20 } }
```

The `$geometry` keyword can specify a filter to match a specific geometry type, for cases with a FeatureCollection includes multiple geometry types:

```yaml
labels:
   filter: { $geometry: point }
```

## multiline strings

One of the reasons we chose YAML is its ability to handle multi-line strings with a minimum of fuss. In _whitespace syntax_ only, start an parameter's value with a "pipe" character (`|`) followed by a newline, and everything that isn't indented _less_ after that will be treated as a single string value, newlines included:

```yaml
element:
    parameter: |
        This is my multi-line string.
        There are many like it,
        But this one is mine.

        It even has an empty line!
        Sublime.

          Indents don't matter
            As long as they don't
                start at the parent's level
                        or less.
```

This lets us put code straight into an attribute value, and it still looks like code:

```yaml
# Multi-line Javascript example:
style:
    color: [0.5, 0.5, 0.5]
    width: |
        function () {
          return 2.5 * Math.log(zoom);
        }

# GLSL Example:
elevator:
    extends: polygons
    animated: true
    shaders:
        blocks:
            position: |
                // Elevator buildings
                if (position.z > 0.01) {
                    position.z *= (sin(position.z + u_time) + 1.0);
                }
```
## Comments

```yaml
# This is a YAML comment.
# There is no way to write block comments in YAML, sorry.

# To quote http://stackoverflow.com/questions/2276572/how-do-you-do-block-comment-in-yaml:

# If you want to write
# a block-commented Haiku
# you'll need three pound signs
```

Bear in mind that in multi-line strings, pound signs lose their ability to comment code! You'll have to use the comment convention in whatever language you're writing. (This can be a pain when using auto-commenting in text editors.)

## `url`

`url` attributes may be used to link to external YAML files in the `styles` element, as well as in the shader block elements `globals`, `color`, and `position`. This allows for more modular shader construction as well as easy sharing of styles.

#### linked `styles`

```yaml
dots:
    url: styles/dots.yaml
```

When the `url` parameter is used in the `styles` block, the linked `.yaml` file should include a replica of the entire style element as it would appear in the scene file, including the _style name_:

```yaml
# styles/dots.yaml
dots:
    extends: polygons
    shaders:
        uniforms:
            ...
```

#### linked shader `blocks`

```yaml
shaders:
    blocks:
        globals:
            url: functions/hsv2rgb.glsl
```

When the `url` attribute is used in place of any `blocks` parameters, the linked `.glsl` file should only include GLSL code, as it would appear in the parameter's value:

```glsl
// hsv2rgb.glsl
vec3 hsv2rgb(vec3 c) {
    vec4 K = vec4(1.0, 2.0 / 3.0, 1.0 / 3.0, 3.0);
    vec3 p = abs(fract(c.xxx + K.xyz) * 6.0 - K.www);
    return c.z * mix(K.xxx, clamp(p - K.xxx, 0.0, 1.0), c.y);
}
```

## JSON compatibility

YAML's capabilities are officially a superset of JSON, which makes conversion between the two formats a cinch.