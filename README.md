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

## theAppThatLetsUsProcessToomersCornerBeingRolled

The core of our project is
[theAppThatLetsUsProcessToomersCornerBeingRolled][0],
which uses the Haskell functional programming language to watch the
Toomer's Corner webcam and assess whether it is currently being rolled.

[0]: https://github.com/IsToomersCornerBeingRolledRightNow/theAppThatLetsUsProcessToomersCornerBeingRolled

We used the [friday](http://hackage.haskell.org/package/friday) and
[friday-devil](http://hackage.haskell.org/package/friday-devil)
Haskell packages to do our image processing. Both of these depend on the
C library [DevIL (Developer's Image Library)](http://openil.sourceforge.net/).

Our script first grabs stills from the live feed which are
fed into our Haskell code. We use the
[Canny edge detector](https://en.wikipedia.org/wiki/Canny_edge_detector)
to get a stripped-down image with just the sharp edges.
(WARNING: a mathematician approaches!)
We think of the edge-detected image as a (discretized) real-valued
function of two real variables. The
[Sobel operator](https://en.wikipedia.org/wiki/Sobel_operator)
on this edge-detected image gives use a (discretized) gradient vector field.
We then compute the direction of each gradient vector and count up the number
of more or less horizontal (within about 10 degrees of true horizontal) gradients.
The number of horizontal gradients is the number we use to determine whether
Toomer's Corner has been rolled. Our reasoning for using this number
is that strips of TP hanging down from trees and lamp posts
(and everything really) will give us a large number of horizontal gradients because there
are a large number of vertical edges. At least this seems to be the case - our testing 
generated no false positives. However, we'll need to wait until the next big ballgame for
more rigorous field testing.

## theTinyAppThatTweetsAPhotoWhenToomersIsRolled

Once we've determined that Toomer's is being rolled, we've got to
get the word out. (After all, how ELSE could people know?)
That's where [theTinyAppThatTweetsAPhotoWhenToomersIsRolled][2]
comes in.

[2]: https://github.com/IsToomersCornerBeingRolledRightNow/theTinyAppThatTweetsAPhotoWhenToomersIsRolled

Rather than rolling our own server to persist the good news, we cut to the
chase by using Twitter. The Twitter API has several restrictions
(which we'll describe in more detail in the next section); however,
since our detection app will request a tweet very infrequently, we won't
hit any issues here.

Using the [Twitter RubyGem library](https://github.com/sferik/twitter),
we may easily authenticate with the Twitter API 1.1, and post a message
to [@IsToomersRolled](https://twitter.com/IsToomersRolled)
affirming the rolling, as well as the video still which triggered our
program.

## aWebAppThatCanCheckTwitterForYouToSeeIfToomersIsBeingRolled

We decided we should also create
[aWebAppThatCanCheckTwitterForYouToSeeIfToomersIsBeingRolled][1]
in the vein of similar projects such as <https://isitchristmas.com/>.
It's hosted by [Heroku](https://heroku.com)
at <https://istoomerscornerbeingrolled.herokuapp.com/>.

[1]: https://github.com/IsToomersCornerBeingRolledRightNow/aWebAppThatCanCheckTwitterForYouToSeeIfToomersIsRolled

Since we're using Twitter as our persistence layer, we need to interface
with the Twitter API again to detect what our latest tweet was. If we
can do that, the rest is easy: if the last tweet was within 12 hours,
we assume the rolling is still relevant, so we display the tweeted (twatted?)
message. Otherwise, we just display "No..." and move on.

PROBLEM! The new Twitter API (1.1) is very stringent about how many
requests it will accept from an application before timing out: about one
request per minute! It also requires authentication, even though all our
tweets are public. Therefore our original plan to develop a Single Page App
to read the feed with pure Javascript didn't fly.

We returned to Ruby to solve this problem, opting for the lightweight
[Sinatra](http://www.sinatrarb.com/) platform rather than a more heavyweight
solution like Rails. Using the Twitter RubyGem again, we can authenticate
and pull our latest tweet on the backend, and use a simple ERB template
to display the appropriate response.

The hardest puzzle here was how to circumvent Twitter's rate limit, without
adopting a heavyweight/redundant database in our application.
Enter the flat-file persentence pattern: on each request, we check to see
if `.twitter_cache` was written within the last 75 seconds. If not,
then we make a new request to the Twitter API, which we write to our
cache file for use within the last 75 seconds. ActiveRecord, eat your heart out.

Since the Twitter API sucks, we provided our own API for the
cornucopia of applications which could use this valuable data:
<https://istoomerscornerbeingrolled.herokuapp.com/api/>. Just remember
to check that `data["stale"] == False` before displaying an outdated
message!