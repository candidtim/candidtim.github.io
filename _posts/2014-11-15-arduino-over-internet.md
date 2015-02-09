---
layout: post
title:  "Arduino robot controlled over the Internet - initial thoughts"
date:   2014-11-15
categories: arduino
---
Building this proof-of-concept raised, at once, few important decisions to make.

As the robot has to be controlled over the Internet - it needs to be connected to the Internet, but also an operator
needs to be able connect to it in the end, wherever the robot is located (yet supposing there is an way to connect to
the Internet at this place - Wi-Fi or 3g/4g).

First solution to come with - how to connect a robot to the Internet. I would like to keep "Viper" as simple and as
cheap as possible (to be able to reproduce it multiple times and probably not only by me). And all I'm starting with is
an Arduino Uno :)

I've considered several options, among which where buying Arduino shields for Wi-Fi, 3g, etc, checking out Arduino Yun,
or RaspberryPI instead of Arduino, etc. etc. I've realized however that I'll need to buy a lot of spare parts in either
case, but in the end I want: an internet connection for my robot, a screen, connect a number of sensors, etc. There are
certainly many valid solutions here, but I've chosen the one I could try at once with all the components I had at my
hand: Arduino Uno board and... an Android phone.

Android phone seems to have everything I need even in the long run: internet? check! (both wi-fi and 4g!); screen?
check!; video camera and a microphone? check!; some sensors? check! (gps! magentic field sensor, ambient temperature
sensor, and many more - phones today are really packed with stuff).

Well, the phone probably costs a lot by itself compared to buying spare parts for a robot, but - I already have it. If
I would be able to come with an easy kind of a "plug and play" solution - I would be able to put a phone into the robot
to make it alive, and take it away when I need a phone but not the robot. Plus, OK, in reality, for now I have 2 phones
and I don't use one of them...

This solution however raises several issues. First, I will need to implement an Android application and I never did it
before (OK, to be honest, I did it once somewhere around 2010...), so I will need to learn programming for Android
platform. This is something interesting and I would like to learn. Second, Android application will need to control the
Arduino platform, as I still would like to benefit from Arduino's analogue IO feature to control the robot and probably
have some sensors on it in the future. This is something that sounds like a challenge, so I would be also interested in
that. Let's see...