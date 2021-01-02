---
layout: post
title: Making and interfacing a Wifi RGB LED matrix (neo-pixel)
subtitle: 
bigimg: /img/cables.jpg
tags: [ESP32, wifi, LED matrix, neopixel, python, ascii art, personal]
---

# The magic of Neo-pixels
After writing this title, I realized there is a post from [Adafruit](https://learn.adafruit.com/adafruit-neopixel-uberguide?view=all) with the same title. And that is for a good reason in my opinion! 

Neo-pixels are not just a strip of LEDs in series! They are individually addressable through a single data wire and this makes them very special in terms of what you can do with them! Each one of the LEDs has an integrated circuit (called **WS2812**).  I really suggest [this part](https://learn.adafruit.com/adafruit-neopixel-uberguide?view=all#writing-your-own-library-16-13) again from adafruit if you are interested! 

Bottom line is that there are 3 bytes of data for each RGB LED, each one representing how much of each R/G/B is going to be in each LED. 1 byte is an integer from 1 to 255 and this is your resolution for each color as well. So the awesome thing is the whole strip operates like a huge shift register!

The way that the system works is very simple: after you have fed a new value to the data wire, it is staged for refreshing in the first LED (closer to the micro-controller). When you feed a second set of 24bits, it goes to the second LED in the chain and so on. Finally, by setting the data wire "low" for more than 50 microseconds the colors are refreshed for the whole strip. 


# Construction/inspiration
I was mostly inspired (and educated about the project) by [Great Scott](https://www.youtube.com/watch?v=D_QBlFIQk-o) that did the same with an arduino. I used directly an ESP32 because I wanted to be able to refresh the screen wirelessly. I also have a single arduino for experimentation and I wanted this to be a permanent installation in my room.

## Components
* The LED strip (ordered from Ali-express)
* a strong power supply (4 amps). The LEDs draw quite some current when they are fully lit.
* Beach wood for support
* a thick while acrylic sheet (I found that on the street actually - that was nice since it's not cheap!)
* carton (that is to separate the LEDs so they have a "pixel-like" definition when lit. 
* ESP32 to control the LEDs and interface with Wifi.

### Tips:
you might have to put some supply in parallel - the LEDs have quite some resistance and LEDs closer to the end of long chains may end up with lower current and inaccurate/faded colors. Maybe have a look at [this video](https://youtu.be/Ew0HmLy_Td8?t=417) to better understand what I'm talking about.

In my [github repo]( https://github.com/mylonasc/esp32wirelessledmatrix) you can find a version of the code I used plus a python script I made to control it.

The way the remote control works, is that I send REST requests containing serially encoded in the text the values for all the LEDs. If you think what I should have done to do this first it will be easier to read my code. In any case, drop me an email at mylonas.charilaos@gmail.com in case you are interested in adapting it - I'd be happy to help!

Here are some fotos of the construction. I had to clamp the board to sth so I can saw it:
![construction 1 ](/img/construction1.jpg)
![construction 2 ](/img/construction2.jpg)

The matrix after I have split the strip, glued it to the wood and before I soldered all the connections for the power and data.
![construction 3 ](/img/construction3.jpg)

Constructing the frame:
![construction 3 ](/img/construction4.jpg)

A video of the matrix running a default script with an arduino before I constructed the carton grid to give it the pixel look:

<video width="720" height="480" controls="controls">
  <source src="/img/video_led_matrix.mp4" type="video/mp4">
</video>



Finally, I made a short python script that translates ASCII art to color data for the matrix! Here is an example:
![a smiley face](/img/smiley.jpg)

In the github code upload you will also find the ASCII codes for that.

H.






[//]: # " # Variational Autoencoders"
[//]: # "Variational techniques in statistics have been around for some time. Relatively recently"

[//]: # # "Speech Recognition"
[//]: # "Inspired by recent developments in *text-to-speech* systems [1,2]() and I have decided to try putting together a speech transcription system."

[//]: # "The key novelty, in my opinion, of the two papers is that they use [Normalizing Flows](https://arxiv.org/abs/1505.05770)"

[//]: # "his is an attempt to make a small AE model with the normalizing flows for that task. An autoencoder for speech frames is to be constructed. The continuous dynamics of frames and transitions are expected to be captured by transitions in the latent space. By training a flow, the transition matrix based modeling of the HMMs can be replaced by an MCMC technique on continuous space but with proposal distributions that are trained by the neural network. The speaker normalization is a part of the parametrization of the autoencoder,"
[//]: # "(hopefully making it flexible enough for speech style transfer ;)."

 [//]: # " Random notes for speech recognition with NN: "

 [//]: # " ## 18/11/2018 "
 [//]: # " ### Reading the data, first signal analysis results"
 [//]: # " * Found TIMIT dataset on Academic torrents "
 [//]: # " * Played around with transformation from stft/mel/invmel/invstft "
 [//]: # " * audio reconstruction quite good with 80 mel banks (what Andrew Senior mentions that Google uses in [this youtube video](https://www.youtube.com/watch?v=HyUtT_z-cms) ) "
 [//]: # " * fourier size for 16khz: 512 samples (32ms) " 
 [//]: # " * overlap of half-window seems reasonable. "
 [//]: # " * Again from Senior, 26 frames are suggested. This may be a bit excessive, perhaps I should also capture the transitions as some sort of parametrized norm/flow "

 [//]: # " Goal is to squish the high input dimensions fast, with huge matrices and a lot of dropout. "
 [//]: # " At the moment I have complex mel inputs for the network. I'm thinking of treating them uniformly - it doesn't make sense to simply discard them. The network should find out what to do with them."

 [//]: # " # References"
 [//]: # " [1]() [FlowWaveNet](https://arxiv.org/abs/1505.05770)"
 [//]: # " [2]() [WaveGlow](https://github.com/NVIDIA/waveglow)"

