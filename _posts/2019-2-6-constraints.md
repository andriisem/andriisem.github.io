---
title: "Constraints"
date: 2019-2-6 20:05:33 -0000
categories: blog
tags:
  - Constraints
  - Verlet
toc: true
---
The essence of constraint is projection.

**Find the minimum movement that satisfies the constraint.**

## Basic Distance Constraint

The most basic constraint is the distance constraint

<!-- Load the Paper.js library -->
<script type="text/javascript">
  document.getElementsByClassName('toc')[0].style.display = 'none';
</script>
<script type="text/javascript" src="../../assets/js/paper-full.min.js"></script>
<script type="text/paperscript" src="../../assets/js/Constraints/SimpleDistance.js" canvas="distance1"></script>
<canvas id="distance1" width="350" height="350"></canvas>
~~~ javascript
function ConstrainDistance(point, anchor, distance) {
  return ((point - anchor).normalize() * distance) + anchor;
}
~~~


It is satisfied by _projecting_ the point onto a circle around the anchor.

## Distance Constraint Chain

As with all constraints, distance constraints can be chained together

<script type="text/paperscript" src="../../assets/js/Constraints/Chain.js" canvas="distance2"></script>
<canvas id="distance2" width="350" height="350"></canvas>
<div class="togglebox">
  <input id="toggle1" type="checkbox" name="toggle" />
  <label for="toggle1">Show Code</label>
  <section id="content1" markdown="1" >
~~~ javascript
//Set the first link's position to be at the mouse
rope.segments[0] = mousePos;
for (i = 1; i < segments.length; i++) {
  //Pull the next segment to the previous one
  rope.segments[i] = ConstrainDistance(
    rope.segments[i], rope.segments[i-1], distance
  );
}
~~~
<a href="https://github.com/zalo/zalo.github.io/blob/master/assets/js/Constraints/Chain.js"><small>Full Source</small></a>
  </section>
</div>

The order in which constraints are satisfied is important.  The ones here are solved stepping away from the mouse, which pulls them _towards_ the mouse.

## FABRIK Chain

If the distance constraints are first solved in one direction, and then the other, it creates a form of Inverse Kinematics called "FABRIK", or "Forwards and Backwards Reaching Inverse Kinematics".

<script type="text/paperscript" src="../../assets/js/Constraints/FABRIK.js" canvas="distance3"></script>
<canvas id="distance3" width="350" height="350"></canvas>
<div class="togglebox">
  <input id="toggle2" type="checkbox" name="toggle" />
  <label for="toggle2">Show Code</label>
  <section id="content2" markdown="1" >
~~~ javascript
//Set the first link's position to be at the mouse
rope.segments[0] = mousePos;
for (i = 1; i < segments.length; i++) {
  //Pull the current segment to the previous one
  rope.segments[i] = ConstrainDistance(
    rope.segments[i], rope.segments[i-1], distance
  );
}

//Set the base link's position to be at the ball
rope.segments[segments.length - 1] = ball;
for (i = segments.length - 1; i > 0; i--) {
  //Pull the previous segment to the current one
  rope.segments[i-1] = ConstrainDistance(
    rope.segments[i-1], rope.segments[i], distance
  );
}
~~~
<a href="https://github.com/zalo/zalo.github.io/blob/master/assets/js/Constraints/FABRIK.js"><small>Full Source</small></a>
  </section>
</div>

## Collision Constraint

Distance Constraints can also be used to separate

<script type="text/paperscript" src="../../assets/js/Constraints/Collision.js" canvas="distance4"></script>
<canvas id="distance4" width="350" height="350"></canvas>
<div class="togglebox">
  <input id="toggle3" type="checkbox" name="toggle" />
  <label for="toggle3">Show Code</label>
  <section id="content3" markdown="1" >
~~~ javascript
//Separate the balls from the mouse
float cRadius = mRadius + bRadius;
for(i = 0; i < balls.length; i++){
  //If the mouse is closer than some distance
  if((mousePos-balls[i]).magnitude < cRadius){
    //Push the ball away from the mouse
    balls[i] = ConstrainDistance(balls[i], mousePos, cRadius);
  }
}

//Separate the balls from each other
for(i = 0; i < balls.length; i++){
  for(j = i; j < balls.length; j++){
    //If the balls are closer than 2x their radius
    var curDisplacement = balls[j].position - balls[i].position;
    if (curDisplacement.magnitude < bRadius*2) {
      //Move each ball half of the distance away from the other
      var temp = ConstrainDistance(balls[i], balls[j], bRadius);
      balls[j] = ConstrainDistance(balls[j], balls[i], bRadius);
      balls[i] = temp;
    }
  }
}
~~~
<a href="https://github.com/zalo/zalo.github.io/blob/master/assets/js/Constraints/Collision.js"><small>Full Source</small></a>
  </section>
</div>

## Collision Constraints with Verlet

If the constraints act symmetrically (where each participant steps half-way towards satisfying the constraint), then one can simulate physics by adding momentum with Verlet Integration.

<script type="text/paperscript" src="../../assets/js/Constraints/VerletCollision.js" canvas="distance5">

</script>
<canvas id="distance5" width="350" height="350"></canvas>
<div class="togglebox">
  <input id="toggle4" type="checkbox" name="toggle" />
  <label for="toggle4">Show Code</label>
  <section id="content4" markdown="1" >
~~~ javascript
for(i = 0; i < balls.length; i++){
  //-*Use Verlet Integration to add inertia*-
  var curPosition = balls[i];
  balls[i] += balls[i] - prevBalls[i];
  prevBalls[i] = curPosition;

  //Exert gravity by translating downwards one pixel each frame
  balls[i] += new Point(0, 1);
}

for(iterations = 0; iterations < 5; iterations++){
  //The previous example's code here!
  //It must be iterated to fully resolve all collisions
}
~~~
<a href="https://github.com/zalo/zalo.github.io/blob/master/assets/js/Constraints/VerletCollision.js"><small>Full Source</small></a>
  </section>
</div>

## Verlet Rope

Solving constraints sequentially is called the _Gauss-Seidel Method_. It converges faster, but it is not technically correct.

<script type="text/paperscript" src="../../assets/js/Constraints/RedRope.js" canvas="redRope"></script>
<canvas id="redRope" width="350" height="350"></canvas>
<a href="https://github.com/zalo/zalo.github.io/blob/master/assets/js/Constraints/RedRope.js"><small>Full Source</small></a>

## Volume Preserving Soft Body

The alternative is to average the contributions from each constraint before applying them.  This is the _Jacobi Method_.  It is more stable, and makes it so the order does not matter.  However, it is "squishier" because it converges more slowly.

If one wraps the rope above into a circle, and constrains the shape's volume, one can create a volume preserving soft-body

<script type="text/paperscript" src="../../assets/js/Constraints/VolumeBlob.js" canvas="softBody"></script>
<canvas id="softBody" width="350" height="350"></canvas>
<a href="https://github.com/zalo/zalo.github.io/blob/master/assets/js/Constraints/VolumeBlob.js"><small>Full Source</small></a>

The Jacobi Method is useful for keeping phantom forces from appearing in complex systems like this one.

This light introduction to constraints is the first in (hopefully) a series of blog posts exploring the power of simple mathematics.