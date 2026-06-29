# How Drone Signals Work

## Introduction

When you get first start to get into the weeds of signals and "the invisible world", the sheer quantity and heft of information is staggering. There's an unseen universe of activity occurring all around us (and propagating through us) that we basically ignore most of the time, even if we are entirely reliant on the accompanying technology.

Why did I settle on this topic? While most people have at least a basic handle on how gadgets such as cell phones and WiFi routers work, the wide world of unmanned systems is something fresh and new for most folks. The rising tide of AI is also invariably going to lift the boat that is "drones": police drones, military drones, agricultural drones, cinematography drones, racing and freestyle drones, surveying drones - you name it, it's going to explode in popularity in the next 3-5 years.

Also also, on a recent hike, a dear friend of mine asked me a technical question about drones that I couldn't answer well. That bothered me, so I decided to dig into it and write an article.

V, my brother, this one's for you.

## Basic (Yes, Basic) Signal Theory

If you are a radio or signals enthusiast and have an understanding of how these things work (or you just wanna skip to the comparison), click [here](#analog-signal-drones).

Analog signals are like a huge, neon sign with text - the message is just beaming out into the world, globally available to anyone that turns their head to see it. Distance, sign brightness, and local environmental conditions (e.g. if there's a building in the way) are determining factors when it comes to you actually "receiving" the message, and you can even read the sign reflected off of water (or a glass building) to a certain degree. 

In nerd-speak, analog signals are continuous waveforms whose oscillations change depending on what they're representing. When some gadget broadcasts using an analog signal, it is basically blasting information openly over a set frequency, which means that anyone with the technology to intercept those waveforms can both do so and see what's being broadcasted. It's not subtle, and it's typically not meant to be.

Digital signals are packets of binary being broadcasted from a linked transmitter (TX), such as a DJI drone, to its linked receiver (RX), e.g. DJI goggles. There are specific tradeoffs that we'll get into shortly, and way more steps involved in the data transfer. 

Think of two people across a large distance communicating by turning a flashlight off and on in their own proprietary "morse code". Unless you're right between them, it's exceedingly difficult to see that communication is ongoing at all, and even if you see the lights flashing, you can't decode the information that's being sent without a cipher.

We'll also be talking a bit about "line-of-sight" (LOS) principles here, so I'll list out a few bullet points to know:
- the higher the frequency, the shorter the corresponding waveform, and the differences are big - the length of a 2.4 GHz wave (e.g. for WiFi) is just under 5 inches (~12 cm), but an FM radio wave usually sits at around 9-13 feet (~2.8-3.4 meters) long;
- all radio waves interact with their surroundings and either get absorbed, bend, bounce off of things, or simply scatter. A lot of this interaction depends on the wavelength;
- verticality is king (radio horizon scales with the square root of height via the formula $d≈3.96×\sqrt{h}$);

![A graph of the correlation between y and sqrt(x)](/assets/analog-vs-digital-signals/sqrt.png "See? It's better than linear")

## In-depth Signal Theory

Once again, if you wanna skip to the drone stuff, click [here](#analog-signal-drones).

For the purposes of this article, we'll mostly be discussing frequencies in the range that's relevant to multicopters, namely 900 MHz - 6 GHz. We will also be focusing specifically on Unmanned (or Uncrewed) Aerial Systems, since the civilian and military applications of ground and surface (e.g. water) drones are different beasts entirely. Even more specifically, we're focusing on multi-rotor aircraft such as quad/hexa/octocopters with a camera(s) strapped to it - fixed-wing UAS are also very much "their own thing". 

Let's get into the sweet deets with a higher-level comparison to a common civilian technology - FM radio. FM radios, both personal and commercial, use analog technology to transmit their high frequency (HF) signals, but radio waves in this range are still considered to be fundamentally LOS technology, or at least heavily bound by LOS principles.

A local commercial station broadcasts at around 100 W or less, and the difference in "good reception range" can be as much as an order of magnitude or more depending on the topography (e.g. a coastline versus in the mountains). When you scale up to major stations with 150-300 m towers and 100 kW of juice, the differences in good RX range become clear:

![A graph of radio horizon distance based on topography](/assets/analog-vs-digital-signals/001.png "Differences in radio reception by topography") 

Let's take a digital DJI setup for comparison, which broadcasts at frequency of a 5.8 GHz. The wavelength of this video signal (VTX) is only about 5 centimeters (2 inches) long, or roughly 2 orders of magnitude higher than that of FM radio (100 MHz has a wavelength of 3 meters), and this introduces certain challenges:
- higher [path loss](https://en.wikipedia.org/wiki/Path_loss), i.e. signal fidelity falls off faster over the same distance;
- the signal is far more susceptible to absorption by trees, rain, building materials, etc.;
- the signal is much more sensitive to multipath interference; 

There are also more practical constraints to consider. Obviously, no 7-inch quadcopter in this universe is broadcasting at 100 kW, no civilians can get away with having a 300 m antenna tower in their backyard, and no military would make such a terrible tactical decision - electronic intelligence (ELINT) or signals intelligence (SIGINT) teams would easily see a structure like that screaming into the electromagnetic spectrum and move quickly to turn it into constituent parts.

But don't count these little copters out - there's a great video on [Youtube](https://www.youtube.com/watch?v=U6C4aNJlC_k) showing a side-by-side flight comparison with a 1.2/1.3 GHz (600 mW) omnidirectional antenna and a 5.8 GHz (1000 mW) directional antenna. The 1.2 GHz shows a much clearer picture at a mile out, but as the video author says, it's a different "class" of signal that presents its own unique challenges.

The internet has hobbyists **claiming** they're reaching 4-5 miles of range at 25 mW on 5.8 GHz sans fancy setup, but maxing out the range on a quadcopter that actually flies back to you requires a laundry list of aligning stars, such as:
- cleanliness of radio environment (ideally, nowhere close to anything else broadcasting such as cell or radio towers, houses with satellites or WiFi, etc.);
- cleanliness of physical environment (i.e. perfectly open, flat and featureless terrain);
- good weather (i.e. not humid, no fog or clouds, not too hot or cold, etc.);
- special equipment (e.g. directional or high-gain antennas);

But enough about the environment, let's get to the actual components. Both analog and digital civilian pilots use the same basic setup: a drone, a drone-mounted camera, a remote that broadcasts a command signal, and goggles or a monitor, which receives the drone's VTX. 

The drones themselves will each have one motor per propeller, a flight controller (FC) that interprets pilot commands into drone-friendly motor commands, an electronic speed controller (ESC) that regulates motor function, antennas for receiving command signals and broadcasting VTX, as well as a battery.

In both hardware and software terms, this is basically where the similarities end.

## Analog Signal Drones

Analog systems are the uncontested champions of competitive drone racing and freestyle FPV, as well as an ideal option for folks who love taking apart and taking full ownership of things that they own at the cost of a higher learning curve.

### Hardware

A barebones analog drone doesn't actually require any more components than the ones listed above, which is why it's insanely cheap to put together. There are already several worthy guides for assembling your first FPV on the internet already (such as [this one](https://www.mepsking.shop/blog/what-are-the-parts-of-fpv-drone.html)), and they're very handy when you're writing articles and want to inject visuals:

![A list of common FPV parts](/assets/analog-vs-digital-signals/002 "Parts Short List") 

In the Year of our Lawd 2026, both the FC and ESC have several in-built functions/bits that are worth mentioning separately as well. 

Basically modern FCs are going to come with an in-built On-Screen Display (OSD), which is a chip that helps overlay flight data such as voltage, current draw, GPS coordinates, RSSI, speed, etc., onto the video signal before the drone's VTX broadcasts it to the pilot. 

![A screenshot from a drone flying](/assets/analog-vs-digital-signals/003.png "The OSD of My Drone, Zoomie")

The FC for my BetaFPV Meteor75 Pro also has blackbox logging, which would be useful if I were a bit more knowledgeable about my drone's proprotional/integral/derivative (PID) tuning. Fortunately, I haven't had any mechanical crashes either, which I could diagnose with these logs - all my unplanned landings have been the result of pilot error.

Modern ESCs also typically come in 4-in-1 stacks (i.e. 1 ESC unit for 4 motors) that have sensors and/or meters to measure current draw in real time. 

GPS modules are a completely optional add-on that enable various functions such as: return-to-home (RTH), position hold, rescue mode (i.e. the quad turns around and returns to a preset place if it loses the command signal), and broadcasting of GPS coordinates on the OSD. 

Last but not least, a good buzzer or beeper will save your skin when wandering through a field of equally high grass searching for a downed drone. Smaller quads, also known as "Whoops", can generate a buzzing sound simply by providing an electric pulse to the motors, but even I'm thinking of mounting a dedicated pico buzzer onto my unit (also because I want a custom, silly "find me" melody).

### Analog Signal Processing

As stated previously, an analog signal is a continuous, time-varying radio frequency (RF) waveform, which vastly simplifies the process of transferring a video signal to a pilot's goggles or monitor.

The step-by-step breakdown looks like this:

- Image sensor turns light into an electrical signal;
- Sync generator provides pulses for line/frame "timing";
- Frequency modulator generates carrier wave;
- Power amplifier and antenna boost and broadcast the signal;
- Goggle/monitor-side receiver intercepts signal;
- Frequency demodulator "rebuilds" the composite video waveform;
- Sync separator organizes display of lines and frames;
- Remaining light and color information is displayed;

Slightly longer summaries of each step can be found below:

**Step 1)** Row by row, an image sensor in the camera takes in light and reads it out as an electrical signal, producing a varying voltage proportional to the brightness of the image. 

> Fun fact: This is why rows of analog video get "fuzzy" when the signal quality begins to deteriorate!

At the same time, color information is encoded with the help of a subcarrier, forming the "composite" part of composite video.

**Step 2)** A sync generator inserts pulses, or brief voltage dips to a defined level, in between scanlines (i.e. horizontally) and frames (i.e. vertically) to give the receiver a sense of where each line and frame starts and ends. This is all baremetal - a hardwired analog circuit with no software or buffering or true "processing", and adds near-zero latency that is measured in microseconds. 

**Step 3)** A VTX-FM modulator takes the video signal and generates a carrier wave, where the frequency shifts up and down proportionally to the input voltage - remember from  Step (1) that brighter pixels "produce" a different voltage than darker pixels!

**Step 4)** A power amplifier boosts the now-modulated signal to a set output power level (typically 25 mW, 200 mW, 800 mW, etc.) and passes it on to the drone's VTX antenna. 

**Step 5)** The VRX receiver that the pilot set to a specific channel frequency before liftoff is now receiving the modulated signal. There are a few optional sub-processes that can occur at this stage, such as diversity switching, but let's push past that for now.

**Step 6)** The receiver pushes the signal to a frequency demodulator, which undoes the work that the modulator did in Step (3) by reading how the carrier frequency shifts from moment to moment.

**Step 7)** A circuit known as a "sync separator" strips the pulses from Step (2) out of the waveform and uses them to time the display's horizontal and vertical scan. Once the pulses are "stripped" from the signal, all that remains is luminance (i.e. pixel brightness) and color information, which is painted directly on the goggles' display.

