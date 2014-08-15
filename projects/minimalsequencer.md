---
layout: page
title: Minimal sequencer
permalink: /projects/minimalsequencer/
timeline: Fall 2013 - Current
category: musicalinterface
description: is a project for creating a minimal hardware sequencer interface that offers the most relevant features.
priority: 1
---

I have been working on a project to create a minimal interface for a natural user interaction with a physical sequencer to create a simple, economical, intuitive and compact device. In this project I try to avoid menu diving, and instead of an LCD display have LED indicators and direct mapping from control (such as potentiometer, button and rotary encoder) to parameter (e.g. pitch, step, duration). 

I'm currently conducting research about the affordances that a sequencer can have, and classifying their need for immediate and secondary availability to the user.

##Mr. Peach
Mr. Peach is the first prototype I have made. It consists of a rotary dial for step selection (12 steps), a slider for pitch selection, LED's for step and pitch indication and one potentiometer for speed/tempo selection. Under the hood there is an atmega 328 running the arduino bootloader, and a digital potentiometer for control voltage output to control analog synthesizers.

<iframe src="//player.vimeo.com/video/103507474" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> 

The prototype works for programming pitch sequences, and is somewhat fun to play with, especially due to the lack of scale-quantization (which should also be possible to have as an option). The slider proved less than ideal for pitch selection, as it doesn't really represent the pitch on each step intuitively, even with the LED indicator - as the slider stays in the same position when the step is changed. In the next prototype I plan to move from potentiometers to either rotary encoders or motorized sliders, and add RGB LED's for more expressive (and beautiful) indicators.