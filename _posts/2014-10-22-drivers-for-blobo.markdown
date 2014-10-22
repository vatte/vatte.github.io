---
layout: post
title:  "Hacked drivers for Blobo"
date:   2014-10-22 20:35:00
categories: projects
---

This past weekend I needed to use the Blobo again for making an *Interactive Conductors Baton* as a part of our *FractConductor* project at [SibHack](http://sibhack.fi). The project is a virtual conducting experience combined with amazing fractal graphics. I will write more about it later, in the meantime have a look at this teaser we shot during the night at the *Finnish National Gallery, Ateneum*:

<iframe src="//player.vimeo.com/video/109755593" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>


###Show me the drivers already

Opening the serial port in OS X Mavericks was impossible as Bluetooth settings have been severely limited. Whatever I did, the result was "pairing rejected". I ran into the same problem in Ubuntu 14.04 when using the GUI, but thankfully Linux comes with the `rfcomm` command. I made a simple driver for the Blobo in Python, which you can [download from the Github repository](https://github.com/vatte/Blobo).


###What's a Blobo anyway?
Blobo is a Bluetooth game controller made by a Finnish company Ball-IT. It contains 9DOF IMU sensors (accelerometer, gyroscope and magnetometer) as well as a barometric pressure sensor for detecting squeezing. It has been discontinued, but you might find one if you look around. With a built-in Li-Ion battery and a Bluetooth connection it makes for a cool component for a variety of projects. Unfortunately Ball-IT hasn't released the source code for the driver. Thanks to Jussi Virkkala, the serial data structure [is publicly available](http://www.neuroupdate.com/blobo/) making hacking easy.

I was using Blobo's already in the [Stronger Harder Faster Jester](http://vatte.github.io/projects/strongerharderfasterjester/) pitch-following juggling balls. In OS X Lion, which I happened to be running at the time, creating a Bluetooth serial port for the Blobo was a piece of cake, and I just parsed the data inside Max MSP according to Jussi's notes. The above python drivers should work in this case as well, without the extra step of binding the serial port with `rfcomm`.
