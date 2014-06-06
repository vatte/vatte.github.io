---
layout: post
title:  "Littlebits Synth Kit + Arduino = MIDI"
date:   2014-06-05 14:13:00
categories: projects
---

I've had the [Korg/Littlebits Synth Kit](http://littlebits.cc/kits/synth-kit/) lying around since last christmas, but it hasn't seen as much use as it should, because it doesn't really offer connectivity except for the output audio jack.

The goal of the project was to add MIDI connectivity to it, so that I could use it with a full sized keyboard and any other MIDI devices and software. Currently I've implemented note on, note off and pitch bend to control the oscillators with CV from an analog-digital converter(ADC) and trigger the envelope and sequencer with a digital signal.

<iframe src="//player.vimeo.com/video/97521647" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> 

I set out to buy an ADC (MCP4821) and a JST SH 3-wire connector to connect the Littlebits Wire module to an Arduino + MIDI shield. Getting hold of the connector in Helsinki proved impossible, so I had to flatten some jumper cable to fit in the tiny connector. (Meanwhile, I ordered the real connectors from [Sparkfun](https://www.sparkfun.com/products/10358).)

After making an experiment by controlling the voltage and reading the frequency in Max, I concluded that the CV protocol of the oscillator is of the V/Octave variant, unlike Hz/V used in some of the other Korg synths, such as MS-20 (Mini). 

<img src="{{ site.url }}/img/littlebits-cv.png">

With the tune knob on the oscillator bit, the pitch range could be adjusted accurately so that each semitone is represented by 1000/12 or 83 and 1/3 milliVolts. My master keyboard begins at C2, or MIDI note 36, so I adjusted the pitch of the oscillator so that the first audible tone matched to that.

I found out that the trigger on the envelope bit was activated by a rising edge, or a signal that connects from ground to a higher voltage. Surprisingly, connecting the signal connector of the trigger inlet of the envelope bit to one of Arduino's digital pins and writing LOW and HIGH to it did not work. I solved the issue by adding a 15KOhm pull-down resistor from that pin to ground.

Finally, the [Arduino code](http://github.com/vatte/littlebits-synth/) was simple enough, thanks to [Kerry Wong's MCP4821 example](http://www.kerrywong.com/2012/07/25/code-for-mcp4821-mcp4822/) and the excellent [Arduino MIDI library](http://arduinomidilib.sourceforge.net/index.html).



<div class="highlight"><pre><span class="cp">#include &lt;MIDI.h&gt;</span>
<span class="cp">#include &lt;SPI.h&gt;</span>
 
<span class="cp">#define PIN_CS 10</span>
<span class="cp">#define GAIN_1 0x1</span>
<span class="cp">#define GAIN_2 0x0</span>
<span class="cp">#define DAC_MAX 4095</span>

<span class="cp">#define TRIG_PIN 3</span>
<span class="cp">#define SEMITONE 83.333 </span><span class="c1">//millivolts per semitone</span>
<span class="cp">#define PITCH_BEND 833.33</span>
<span class="cp">#define ZERONOTE 33 </span><span class="c1">//zero volts midi note</span>
<span class="cp">#define MAXNOTE 82</span>

<span class="n">byte</span> <span class="n">lastNoteOn</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
<span class="kt">int</span> <span class="n">lastPitchBend</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>

<span class="kt">void</span> <span class="nf">setup</span><span class="p">()</span>
<span class="p">{</span>
  <span class="n">pinMode</span><span class="p">(</span><span class="n">PIN_CS</span><span class="p">,</span> <span class="n">OUTPUT</span><span class="p">);</span>
  <span class="n">SPI</span><span class="p">.</span><span class="n">begin</span><span class="p">();</span>  
  <span class="n">SPI</span><span class="p">.</span><span class="n">setClockDivider</span><span class="p">(</span><span class="n">SPI_CLOCK_DIV2</span><span class="p">);</span>
  
  <span class="n">MIDI</span><span class="p">.</span><span class="n">begin</span><span class="p">(</span><span class="n">MIDI_CHANNEL_OMNI</span><span class="p">);</span>
  <span class="n">MIDI</span><span class="p">.</span><span class="n">setHandleNoteOn</span><span class="p">(</span><span class="n">HandleNoteOn</span><span class="p">);</span>
  <span class="n">MIDI</span><span class="p">.</span><span class="n">setHandlePitchBend</span><span class="p">(</span><span class="n">HandlePitchBend</span><span class="p">);</span>
<span class="p">}</span>
 
<span class="c1">//assuming single channel, gain=2</span>
<span class="kt">void</span> <span class="nf">setOutput</span><span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">val</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">byte</span> <span class="n">lowByte</span> <span class="o">=</span> <span class="n">val</span> <span class="o">&amp;</span> <span class="mh">0xff</span><span class="p">;</span>
  <span class="n">byte</span> <span class="n">highByte</span> <span class="o">=</span> <span class="p">((</span><span class="n">val</span> <span class="o">&gt;&gt;</span> <span class="mi">8</span><span class="p">)</span> <span class="o">&amp;</span> <span class="mh">0xff</span><span class="p">)</span> <span class="o">|</span> <span class="mh">0x10</span><span class="p">;</span>
   
  <span class="n">PORTB</span> <span class="o">&amp;=</span> <span class="mh">0xfb</span><span class="p">;</span>
  <span class="n">SPI</span><span class="p">.</span><span class="n">transfer</span><span class="p">(</span><span class="n">highByte</span><span class="p">);</span>
  <span class="n">SPI</span><span class="p">.</span><span class="n">transfer</span><span class="p">(</span><span class="n">lowByte</span><span class="p">);</span>
  <span class="n">PORTB</span> <span class="o">|=</span> <span class="mh">0x4</span><span class="p">;</span>
<span class="p">}</span>

<span class="kt">void</span> <span class="nf">loop</span><span class="p">()</span> <span class="p">{</span>
  <span class="n">MIDI</span><span class="p">.</span><span class="n">read</span><span class="p">();</span>
<span class="p">}</span>

<span class="kt">void</span> <span class="nf">HandleNoteOn</span><span class="p">(</span><span class="n">byte</span> <span class="n">channel</span><span class="p">,</span> <span class="n">byte</span> <span class="n">pitch</span><span class="p">,</span> <span class="n">byte</span> <span class="n">velocity</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if</span><span class="p">(</span><span class="n">pitch</span> <span class="o">&gt;=</span> <span class="n">ZERONOTE</span> <span class="o">&amp;&amp;</span> <span class="n">pitch</span> <span class="o">&lt;=</span> <span class="n">MAXNOTE</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if</span><span class="p">(</span><span class="n">velocity</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="n">lastNoteOn</span> <span class="o">=</span> <span class="n">pitch</span><span class="p">;</span>
      <span class="kt">int</span> <span class="n">freq</span> <span class="o">=</span> <span class="n">round</span><span class="p">(</span><span class="n">SEMITONE</span><span class="o">*</span><span class="p">(</span><span class="n">lastNoteOn</span><span class="o">-</span><span class="n">ZERONOTE</span><span class="p">)</span> <span class="o">+</span> <span class="n">PITCH_BEND</span><span class="o">*</span><span class="p">(</span><span class="n">lastPitchBend</span><span class="o">/</span><span class="mf">8192.0</span><span class="p">));</span>
      <span class="n">freq</span> <span class="o">=</span> <span class="n">max</span><span class="p">(</span><span class="n">min</span><span class="p">(</span><span class="n">freq</span><span class="p">,</span> <span class="n">DAC_MAX</span><span class="p">),</span> <span class="mi">0</span><span class="p">);</span>
      <span class="n">setOutput</span><span class="p">(</span><span class="n">freq</span><span class="p">);</span>
      <span class="n">digitalWrite</span><span class="p">(</span><span class="n">TRIG_PIN</span><span class="p">,</span> <span class="n">HIGH</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="k">else</span> <span class="k">if</span><span class="p">(</span><span class="n">pitch</span> <span class="o">==</span> <span class="n">lastNoteOn</span><span class="p">)</span> <span class="p">{</span>
      <span class="n">digitalWrite</span><span class="p">(</span><span class="n">TRIG_PIN</span><span class="p">,</span> <span class="n">LOW</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="kt">void</span> <span class="nf">HandlePitchBend</span><span class="p">(</span><span class="n">byte</span> <span class="n">channel</span><span class="p">,</span> <span class="kt">int</span> <span class="n">bend</span><span class="p">)</span> <span class="p">{</span>
  <span class="n">lastPitchBend</span> <span class="o">=</span> <span class="n">bend</span><span class="p">;</span>
  <span class="kt">int</span> <span class="n">freq</span> <span class="o">=</span> <span class="n">round</span><span class="p">(</span><span class="n">SEMITONE</span><span class="o">*</span><span class="p">(</span><span class="n">lastNoteOn</span><span class="o">-</span><span class="n">ZERONOTE</span><span class="p">)</span> <span class="o">+</span> <span class="n">PITCH_BEND</span><span class="o">*</span><span class="p">(</span><span class="n">lastPitchBend</span><span class="o">/</span><span class="mf">8192.0</span><span class="p">));</span>
  <span class="n">freq</span> <span class="o">=</span> <span class="n">max</span><span class="p">(</span><span class="n">min</span><span class="p">(</span><span class="n">freq</span><span class="p">,</span> <span class="n">DAC_MAX</span><span class="p">),</span> <span class="mi">0</span><span class="p">);</span>
  <span class="n">setOutput</span><span class="p">(</span><span class="n">freq</span><span class="p">);</span>
<span class="p">}</span>
</pre></div>
