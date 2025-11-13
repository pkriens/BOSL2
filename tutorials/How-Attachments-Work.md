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

## Coordinates

All _shapes_ are rendered in a _coordinate system_ that assigns a unique infinitely small _point_ 
with 3 numbers: x, y, and z. The x moves to the right and when negative to the left, the y moves
backward (+) and forward (-), and the z up and down. Points are specified with a list of 
an x, an y and a z, like `[x,y,z]`. For convenience you can use list.x, list.y, and list.z to acccess
position list[0], list[1] and list[2]. By default, the meaning of the numbers are _millimeters_, about
1/24th of an inch. Numbers are extremely accurate floating points so you can go as small or as big as 
you like.

Point [0,0,0] is the _origin_ of the coordinate system. In general, when we create a shape,
it will have its _center_ at the origin. For example, when we create a `cuboid()`, it will
center around the origin, with as much of its body on each side.
```
include <BOSL2/std.scad>
cuboid(10);
```
<img width="255" height="256" alt="image" src="https://github.com/user-attachments/assets/f15ada3e-a7dd-4a15-866e-6e0c22af6a13" />

We can move the cube around with the `translate(...)` module but BOSL2 adds modules for 
easier to use modules that move in one direction.

* `x` – xmove, left, right
* `y` – ymove, fwd, back
* `z` – zmove, up, down

These functions _transform_ the origin for the subsequent module. For example, move the cube 5 up, 
which will make it rest on the x-y plane because 5 is half of the size.z of the cube.
```
include <BOSL2/std.scad>
up(5) cuboid(10);
```
<img width="256" height="256" alt="image" src="https://github.com/user-attachments/assets/94991ea6-257f-4e4e-9550-f27c9edffc6b" />

Each of the 6 sides of the cube has a name: TOP, DOWN, LEFT, RIGHT, FWD, BACK. In the picture you can see 
the x (red), y (green), and z (blue) axises. When you press the Y- this matches the real world
persective. However, normally you navigate a lot and at any moment in time the real world 
perspective does not match the screen which is utterly confusing. So let's make a cube 
that marks its sides with a letter: T for TOP, D for DOWN, L for LEFT, R for Right, F for FWD, and B for BACK.

To put a letter on a side we can transform the coordinate system so that the origin is 
exactly in the middle of the side. For the RIGHT side, we need to go left half de cube's
side, for the TOP, we need to go up and DOWN we go down since the current origin 
is at the center of the cube. However, when we place the character on the side we
find that the character is not aligned with the side. We need to rotate the coordinate
system to ensure that the character lies flat on the side and in the right orientation.
These transformations are at the heart of this tutorial but are best explained 
once we have the `idcube()` so explanations will come later. So just copy the code
and assume it will become clearer later.
```
module idcube(size=10) {
    act_size = is_list(size) ? size : [size,size,size];
    lh=min(size)/2+1;
    dz = 0.001;        
    module label(translate, rotate, char) {
        translate(translate) 
        rotate(rotate)
        translate([-lh/2,-lh/2,-dz/2])
        color("blue")
        linear_extrude(dz) 
        text(char, size=lh)
        ;
    }
    cube(size,center=true);
    label([act_size.x/2,0,0],[90,0,90], "L");
    label([-act_size.x/2,0,0],[90,0,-90],"R");
    label([0,act_size.y/2,0],[90,0,180],"B");
    label([0, -act_size.y/2,0],[90,0,0],"F");
    label([0,0,act_size.z/2],[0,0,0],"T");
    label([0,0,-act_size.z/2],[0,0,180],"D");
}
idcube();
```
<img width="256" height="254" alt="image" src="https://github.com/user-attachments/assets/5bc1824c-0ac1-4032-9922-bf16f716418a" />

## Transformations

The `text(...)` module creates a 2D polygon on the x-y plane with z=0. 
This matches the TOP and DOWN sides perfectly but on the other sides
the letter will stick out. For the RIGHT side we need to rotate the 
letter 90 degrees over the x-axis to make it stand up right and then 
90 degrees over the z-axis to align it with the RIGHT side.

<img width="379" height="253" alt="image" src="https://github.com/user-attachments/assets/2c37a2be-c45d-4b58-9832-08d69d89da3f" />

These rotations tend too make your head spin, they are very unpredictable, at least for me. A large part 
of the power of OpenSCAD is that can just try out until it looks good. The scale and skew 
transformations are much less confusing.

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




