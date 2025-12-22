# Attachments

## Introduction

When you design 3D parts then it is not to hard to make one of the _primitive_ shapes: a cube, a sphere, a cylinder, etc. that he OpenSCAD language has as built-in _modules_. You can then use _operators_ to position these shapes relative to each other and other operators to subtract, intersect, or add shapes. One of the great advantages of OpenSCAD is that it is actually incredibly easy to get started. Few forget the thrill of their first 3D model in OpenSCAD after copy and pasting a few lines.

However, life is surprisingly complex and most interesting 3D _models_ are are non-trivial. Their complexity gets multiplied if you want them to have soft edges and when you also want to properly parameterize them their complexity easily gets squared.

The standard computer science solution to complexity is _modularity_. Modularity encapsulates local details and provides a way to handle them as a whole. The goal is to make it easier to combine these modules in many different situations without having to worry about their local details. 

In this tutorial we call these modules _components_. A component is an OpenSCAD _module_ that is designed to be easy to assemble in _composites_, where a composite consists of an _assembly_ of other components or composites and is in itself a component. This is a recursive model.

To make components work it is crucial that:

* modules can hide their inner details, and 
* be oblivious about what components uses them and how they are used. 

A component can specify _parameters_ since it is an OpenSCAD module; a parent component should be able to fully assemble the component through these parameters. However, the parent component must also set the _context_ for the child component before it calls its the OpenSCAD module. 

Clearly the 3D transformation (the position, scale, and rotation) is the primary aspect of the context. However, in OpenSCAD we need to standardize quite a bit of variables in the context because the language has no _state_ nor can a module return any value when called. That is, no information can flow from child to parent. Zero, nada. 

For example, if you want to place several components next to each other you need to know their _bounding box_. However, there is _no_ way to get this information from the component itself. Not being able to _adapt_ to the shape of a child component would make any component model infeasable.

BOSL2 found a (partial) solution by specifying the intent and then delegating the caclulation of the transformation matrix to a point where the child component's geometry is known. Using this geometry and the geometry of the parent you can position a component for example with the LEFT sides aligned since you know exactly how wide the parent and the child are. 

If you think this sounds complex then you're on the right way to understanding it because it is complex, made even more complex because it has to work around some of OpenSCAD's severe limitations. In BOSL2 it is called the attachment model.

However complex this solution is, it is relatively easy to use once you get familiar with the patterns. In this tutorial we will start at the absolute basics and progress to making 3D models out of advanced components.

## Anchors

Invoking `%cube(10)` renders a cube of 10mm on all sides, the percent sign makes it transparent:
```
include <BOSL2/std.scad>
%cube(10);
```
<img width="417" height="338" alt="image" src="https://github.com/user-attachments/assets/680c8ab7-3cb2-46e6-bda0-9b93111cc4d9" />

The first thing noticeable is that the cube is not _centered_ around the _origin_ but that the origin is aligned with the LEFT BOTTOM FRONT corner. In virtually all cases life is easier when shapes are centered around the origin because it makes the shape symmetric and that tends to simplify the operations because there are no preferential directions, all axis are treated evenly. The OpenSCAD designers must have realized this because they added a `center=true|false` attribute.
```
include <BOSL2/std.scad>
%cube(10,center=true);
```
<img width="443" height="371" alt="image" src="https://github.com/user-attachments/assets/d476f9ab-30d8-41f1-8211-0878a11a041e" />

This is a bit of a kludge and one of the reasons BOSL2 added a `cuboid()` module that is centered by default. However, what if we'd like to align the origin with the LEFT BOTTOM FRONT corner? 

Every shape consists of _faces_, _edges_ where faces meet and _corners_ where the edges meet. A simple cube has six faces (think of the faces of a dice), 12 edges and 8 corners. All in all 26 positions. Trying to position over this many parts with booleans, as in `center=true`, would become prohibitively cumbersome.

The solution to be able to navigate this space was _anchors_. An anchor is a _named_ position on a shape and all BOSL2 shapes, which includes all the OpenSCAD primitive shapes, have anchors for the common positions like LEFT, RIGHT, FRONT, BOTTOM, TOP, etc. The reason I use upper cased names is that they are actually constants for these names in BOSL2.

Each BOSL2 has an `anchor` parameter that can be set to the name of the anchor we want the origin on. For example, if we want the cube to be positioned on the center of the BOTTOM face we can specify this as follows
```
include <BOSL2/std.scad>
%cube(10, anchor=BOTTOM);
```
<img width="442" height="373" alt="image" src="https://github.com/user-attachments/assets/9946ad7a-203f-4347-9c4e-31cbfc3500ac" />

