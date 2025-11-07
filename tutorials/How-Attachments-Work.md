# Attachments

## Introduction
Attachments in BOSL2 provide positioning relative to other parent objects. For example, you can position a cube on a cube.

<img width="256" height="256" alt="image" src="https://github.com/user-attachments/assets/0d34e604-41c1-4774-b9d1-21d02c0f1f63" />

```
include <BOSL2/std.scad>
cuboid(100) attach(TOP) cuboid(10);
```
Attachments are really easy to use until they're not. The reason is that repeated 3D transformations 
are surprisingly tricky to follow in our heads. This tutorial is a thorough behind the curtains tour
to make you understand what is going on. It starts at the beginning so you do not need any prior
knowledge of openscad and nor BOSL2.

## Execution Model

At first sight OpenSCAD looks like a C like script language:

```
a = 10;
translate( [a,0,0] ) cube( a );
```

However, the execution model is kind of unique. For each module, and a file is a module, OpenSCAD will 
first execute all assignments before it executes the _module invocations_. It has some special rules
for variables but the interesting part with respect to BOSL2 is the module invocation. When OpenSCAD 
executes the previous program it will call `translate(...)` as the first invocation. Translate will
modify the _transformation matrix_ and will then call its _children_, `cube(...)` in this case. When it
returns 

The transformation matrix defines how the _shapes_ (cube,sphere, etc.) are rendered. It can translate
a shape to a new _position_, which is a point in 3D [x,y,z] space. It can also scale shapes, rotate them,
and skew them. Therefore if you call `scale(10)` then scale will work similarly to `translate(...)`. It
will set the transformation matrix to scale the given amount and then call the children.




