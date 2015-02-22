# Is Toomer Corner Being Rolled Right Now?

A [#SmartCityHack](http://www.global.datafest.net/) project rolled by
[Alex](http://github.com/redxaxder),
[Daniel](http://github.com/friedbrice),
[Steven](http://github.com/StevenClontz),
and [Zack!](http://github.com/ZSarver)

Abstract: using a live feed provided by the City of Auburn,
<http://www.auburnalabama.org/mvc/cams/City-Cameras/Toomer's-Corner>,
we can programmatically determine if Auburn Tigers fans have begun
[rolling Toomer's Corner](http://en.wikipedia.org/wiki/Auburn_University_traditions#Toomer.27s_Corner).
This program tweets at
[@IsToomersRolled](https://twitter.com/IsToomersRolled),
which is used by our web app at
<https://istoomerscornerbeingrolled.herokuapp.com/> to display
either "No..." or "Yes!".

## A Picture Is Worth a 1000 Words

![sketch of our stack](https://istoomerscornerbeingrolledrightnow.github.io/assets/appStackSketch.svg)

## The Haskell

We used the [friday](http://hackage.haskell.org/package/friday) and 
[friday-devil](http://hackage.haskell.org/package/friday-devil) Haskell packages to do 
image processing. Both in turn depend on the C library 
[DevIL (Developer's Image Library)](http://openil.sourceforge.net/).

First we have a script that grabs stills from the live feed. These stills are fed into our
Haskell code. Our Haskell code uses the 
[Canny edge detector](https://en.wikipedia.org/wiki/Canny_edge_detector) to first get a
stripped-down image with just the sharp edges. We think of the edge-detected image as a
(discretized) real-valued function of two real variables. The 
[Sobel operator](https://en.wikipedia.org/wiki/Sobel_operator) on this edge-detected image
gives use a (discretized) gradient vector field. We then compute the direction of each
gradient vector and count up the number of more or less vertical (within about 10 degrees
of true vertical) gradients. The number of vertical gradients is the number we use to
determine whether Toomer's Corner has been rolled. Our reasoning for using this number
is that strips of TP hanging down from trees and lamp posts (and everything really) will
give is a large number of vertical gradients. Indeed this seems to be the case, as in our
testing we generate no false positives.