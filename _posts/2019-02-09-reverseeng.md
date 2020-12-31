---
layout: post
title: Reverse engineering IR protocols without a logic analyzer or an oscilloscope for stereo remote hacking and robot vacuum control
subtitle: 
bigimg: /img/cables.jpg
tags: [arduino, reverse engineering, IR protocols, remote control, signal analysis, eufy]
---

# The project(s)
I came across an infra-red (IR) controlled stereo and I wanted to make it remote controlled from my PC. Although apparently this is totally unnecessary, it is a useful project since IR is everywhere. 
Later that I wanted to control a robot vacuum cleaner from my PC the things I learned from this project turned out to be quite useful! As a matter of fact I chose a cheaper robot vacuum cleaner since I knew I can easily control it with IR from my PC. This is a short account of where I started and where this project took me. Using a cheap ESP32-CAM module which has a camera and can also run a small server I was able to run my eufy vacuum from wifi.


## Recording and Analyzing the signal without an oscilloscope

First I had to figure out how the IR remote works. [This post](https://www.sbprojects.net/knowledge/ir/sirc.php) was really helpful. Now that I knew what to look for in the flashing led, I could start thinking how to record it. I could put together a simple amplifier circuit, but I didn't have an IR receiver module laying around (after a long time, I bought some IR receivers and transmitters - this makes life much easier). So I took the easy/faster way and started thinking of alternatives. 

## The transmitted signal
Bottom line is that there is a "command start" part with a long pulse burst of, say, 4 ms and then a series of bits, varying with the complexity of the device and the remote. So a "1" is a 0.6 milliseconds pulse followed by a 0.6ms off and "0" is a 0.6ms burst in the carrier frequency followed by 0.6ms off (see also the picture in the following). The receiver circuits are surprisingly forgiving on timing as I figured out later on (they can manage some miss-timings for the transmitted bits). Especially the carrier frequency was not a huge issue to capture and reproduce exactly. This is a simple PSK protocol called [Manchester coding](https://en.wikipedia.org/wiki/Manchester_code). Short note: why wouldn't one encode more info on the amplitude of the signal, you may ask? Later excursions to radio signals using my software defined radio dongle, and reverse engineering RC plane transmitters took me to the word of [quadrature amplitude modulation (QAM)](https://en.wikipedia.org/wiki/Quadrature_amplitude_modulation) which is more-or-less what all high-throughput modern radio uses (plus some [error correcting tricks](https://www.youtube.com/watch?v=Lto-ajuqW3w&list=PLvc8eb5AdU4fPkWSJVLip-g9wcSO6MhAC&ab_channel=Computerphile)). 

### Sensing with a cellphone camera:
Digital cameras are very sensitive to IR light and some of them have pretty decent frame rates (at least that's what I thought). So here is the deal: I record a video in slow motion (fast shutter) with the cellphone and then I pass the video to my PC and extract the flashing patterns to implement them in arduino afterwards. 

<video width="720" height="480" controls="controls">
  <source src="/img/video_remote.mp4" type="video/mp4">
</video>


Both of these approaches can work but they are multi-step complicated approaches (I'm more bored of passing the videos from the cellphone to the PC honestly). Also I wasn't sure how easy it would be to get the timing of the bursts right (also the carrier frequency has to be be entirely guessed). I'm looking for ghetto quick and dirty solutions since I have a weekend to finish that! 

### Short sidenote/update on camera based approach:
Actually after writing this post, I realized that if I set the shutterspeed very high on my samsung S7, I can clearly see the binary code pop out! Here is a picture from the sony remote I managed to get signals such as the following one

![sony remote](/img/sonyremote.jpg)

And a video showing the patterns from an optoma remote (the protocol is different and the pattern containing all the data unfortunately non-repeating):

<iframe width="560" height="315" src="https://www.youtube.com/embed/s1sUocGfjmE" frameborder="0" allowfullscreen></iframe>

the borders of the patterns are quite crisp.

Why this works is much more interesting, considering that the framerate of the camera is much slower than required to capture the signal. It has to do with the fact that the scanning of the sensor is in reality much faster than the "framerate" of the videos. There are two large families of sensors: CCD and CMOS - samsung S7 supposedly has a CMOS which means that every pixel value is amplified on the spot and then serially sent for storage etc. As described in [this very nice video](https://www.youtube.com/watch?v=9vgtJJ2wwMA) some rolling artifacts should have been there. What we observe looks like there is a shift register serving the large dimension of the screen, as if the sensor was a CCD sensor. What I think is happening is that the **do** have a shift register somewhere dealing with the large dimension to deal better with rolling artifacts or for convenient upstream processing. Or they do some upstream processing to deal with rolling artefacts. In any case, **finding binary codes from cellphone may work**.

#### Update (2):
When I started working on the vacuum cleaner project, I still haven't bought the IR receivers yet. 
I gave the high-speed IR sensing idea another try. I wanted to do some simple thresholding  for the purple light of IR that is sensed from the camera. 
The idea was to record a video with multiple pressings of the button, align the incomplete signals and then threshold to get the actual raw IR signal. 
This worked sort-of ok, but with some small additional caveats I detail bellow. Firstly, the LED light does not reach the whole screen with the same intensity. In the following a plot of the mean pixel intensity is shown

![pixintens](/img/pixintens.png)

I normalized and thresholded with a different intensity according to the pixel position - this left me with a clean binary signal that contained some gaps.
I then continued tackling the missing frames issue. I needed a way to somehow align the signals - then by summing them it should be possible to fill-in the blanks. 
With some simple python code, I aligned the incomplete frames according to the preample. This was quite easy 
since the missing frames had the same length and the preample "on" part was quite large and easy to detect. 

At that point, I was hoping that I can simply average or sum the aligned signals and get the final raw IR signal.
However, a different signal is sent every time a button is pressed for the on-off command. This is the first command I needed to rev-eng since otherwise I would need to have the
 vacuum running around while trying to send a signal to it with the microcontroller-mounted IR LED. Here is a plot of some aligned gappy signals that are either on or off commands. 
I also figured out a simple way to sort the gaps for easier visual inspection. 
![mixsig](/img/camera_mixed_signals.png)

Fortunately it is really simple to separate these signals, especially if one considers that they are strictly positive and there are two of them! (the on and off command). 
I first tried some hand-designed tricks (like figuring out visually what bit belongs to just one signal etc) but decided to put some simple ML to good use to do this more effectively.
This can be done by using [non-negative matrix factorization](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.NMF.html). 
The problem is cast to finding two positive components that explain the signal, and in turn to a matrix factorization problem. The algorithm should discover the two signals (the on and off command).

It was easy to cluster the on and off signals in exactly two clusters according to the NMF factor loadings (actually the NMF components *are* the raw on/off signals!). Here is a plot of the signals:

![sepsig](/img/sepsig.png)

The x-axis is labeled pixel/time because at this point I didn't know how fast exactly was the vertical sensing speed of the cellphone camera. I figured this out by finding some timings from the [make eufy smart again](https://github.com/nakulbende/Make-Eufy-Smart-Again) project which did something similar for another eufy model (unfortunately the IR codes didn't match!). I ended up using an IR receiver to get more reliable and accurate results (plus not having to do any video processing). 

### Sensing with audio input

An simple way to record a 44khz electrical signal is simply using the microphone input of your PC. Even if you have a single jack (no dedicated mic input) you can still record audio with a male-male AUX audio cable. Here is the trick:

![jack with mic and jack without mic](/img/jacks.jpg)

In the picture above you see a "jack" cable with a mic io and another one without one. The difference is the first contact part - that's the mic part! In order to "fake" a microphone to your sound card, you need to put the regular cable partially in your PCs audio IO port. Basically you want the "top" part (which is ground) to contact where it should contact, but the last part of the cable to contact on the part that normally the microphone input would contact. 

I wanted to sense directly the electrical signal out of the infra-red LED:
![LED soldered - connected with crocodile clips (not shown) to the audio jack](/img/led_of_remote.jpg)

Now I could record directly the signal that the led is supposed to transmit (and maybe some phase distortion from high pass filters in my sound card or something but this turned out not to be a huge problem). Every time I press a button, a signal that I know more-or-less its parts is going to be emitted from the LED. So now I can simply use a recording software to time the signals. I simply used Audacity for that

![Audacity: Matching arduino pulses with remote pulses](/img/audacity_signals.jpg)

In the image above you also see the arduino uno I used for serial communication and pulsing the LED. Since it was easier for development (and because I didn't have a decent IR led laying around) I used the remote's LED for testing my arduino PWM code. One has to use one of the pulse width modulation (PWM) capable output pins on the arduino for hooking the cables connected to the remote's IR led. The same is true when using other micro-controllers (like the wifi-enabled ESP32 which I used for the vacuum cleaner).

Now I had the exact signals to pulse the LED with and control the stereo (at least in principle). The final issue to be dealt with was the inaccurate timing of the PWM output. Although the receiver was pretty forgiving (no stats on that) getting the timings right from the arduino side was a pain. I was using the `delaymicroseconds()` and a simple array for the different commands. The arduino was getting serial signals. I also used [this post](https://playground.arduino.cc/Interfacing/LinuxTTY) to pass directly from bash the commands I wanted.
So here is the final result using the stereo:

<video width="720" height="480" controls="controls">
  <source src="/img/ir_remote_video.mp4" type="video/mp4">
</video>

# More on the code
For producing the carrier frequency you can either use a dedicated circuit with an oscillator or do it by software and rely on accurate timing and a fast dedicated micro-controller. I went with the latter - this is a technique called [bit-banging](https://en.wikipedia.org/wiki/Bit_banging). 

The problem with bit-banging is that the timing of the micro-controller is not going to be so reliable (the micro-controller has to do other stuff as well). The timing issues were solved by trial and error: I was running the same code with slighly different delays until the signals in Audacity were matching for 1s and 0s. 

# The Eufy Vacuum Cleaner "hack"

Here is a video of the IR controller in action!
<video width="720" height="480" controls="controls">
  <source src="/img/eufy_hack.mp4" type="video/mp4">
</video>

The end goal of this project is to also localize the vacuum (for instance with some simple monocular visual SLAM) and control it from the PC. This is normally ported together with more 
expensive vacuum cleaners but I think it's not difficult to set it up. The algorithm should work offline from the PC initially since I can stream the frames of the ESP-mounted camera.
Other ways I thought about for localizing the vacuum were audio, floor vibrations and by checking the signal strength of surrounding wifi access points. For the audio approach I would 
probably I would need 2-3 mics around the house and some training (the vacuum is noisy so it is audible from the whole house and imperceptible sound distortions due to the layout should help with localization.)




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

