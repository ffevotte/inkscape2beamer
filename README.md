# `inkscape2beamer`

This small command-line utility takes an SVG file (as produced by Inkscape) as
an input to generate LaTeX Beamer animations. Each layer in the SVG image is
considered to be a frame in the animation. Each layer's name can encode a
specification determining how the animation proceeds.

By default (i.e. if all layer names are "normal" strings), the produced
animation has as many steps as there are layers in the SVG image: at each step,
a new layer is overlaid on top of previous ones.

## Usage

```sh
$ inkscape2beamer myanimation.svg
Collecting layers...
  Id           Z  Steps  Name
  --------------------------------------
  layer1      10  1-     background
  layer2      10  2-     first_step
  layer3      1   3-4    second_step
  layer4      10  4-     another_step
  layer5      0   5-     unused

Generating PDF frames:
  myanimation_second_step.pdf...    done
  myanimation_background.pdf...     done
  myanimation_first_step.pdf...     done
  myanimation_another_step.pdf...   done
Generating LaTeX files:
  myanimation.tex...                done
  myanimation-test.tex...           done
```

A "test" LaTeX file is produced to help you check that the animation conforms to
your expectations in a standalone pdf document.

```sh
$ latexmk -pdf myanimation-test.tex
[...]

$ xdg-open myanimation-test.pdf
```

Once you're satisfied with the result, directly embed `myanimation.tex` within
your real Beamer presentation.


## Animation specification in layer names

Each layer's name should be composed of 1 to 3 parts, separated by pipe ("`|`") characters.

- Valid forms are:
  - "`steps|z|name`" (full specification)
  - "`step|name`" (missing `z` component)
  - "`name`" (frame name only)

- The meaning of these three parts is as follows:
  - `steps` specifies the steps during which the layer is shown. It should be
    composed of 1 to 2 parts, separated by a colon ("`:`").
    - Valid forms include:
      - "`begin:end`" (full specification)
      - "`begin:`" (empty end step)
      - "`begin`" (missing end step)
      - empty or missing entirely: in that case, the default steps specification is "`+1:`"
    - the meaning of these parts is as follows
      - `begin` and `end` respectively specify the frame number when the frame
        starts and stops being displayed
      - `begin` can be an absolute step number or a relative one. In the latter
        case, it starts with `+` or `-` and is interpreted relative to the beginning
        step of the previous frame.
      - `end` can be an absolute step number of a relative one. In the latter case,
        it starts with `+` or `-` and is interpreted relative to the beginning step
        of the current frame.
      - an empty end step (when there is nothing after the "`:`") denotes a
        frame that remains on display untile the end of the animation.
      - a missing end step (when there is no "`:`" at all) is replaced by "`+0`" :
        this denotes a frame displayed during only one step.
    - For example: "`+1:`" (the default steps specification if none is provided)
      means that the current frame should start 1 step after the previous one,
      and remain on display until the end of the animation

  - `z` is a z-buffer order: layers with lower `z` values are displayed in the
    background
    - if empty or missing, the default value is `10`.
    - as a special case, a value of `0` is interpreted to mean that the layer
      should not be exported and displayed at all in the animation.

  - `name` is the frame name, used as part of the file name when exporting
    frames as pdf images