### Comparative Advantages

- *Super duper signal speed* - all of this VX transfer occurs at the speed of electronics, not software. A good analog system will have a total end-to-end latency of 1-3 ms, which is why analog setups dominate the racing scene.
- *Signal feedback* - if, for whatever reason, a pixel is corrupted or otherwise fails to reach a pilot's display, the corresponding single scanline (i.e. the horizontal line on the screen that contains the pixel) can distort. This allows the pilot to constantly receive input about the quality of their signal, even without the aid of extra features like "Link Quality" on the OSD. The signal's graceful degradation is incredibly helpful when flying - if your display starts going fuzzy or tearing, then you know there's a problem. 
- *Budget-friendly* - breaking into the hobby doesn't break the bank when you can buy ready-to-fly analog FPV Whoops for less than $100. Due to the nature of the technology involved, the availability of parts and manufacturers, and the onset of 3D printing, putting together an analog kit or buying your first FPV quad has never been cheaper. 
- *Modularity and ease of repair* - analog drones are far less picky about hardware mixing and matching than digital drones, which allows hobbyists and professionals alike to be extremely creative in their tinkering and experimentation. Less manufacturer dependency also means more availability for part repair, replacement, and maintenance, even down to the baremetal - there are a handful of well-supported open source configuration suites (such as BetaFlight) available, which opens the door to all sorts of crazy tuning of the FC and ESCs. I've seen videos of analog quads flying upside down, even - a reality that would be inconceivable with a consumer-grade digital rig. 
- *Reduced power consumption* - analog setups have less circuitry overhead and are more power-efficient at the VTX layer. Claude insists that an analog VTX running at 200 mW transmit power will draw 300-400 mW total from the battery (i.e. transmit power + VTX circuits), whereas a DJI O3 will eat 3-4 W (i.e. 8-10x more) simply due to the many additional modules/systems required. More on that in a moment.
- *Easier spectrum sharing* - in environments where multiple people are flying, each pilot simply picks a different channel within a band, and you can set your drone to fly on whichever one  makes the most sense. This makes potential issues with interference far simpler to diagnose and solve. Digital systems have many mechanisms to ensure signal resilience, but the control and VX channels are set by the manufacturer and can't be adjusted without considerable effort and know-how (and, likely, voiding of any warranty you had). Practically speaking, this means that you and your friend need to have a good degree of physical space between you and be pointed in different directions if you're both flying a digital FPV on 2.4 GHz.

