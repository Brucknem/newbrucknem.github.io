---
layout: post
title:  "Getting data from OpenDRIVE HD maps"
date:   2021-02-18 19:55:00 +0100
categories: Studies
tags: coding guided-research autonomous-driving
pin: false
image: 
    src: /posts/2021-02-16-opendrive-parsing/highway_exit_south.png
    alt: A part of the the highway A9 near Munich in a HD map.
    caption: A part of the the highway A9 near Munich in a HD map.
opendrive_url: https://www.asam.net/index.php?eID=dumpFile&t=f&f=3495&token=56b15ffd9dfe23ad8f759523c806fc1f1a90a0e8#
math: true
post_cdn: /blog/posts/2021-02-16-opendrive-parsing/
---

During my [Guided Research]({% post_url 2021-02-12-guided-research %}) I was given some HD maps of some part of the [highway A9 near Munich, Germany](https://www.google.de/maps/place/A9,+Garching+bei+M%C3%BCnchen/@48.2395578,11.632844,4154m/data=!3m1!1e3!4m5!3m4!1s0x47a11d231dc12da1:0xd702677ee0331ed4!8m2!3d48.2385588!4d11.637908).
I use these maps to get the world positions of landmarks like delineators, road signs, trees... These landmarks are then matched with some video footage of cameras mounted on gantry bridges.  

My final goal is to minimize the reprojection error from the landmarks and the corresponding video pixels to find the camera world pose (the extrinsic parameters).

# OpenDRIVE parser & query implementation

I am currently implementing a parser for these maps, which are in the [OpenDRIVE]({{ page.opendrive_url }}) format. This is an XML format which is pretty complex and thus powerful. It's the industry standard to describe road networks.

## Converting XML to C++

I was struggling really hard with the implementation of the parser, and it took me around a week to find out how to implement it. The good thing is that [ASAM](https://www.asam.net/standards/detail/opendrive/) provides a free XML schema definition (XSD) which describes how OpenDRIVE files have to be structured. The hard part was to get the definition into code so that I can process the data.

The first thing I thought of was to use an XML to C++ compiler which should generate C++ data bindings from the XSD files. I quickly found the [Code Synthesis](https://www.codesynthesis.com/products/xsd/) data binding compiler, but I couldn't get it to work at all for the OpenDRIVE V1.6 standard. I thus tried to implement the standard by hand, which did turn out to be not feasibly at all, so I again searched for ready made solutions or how to use the compiler. 

Eventually, after around one week of despair, I finally found out that our maps conform to the OpenDRIVE **V1.4** standard. I then gave the data binding compiler another try and _voila_, after another day of fiddling around, it works as I expect it to do and produces code that can parse the HD maps. I can now represent the data as a nice object model. _What a relief!_

## Geometric primitives define road segments

OpenDRIVE defines [roads]({{ page.opendrive_url}}_roads) as a collection of five different [geometric primitives]({{ page.opendrive_url}}_geometry): [lines]({{ page.opendrive_url}}_straight_line), [spirals]({{ page.opendrive_url}}__spirals), [arcs]({{ page.opendrive_url}}_arc), [cubic polynoms]({{ page.opendrive_url}}_cubic_polynom) (which are deprecated) and [parametric cubic curves]({{ page.opendrive_url}}_parametric_cubic_curve). These primitives are stacked together along a [reference line]({{ page.opendrive_url}}_reference_line) and thus define the roads shape in the [$$(x, y)$$-plane]({{ page.opendrive_url}}_coordinate_systems). The plane itself is the earth [projected]({{ page.opendrive_url }}_georeferencing_in_opendrive) onto a flat surface. You can imaging this like unrolling the globe into a flat map. With the [parser](#converting-xml-to-c) from the previous section retrieving these primitives is straightforward. 

The landmarks I'm interested in are defined as [objects]({{ page.opendrive_url}}_objects) and [signals]({{ page.opendrive_url}}_signals) in the OpenDRIVE standard. The position of these landmarks is given as the $$s$$-coordinate along the reference line. To calculate the world position of the landmarks in the $$(x, y)$$-plane I have to interpolate along the geometric primitives.

![Line interpolation]({{ page.post_cdn}}line.png ){: width="75%"}
_A line going from $$(-47, 0.7)^T$$ to $$(-0.7, 35.5)^T$$._

For a line the interpolation is easy. Just go from the start some amount into the heading direction.


![Parametric cubic curve interpolation]({{ page.post_cdn}}paramPoly3.png ){: width="75%"}
_A parametric cubic curve going from $$(3.1, 1.4)^T$$ to $$(4.7, 4.8)^T$$._

For a parametric cubic curve the interpolation is a bit trickier. The resulting coordinate along the curve can be calculated with the given [parameters and formulae]({{ page.opendrive_url}}_parametric_cubic_curve):

$$
u(p) = aU + bU*p + cU*p^2 + dU*p^3
$$

$$
v(p) = aV + bV*p + cV*p^2 + dV*p^3
$$


## Roads are composed by geometric primitives

Roads often change their shape as they have to adjust to the local surface conditions and terrain properties. To model these changes OpenDRIVE composes roads by stacking geometric primitives. This gives the flexibility to model [arbitrary road shapes]({{ page.opendrive_url }}_generating_arbitrary_road_courses_from_geometry_elements).

![Stacking geometric primitves]({{ page.post_cdn}}stacking_primitives.png ){: width="100%"}
_Roads are defined by stacking the geometric primitives. The primitives also define the curvature and thus directly the angle of the steering wheel._


## Landmarks along the road with $$s$$ and $$t$$ coordinates

The OpenDRIVE standard defines the locations of [objects]({{ page.opendrive_url}}_objects) and [signals]({{ page.opendrive_url}}_signals) relative to the reference line. The $$s$$-coordinate defines how far the object is from the start of the road. The $$t$$-coordinate defines how far the object is offset to the left or right of the reference line.

Together with the roads world position, the geometries, the geometric primitives and the $$s$$- and $$t$$-coordinates I am now able to calculate the world position of the landmarks I am interested in (in the $$(x, y)$$-plane). Here are some google links with the coordinates that show a [tree](https://www.google.de/maps/place/48%C2%B010'33.8%22N+11%C2%B035'27.7%22E/@48.1760579,11.5888264,17z/data=!4m5!3m4!1s0x0:0x0!8m2!3d48.1760579!4d11.5910151), a [delineator](https://www.google.de/maps/place/48%C2%B010'30.5%22N+11%C2%B035'27.5%22E/@48.1751013,11.5908173,65m/data=!3m1!1e3!4m5!3m4!1s0x0:0x0!8m2!3d48.1751242!4d11.59098) and a [streetlamp](https://www.google.de/maps/place/48%C2%B010'32.7%22N+11%C2%B035'34.4%22E/@48.1757256,11.5927528,65m/data=!3m1!1e3!4m5!3m4!1s0x0:0x0!8m2!3d48.1757516!4d11.5928758). Be aware that the calculated positions from the HD map do not exactly align with the Google Maps positions as Google takes images from satellites in [low earth orbit](https://aerospace.csis.org/aerospace101/popular-orbits-101/#:~:text=The%20majority%20of%20satellites%20orbiting,full%20orbit%20around%20the%20Earth.), thus from a high of 160 km to 2000 km above ground. It's a wonder that Google can provide the google maps service at all with images taken from that far, so please be patient with any inaccuracy.

# Ongoing work

As the calculation of the world positions is now finished in the $$(x, y)$$-plane I will now focus on the height of the objects. These can be calculated using the different road shape attributes, i.e. the [elevation]({{ page.opendrive_url}}_methods_of_elevation).

# Implementation 

At my [OpenDRIVE](https://github.com/Brucknem/OpenDRIVE) github repository you can have a look at the implementation.

# Example images

The images each show a part of the highway A9 near Munich. In the images taken from the map you see the roads in green and the relevant landmarks in violet. There is also one delineator highlighted in red.

## Complete camera view

![Map part of the highway A9]({{ page.post_cdn }}map_view.png){: width="100%"}
_A part of the the highway A9 near Munich in the HD map._


![Map part of the highway A9]({{ page.post_cdn }}camera_view.png){: width="100%"}
_More or less the same part of the the highway A9 near Munich viewed from a camera mounted on a gantry bridge._

## Zoomed towards the highway exit

![Map part of the highway A9]({{ page.post_cdn }}map_exit.png){: width="100%"}
_The same part of the the highway A9 near Munich, zoomed towards the highway exit, in the HD map._


![Map part of the highway A9]({{ page.post_cdn }}camera_exit.png){: width="100%"}
_The same part of the the highway A9 near Munich, zoomed towards the highway exit, from a camera mounted on a gantry bridge._


# References 

- The images for the posts are hosted on my other [Github](https://github.com/Brucknem/CDN/tree/main/blog/posts/2021-02-16-opendrive-parsing). There are also the links to the original sources given.
