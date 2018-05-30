
Or maybe start by just saying all three methods upfront?





Here's an exercise: calculate the distance between these two sets of points:

...

You may remember similar problems from your primary (?) school education. The solution was to use Pythagorean theorem:

...


Here's a visulization of what we just calculated:

...

Let's visualize this in three dimensions:

...


Oh no, this doesn't seem right! Earlier we thought these distances we're the same, but when visualized on a globe, it's clear they're different. What's going on here?

## Flat-surface distance

Pythagorean theorem finds the relation between two right angle points on a **flat-surface distance**. The theorem assumes that 1 unit in any direction is equivalent no matter where the point is located. 

This is not the case for EPSG 84. Moving east one degree near the North Pole is not equivalent distance to moving east at the equator.

<https://gis.stackexchange.com/a/183131>

So how do we calculate distances on a three dimensional surface?

While the earth isn't a perfect sphere, it's close enough that we can treat it as a sphere for many geodesic purposes.

## Spherical distance

## Ellipsoidal distance

## Takeaways

When working with geospatial libraries, make sure you understand _how_ they're calculating distance.

## Note

All distances above are in terms of degrees.