## Digital Signal Drones

We've finally arrived to what most people think of when they hear anything about "FPV" or "camera drones" - out-of-the-box (OOB) systems that allow anyone to capture sweeping, cinematic views shot from formerly impossible-to-reach places.

However, digital systems are fundamentally different from analog ones, right down to the philosophies that underpin them. If analog FPV is intended to be a set of tools and pieces that the pilot assembles and configures manually to their liking, digitals are fully integrated, unified platforms running on proprietary *everything* - the "as-is, what you see is what you get" foil to analog's "build and own your rig".

Underneath their ease of use lies a sophisticated web of interconnected bits that, frankly, the user doesn't really have to worry too much about because it's been designed, optimized, and packaged for your convenience by DJI, Fatshark, Walksnail, or whoever else you bought from. You pick up the drone, hit a button to turn it on, turn on the remote control, see the drone's camera feed on its built-in screen, and you're ready to fly - no binding, no BetaFlight, no channel selection, no PID tuning required.

### Hardware

Notably more sophisticated, largely because digital video signals are working with packets, which require a lot more electronics overhead.

Isolated exceptions notwithstanding (e.g. military use of civilian platforms), your run-of-the-mill digital drone is going to come with and utilize:

- all the mechanical and electrical stuff you have on analog setups;
- "extra" electronics for signal processing;
- "extra" sensors such as GPS, barometer (for determining altitude), magnetometer (for the OSD's compass), sensors for autonomous obstacle avoidance, laser range finders, etc.;
- multiple very fancy cameras;
- gimbal(s) for said fancy cameras;
- SDK / onboard computer - allows code to be run on the aircraft;
- remote ID module, a legal requirement to broadcast drone ID and position, as well as the operator's location;
- LTE/5G datalink module (again, more for fancier setups);
- payload release mechanisms (more for enterprise platforms or military use);

### Signal Processing

![Digital signal processing](/assets/analog-vs-digital-signals/005.svg "Drone-Side Digital Signal Processing")

**Step 1) Image sensor -> ISP** - the image sensor captures a raw Bayer-pattern frame, which is just a grid of RGB photosites, and passes them to the image sensor processor (ISP). The ISP then carries out demosaicing (i.e. interpolating the missing color channels of every pixel), noise reduction, white balance, sharpening, and color grading. If this sounds busy, your instincts are spot on - this is, as Claude said, a "non-trivial compute step even on dedicated silicon".

