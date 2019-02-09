---
layout: post
title: Reverse engineering IR protocols without a logic analyzer or an oscilloscope
subtitle: 
bigimg: /img/path.jpg
tags: [arduino, reverse engineering, IR protocols, remote control]
---

# The project
Someone on my building block gave away an old sony stereo (with some relatively decent speakers) together with its remote control. So now I could listen to analogue stereo and my old CDs! However there was a problem - I had to take my hands off the keyboard to press the button to change songs or stations! That's no good of course! 

So I creatively wasted a whole weekend to reverse engineer the remote control and use an arduino to control the stereo ;). 

## Recording and Analyzing the signal without an oscilloscope

So first I figured out how the the IR remote works. [This post](https://www.sbprojects.net/knowledge/ir/sirc.php) was really helpful. Now that I knew what to look for in the flashing led, I could start thinking how to record it. I could put together a simple amplifier circuit, but I didn't have an IR receiver module laying around. I had only an IR LED and some transistors but I was to sort out what exactly I needed as a circuit and if it would work out (probably it would) so I took the easy way and started thinking of alternatives!

Bottom line is: there is a "command start" part with a long pulse burst of, say, 4 ms and then a series of bits, varying with the complexity of the device and the remote. The time appointed for each bit is constant and (around) 1.2 milliseconds. The involved receiver circuits are surprisingly forgiving on timing as I learned later on (they can manage some miss-timings for the transmitted bits).

### Sensing with a camera:
Use a cellphone camera! Digital cameras are very sensitive to IR light and some of them have pretty decent frame rates (at least that's what I thought). So here is the deal: I record a video in slow motion (fast shutter) with the cellphone and then I pass the video to my PC and extract the flashing patterns to implement them in arduino afterwards. 

Now here is the thing: the max FPS of my cell phone camera is 240fps. This gives me a 4Hz sampling frequency in time. How can I get to the 2ms resolution? By [Nyquist theorem]( https://en.wikipedia.org/wiki/Nyquist%E2%80%93Shannon_sampling_theorem) we are doomed! However, since I know the basis my signal is represented, (I know more-or-less how the signals look like in time) I can get away by doing sparse regression. That's the theory of [compressed sensing](https://en.wikipedia.org/wiki/Compressed_sensing) at play! I could even be messy and try some [eulerian video magnification](http://people.csail.mit.edu/mrub/vidmag/) on some proper frequency that will allow me to discriminate the 1s and 0s of the signal! 

Both of these approaches can work but they are multi-step complicated approaches. Also I wasn't sure how easy it would be to get the timing of the bursts right (also the carrier frequency should be entirely guessed). I'm looking for ghetto quick and dirty solutions since I have a weekend to finish that! And accurate timing at the same time.



### Sensing with audio input

An ultra simple way to record a 44khz electrical signal is simply using the microphone input of your PC! Even if you have a single jack (no dedicated mic input) you can still record audio with a male-male AUX audio cable. Here is the trick:

[jack with mic and jack without mic](/img/jacks.jpg)

In the picture above you see a "jack" cable with a mic io and another one without one. The difference is the first contact part - that's the mic part! In order to "fake" a microphone to your sound card, you need to put the regular cable partially in your PCs audio IO port! Basically you want the "top" part (which is ground) to contact where it should contact, but the last part of the cable to contact on the part that normally the microphone input would contact. 

So basically I wanted to sense directly the electrical signal out of the infra-red LED:
[LED soldered - connected with crocodile clips (not shown) to the audio jack](/img/led_of_remote.jpg)

Now I could record directly the signal that the led is supposed to transmit! (and maybe some phase distortion from high pass filters in my sound card or sth but this turned out not to be a huge problem). Every time I press a button, a signal that I know more-or-less its parts is going to be emmited from the LED. So now I can simply use a recording software to time my signals! I used my all favorite Audacity:

[Audacity: Matching arduino pulses with remote pulses](/img/audacity_signals.jpg)

In the image above you also see the arduino uno I used for serial communication and pulsing the LED. Since it was easier for development (and because I didn't have a decent IR led laying around) I used the remote's LED for testing my arduino PWM code. One has to use one of the pulse width modulation (PWM) capable output pins on the arduino for hooking the cables connected to the remote's IR led. 

So now I had the exact signals to pulse the LED with and control my stereo! (at least in principle). The final issue to be dealt with was the innacurate timing of the PWM output. Although the receiver was pretty forgiving (no stats on that - sorry!) getting the timings right from the arduino side was a pain. I was using the `delaymicroseconds()` and a simple array for the different commands. The arduino was getting serial signlals (I also used [this post](https://playground.arduino.cc/Interfacing/LinuxTTY) to pass directly from bash the commands I wanted!
So here is the final result:
[Final result](/img/video_remote.mp4)

Bottom line is that I got bored of it in one day - maybe I'll revisit it with this *compressive sensing* idea for getting the signal out of a time sub-sampled video.









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

