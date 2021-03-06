# 03-quad

## 1. Introduction

Here is a simple example, if we want to pass `color` for all the **fragment**. The only option for passing values to **fragment** is using `varying`. So, we may set attributes to **vertex**, and then share them to **fragment** using varying.

Also, suppose we want add a scale to all the vertex, it is best to be assigned using a `uniform`. A `uniform` applies on all vertices, so, we do not need to assign every vertics a scale value, just use one for all.

Note the `color` attributes and the `v_color` varyings in **vertex**,
```python
vertex = """
uniform float scale
attribute vec2 position;
attribute vec4 color;
varying vec4 v_color;

void main()
{
    gl_Position = vec4(scale*position, 0.0, 1.0);
    v_color = color;
}
"""
```

Now, pass varyings to **fragment**,
```python
fragment = """
varying vec4 v_color;

void main()
{
    gl_FragColor = v_color;
}
"""
```

**Note:** The colors are **interpolation** of the vertex when rendering. This interpolation is made using **distance** of the fragment to each individual vertex.

## 2. Step-by-step

Now, we will also teach you how to using a class to encapsulate all the callbacks in `app.Canvas`. In this example, we will inheritate `app.Canvas` object to a new class used in this demo,

```python
class Canvas(app.Canvas):

    def __init__(self):
        app.Canvas.__init__(self, size=(512, 512), title='scaling quad',
                            keys='interactive')

        # bind a timer
        self.timer = app.Timer('auto', self.on_timer)

        # with 4 vertices
        program = gloo.Program(vert=vertex, frag=fragment, count=4)

        # bind data
        program['a_position'] = [(-1, -1), (-1, 1), (1, -1), (1, 1)]
        program['color'] = [(1, 0, 0, 1),
                            (0, 1, 0, 1),
                            (0, 0, 1, 1),
                            (1, 1, 0, 1)]
        program['scale'] = 1.0
        self.program = program

        # set viewport
        gloo.set_viewport(0, 0, *self.physical_size)

        # initialize timer
        self.clock = 0.0
        self.timer.start()

        # show the canvas
        self.show()
```

The major difference is that we now have a `self.timer` binded to `app.Timer`, the callback function is `self.on_timer`. So, we may define this function in class `Canvas`. The changes of `uniform` will be updated on GPU real-time.

```python
def on_timer(self, event):
    self.clock += 0.01 * np.pi
    self.program['scale'] = 0.5 + 0.5 * np.cos(self.clock)
    self.update()
```

This demo can be run via,
```python
c = Canvas()
app.run()
```

The functions such as `on_draw` and `on_resize` can also be moved into this `Canvas` class. Details can be found in the code.

## 3. Code

[03-quad.py](examples/03-quad.py)

## 4. Exercises

### 4.1 Rotating the quad

The scaling of a quad is simple, a `uniform` variable `scale` is given to all vertices, and the scaling is done ! Think about **rotate** a quad. In this case, you need a parameter, maybe a `theta`, that controls the rotation angles of all points. Note that you have access to the **sin** and **cos** function from within the shader.

This might be mathematically involved at first. However, it can be implemented using `vispy.gloo` with barely any work.

### 4.2 Change the colors dynamically

What aboud change the color dynamically?

### 5. References

This tutorial is based on [Glumpy : The easy way](http://glumpy.readthedocs.org/en/latest/tutorial/easyway.html). `Glumpy` is well documented, so, if you are interested in the low-level `vispy.gloo` functions, please read the document on `Glumpy` first!