**Step 2) Codec encoder** - the ISP output goes into a hardware H.264 or H.265 encoder, which encodes and compresses the frames into bitstreams (i.e. streams of 0s and 1s). In order to reduce the compute load, the encoder uses inter-frame prediction to attempt to process only those aspects of each frame that have changed (etc. motion vectors, other residuals), and this is one of the longest steps in the processing chain latency-wise. The encoder has to have the __whole frame__ before it can encode it, and inter-frame prediction requires buffering.

**Step 3) Packetizer + FEC** - the bitstream then gets chopped up into RF-sized packets and Forward Error Correction (FEC) data is affixed to each packet. FEC adds extra bits of information to each packet that help the receiver reconstruct lost or corrupted packets without having to retransmit the entire thing, because there's no time to retransmit packets when you're flying with real-time video. If a pilot is concerned about adding too much latency overhead, they can tune down the amount of FEC the drone is using, but that comes with the tradeoff that their VX signal will be less robust.

**Step 4) OFDM modem** - __I think this is the coolest part and we're BARELY scratching the surface here.......but here we go.__ The packetized data is then modulated onto dozens or hundreds of closely spaced subcarriers via a process known as Orthogonal Frequency Division Multiplexing (OFDM). This OFDM technology makes the drone's signal extremely resistant to all types of signal interference, be it multipath interference (common when flying in urban places where the signal is bouncing off the surroundings and re-colliding with the drone) or even intentional jamming. Because the subcarrier is a small, narrow part of the overall signal band, even if a packet is reflected/corrupted/lost, the FEC can compensate, which keeps the video signal live in places where analog signals would start to get fuzzy. *On top of all of that coolness,* the OFDM modem can also do frequency hopping, which is when the modem automatically swaps between signals to ensure a better connection, as well as adaptive modulation (between QPSK, 16-QAM, 64-QAM, etc.). This is a fierce and wicked little piece of hardware.