Interestingly, we can combine the anchor 'names' to specify edges (where faces meet):

```
include <BOSL2/std.scad>
%cube(10, anchor=BOTTOM+LEFT);
```
<img width="508" height="373" alt="image" src="https://github.com/user-attachments/assets/12c88293-1923-4c07-a262-15bc7f13c1f4" />

And I guess it is obvious that we can also specify a corner (where edges meet).
```
include <BOSL2/std.scad>
%cube(10, anchor=BOTTOM+LEFT+FRONT);
```
<img width="449" height="349" alt="image" src="https://github.com/user-attachments/assets/0079e4c3-c23a-4abc-830a-197312f34bee" />




## Attach

Since we have anchors it follows that we should be able to position components relative to each other using these anchors. The _attach()_ operator takes care of that.
```
include <BOSL2/std.scad>
%cuboid(10)
attach(TOP,BOTTOM) cube(5)
;
```
<img width="356" height="334" alt="image" src="https://github.com/user-attachments/assets/05d77e31-61f7-488b-87d8-5154ae90278f" />








Clearly the cube is a wonderful and useful shape but a tad limited in its utility. When you want to get some useful work done you likely have to combine different shapes and _transform_ them to the right _position_ and maybe adjust their _rotation_ and _scale_. For example, if you want a smaller cube on top of another cube you could use the _translate_ operator:
```
cube(10);
translate([2.5,2.5,10]) cube(5);
```
<img width="256" height="256" alt="image" src="https://github.com/user-attachments/assets/176f67b2-a44b-40e2-a667-ff2b8ac8c729" />

Depending on your experience level your reaction will vary from puzzlement to horror. A novice will have to think deep where on earth the 2.5 came from and why is the last parameter 10? Why isn't it symmetric. An experienced software expert will shudder when he realises how deep the violation of the law of DNRY (Do Not Repeat Yourself) is. This might work for trival examples but there is a special place in hell for developers that force you to look at such designs.

So how can we do better? 

In the previous example we have 2 independent _statements_. The first statement is the large cube and the second has the translate operator that sets the current transformation for the smaller cube. However, in OpenSCAD a module can also take a child module. We could also remove the first semicolon (';') and concatenate the modules.
```
cube(10)
translate([-2.5,-2.5,5]) cube(5);
```
At first sight, we did not seem to made 


The first observation is that the box is kind of awkwardly positioned with the LEFT+BOTTOM corner on the _origin_, position [0,0,0]. This is _asymmetric_. Asymmetric usually spells trouble in software because it means you have to think of many cases instead of one. In general, designing is easier when you center around the origin. The OpenSCAD designers also realized this afterwards because we can actually center the cube so the the CENTER of the cube is at the origin. 
```
cube(10, center=true);
translate([0,0,7.5]) cube(5, center=true);
```
<img width="256" height="256" alt="image" src="https://github.com/user-attachments/assets/21c2f7c1-c781-4e31-b25e-fa2fcb5a0c3f" />

Centering the origin is so obvious that in BOSL they made it the standard and added alternatives for the primitive shapes so you do not have to add the `center=true` attribute. For cube, this is `cuboid()`.



At least we've got rid of the funny 2.5 but we're still stuck with the 7.5. Clearly this is functional since we place the small cube on top of the little cube so an offset of the sum of half both cubes at least makes some logical sense.

One of the golden rules in software is to think _relative_. 

In OpenSCAD the transformations are also _modules_. A module is the OpenSCAD name for a procedure with one or more _child_ modules. For example, the `translation()` module will change the current transformation matrix 

To build more complex _composites_ you need to position these primitives 

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


## Diffing

        filter( tag!= keep && tag != remove) children();
To produce its output, diff() forms the union of all the base objects and then subtracts all the objects 
with tags in remove. Finally it adds in objects listed in keep. Attachable objects should be tagged using tag() and non-attachable objects with force_tag().
```
diff()

union() {
    difference() {
        filter( tag!= keep && tag != remove) children();
        filter( tag == remove) children();
    }
    filter( tag == keep) children();
}
```

## Intersect

 This module treats the children in three groups: objects matching the intersect tags, objects matching the tags listed in keep and the remaining objects that don't match any listed tags. The intersection is computed between the union of the intersect tagged objects and the union of the objects that don't match any listed tags. Finally the objects listed in keep are union ed with the result.
```
union() {
    intersection() {
        filter( tag == intersect ) children();
        filter( tag != intersect && tag != keep ) children();
    }
    filter( tag == keep ) children();
}

```

