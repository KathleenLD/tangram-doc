Tangram currently has four built-in _draw styles_: `polygons`, `lines`, `points`, and `text`. Each draw style displays data in a different way, and some of them require specific data types and properties.

Draw styles are referenced in two places in the scene file: when defining custom [[styles]] and again in [[draw]] groups.

## draw styles

#### `polygons`
The `polygons` draw style tessellates and extrudes vector shapes into 3D geometry. It requires polygonal data.

#### `lines`
The `lines` draw style can turn either polygonal or line data into lines.

#### `points`
The `points` draw style draws a square at a point, and can fill the point with either a customizable dot or a `sprite`. It can work with point data, lines, or polygons.

#### `text`
The `text` draw style draws a rectangle at a point, and paints it with a texture it generates automatically based on its input datasource. It can work with point, line, or polygon data.

These _draw styles_ are used as the foundation for all custom styling in Tangram. When defining a style, they are explicitly referenced under the style name with the `base` parameter:

```yaml
styles:
    buildings:
        base: polygons
```

And when writing an inline-style under a `layer`, they are referenced in _draw groups_:

```yaml
roads:
    draw:
        polygons:
            color: blur
        lines:
            color: red
```

In this way, the same data can be drawn in multiple styles simultaneously.

## polygons
The *polygons* draw style requires a datasource containing coordinates connected by lines into a "closed" shape. If the lines of the polygon start and stop at different places, it is an "open" shape, and the `polygons` draw style can't use it. But if a sequence of lines connects back onto its own starting point, it is considered "closed", and can be extruded into a 3D shape.

#### `polygons` parameters
Styles which are extensions of the `polygons` draw style can take the following parameters:

- `texcoords`
- `animated`
- `blend`
- `materials` : see [[materials]]
- `shaders`: see [[shaders]]

#### `polygons` shader specs
The `polygons` style allows [[shaders]] to be written which take advantage of certain unique attributes and uniforms.

## lines
The *lines* style requires a datasource containing connected coordinates. Thus it can accept either linear or polygonal input data. It draws a rectangle along each line segment, and can optionally draw special `joins` and `caps`.

#### `lines` parameters
Styles which are extensions of the `lines` draw style can take the following parameters:

- `texcoords`
- `animated`
- `blend`
- `materials` : see [[materials]]
- `shaders`: see [[shaders]]

#### `lines` shader specs
The `lines` draw style allows [[shaders]] to be written which take advantage of certain unique attributes and uniforms.

## points
The `points` draw style is used to draw dots or sprites at points of interest. It also builds a rectangle at a point, and can be colored in a variety of ways:

- with a special shader designed to draw a circle
- with a `texture`
- with a `sprite` from a `texture`

If the point is used to draw a dot, the size and color of this circle can be specified in the scene file with the `size` and `color` parameters.

Points styles have access to a variety of special uniforms and parameters.

## text
The `text` style is similar to the `sprites` style, in that it builds a rectangle at a point. However, instead of being colored with a custom texture, this style builds its own texture, containing text.

The content of the text is based on a parameter specified in the scene file, which can be useful for debugging.

The style of the text is also specified in the scene file.

#### `text` parameters
Styles which are extensions of the `text` style can take the following special parameters:

- `text_source` - Defaults to "name", accepts other parameter name or function.
- `font` - Sets font's typeface, style, size, color, and outline.