**Step 5) RF front-end** - after all that fancy tech, this is straight-up power amplification and antenna switching. In systems with antenna diversity (like DJI's goggles), the receiver continuously evaluates signal quality from multiple antennas and picks the best one per packet.

Now we can move on to the receiver!

![RX side digital signal processing](/assets/analog-vs-digital-signals/006-digital-receiver-side.svg "Receiver-Side Digital Signal Processing")

**Step 1) FEC decoder** - the receiving modem hands off the received packets to the FEC decoder, which identifies and corrects errors, which seems straightforward enough in principle. 

However, this step reveals a crucial difference between analog and digital systems. If the digital signal's link quality is too poor and the FEC cannot recover a packet, the decoder has two choices:

a) request the VTX for a retransmission of the packet (which is too slow and therefore simply never done with FPV drones);
b) attempt to hide the problem by keeping the last good frame on the screen.

There is no graceful degradation of the signal as with analog drones - if the "good packet rate" falls beneath a certain percentage, the video feed drops out altogether until it gets another "good frame". This is what causes the characteristic "screen freeze then full dropout" cliff effect of digital systems when the link quality starts to dip.

Assuming that doesn't happen, though...

**Step 2) Video decoder** the FEC-corrected bitstream goes into a hardware H.264/H.265 decoder, which reconstructs the frame. This step alone adds several milliseconds to the processing time because the decoder has to buffer enough data to reconstruct its inter-frame predictions before it outputs a "good" frame to the operator's goggles/screen.

Altogether, these steps - encode, packetize, transmit, receive, FEC, decode - result in a minimum latency time of 20-40 ms. Certainly much more than with analog drones, but 4k footage for your DVR (and YouTube channel) doesn't just grow on trees.

### Comparative Strengths

- *Signal resilience* - the processing-heavy chain that forces latency onto digital signals also produces incredibly interference-resistant, stable, and high-quality video that is categorically different than analog and retains these characteristics at extremely long ranges.
- *Signal security* - OFDM, frequency hopping, and diversity-enabled platforms makes intercepting a digital signal a significantly more difficult endeavor than sussing out an analog drone rig with a scanner. However, even if you did manage to lock in the signal, all of the packets are encrypted, which makes tapping into the video feed extremely, if not prohibitively, difficult.
- *Lords of the cinema* - digital rigs are the uncontested champions for photography and cinematic applications, allowing operators to capture 4/8K footage in real time with in-built cameras, as well as offering dozens or hundreds of degrees of zoom.
- *Less hassle for beginners* - don't feel like watching hours of YouTube tutorials to choose your rig, bind your remote to your drone, or do troubleshooting in BetaFlight? Neither do a lot of people. Digital drones are bind-and-fly, and the binding happens mostly automatically.
- *Effective training wheels* - built-in object avoidance, "return to home" functions, and less demanding flight modes all make these drones more approachable for people with zero stick time that don't want to worry about paying hundreds of dollars because of a crash (or spring for a separate remote control and simulator).
- *Manufacturer support* - while I don't have experience with this personally, I would assume that having a dedicated party to reach out to for questions, troubleshooting, or repairs is a "piece of mind" factor for many potential buyers.
- *Longer flight times* - LOTS of asterisks here, but extremely generally speaking digital quads can have better battery life and performance because a) the entire surrounding ecosystem (i.e. ESC, FC) is designed, built, and integrated for the specific platform, b) parts on digital drones can be newer and more efficient (e.g. better batteries, more efficient motors, etc.), c) cinematic/photography flying doesn't require battery-draining maneuvers common in analog FPV racing or freestyle.

# Final Thoughts

This emerging hobby has proven to scratch several of my itches at once, which is maybe why it's held my attention for an unusually long amount of time: my interest in signals and radio tech, the love of applying my brain when working with my hands, and my long-established passion for being in the great outdoors.

There are so many flavors and layers to this domain that make it accessible to a wide swath of people, which is also insanely cool. You don't have to be a radio operator or a tech-head to fly, and I'm sure that's exactly what draws people from different walks of life to drones.

I'll leave a few links below for anyone who wants to continue their dive. As always, thanks for reading!

- [FPV Documentary, timestamped to where they break down drones by hobby](https://www.youtube.com/watch?v=UoMWFrqOmQo)
- [Joshua Bardwell - the man, myth, legend - teaching you how to fly](https://www.youtube.com/watch?v=SpuXqNakP2A)
- [One of my favorite freestyle runs](https://www.youtube.com/watch?v=08cDNbkoIuk)
- [Cinema flying with dual-feed (i.e. pilot view and fancy camera view)](https://www.youtube.com/watch?v=YPNtouPtkJY)
- [Side-by-sides of flying with different power outputs and bands](https://www.youtube.com/watch?v=U6C4aNJlC_k)
