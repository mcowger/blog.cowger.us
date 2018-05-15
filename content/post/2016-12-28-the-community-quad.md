---
published: true
layout: post
---
# The VMworld Community Quad

Community is such a strong force in technology, and yet often underestimated.  There are dozens of project and tools out there that are successful because of their communities (sometimes in spite of their technical quality *ahem PHP, Drupal, etc*), and some die on the vine because of a failure to nuture community.  I dont have anyway to back this up, but I suspect that the there are times that [community managers](http://jonasrosland.com/) keep a project alive because of their efforts.

## The Setup

At VMworld last year, Aaron Skogsburg from Pure Storage started a small get together just for the vendors.  Why would he do such a thing?  I mean, VMworld is about the customers.  The fact is that the vast majority of the people in the industry, especially enterprise storage, have been around the block.  Its borderline incestuous.  Aaron has worked with me at EMC, I've worked with people at 3PAR, etc.  Its valuable to maintain those reltionaships simply because when something goes really wrong, we might be able to solve a customer problem faster with 2 people who trust each other working together (and then slugging it out on Twitter later!)

So we did this party, and Aaron suggested a contest.  I agreed to design and build a full racing quality quadcopter (drone, if you must) and Pure would pay for it.  We randomly pulled a winner at the event and [Colin Gallagher from the EMC VxRail](https://twitter.com/WorldC3?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor) team was the winner.

## The Build

I need to start out with an apology.  This build took **MUCH MUCH** longer than I planned or promised.  

That being said - the build is done!  Check out the orange goodness:

<<<<<<< HEAD
![]({{site.baseurl}}/images/IMG_0552.jpg)
=======
![:width="700px"]({{site.baseurl}}/images/IMG_0552.jpg)


>>>>>>> 0bf64b33d0e35ac4cf29e47dccef5acfe17fdc03

This is probably the fastest quad I've ever built, so I'm a little sad to let it go.  Full build specs:

- Frame:[Flynoceros Canis M5 V2](https://theflynoshop.com/product/canis-m5-v2/): This frame is an indestructible tank, and will take pretty much any impact.  Its so strong that the designer Josh actually offers a lifetime warrenty.  You break it, and he'll send you a new one.  Good luck breaking it though :).
-[Motors: Multirotor Mania Mini Titan V2 2204-2300KV](http://www.multirotormania.com/22xx-size/1329-mrm-mini-titan-v2-2204-2300-brushless-motor.html).  These are really well regarded motors, with well over 1.1 kg of thust with right right propellers.  To give an idea - unloaded, at full throttle, these motors will spin at 37,000 RPM.  Nuts.
- ESC: The electronic speed controller is responsible for converting requests from the flight controller into actual spinning of the motors.  I used the 30A (yes, 30A - each!) [FlyColor Raptor ESCs](https://hobbyking.com/en_us/flycolor-raptor-mini-30a-f330-powered-blheli-multi-rotor-esc-2-4s-opto.html).  They support all the latest technology and protocols and even run a 48Mhz CPU each.  There are 4 of these, 1 for each motor.   
- Flight Controller: The quad copter is a natuarally unstable aircraft - it needs active efforts at about 1000-4000Hz to keep it level.  As a result, there's an onboard computer to keep everything stable and interpert control requests from the pilot.  I used an old standby, the[BeeRotor F3](http://rctimer.com/product-1527.html) because its reliable, easy to wire and includes easy monitoring of voltage and current usage right out of the box.   Also, it includes the technology required to overlay that data (called an OSD or onscreen display) onto the video.  I had initially tried a different flight controllers, the SirinFPV, but found it unreliable.
- FPV Camera: I used the [RunCam Swift](http://shop.runcam.com/runcam-swift/) as its well understand, reliable, hardy and has great colors.
- Video Transmitter (VTX): I've tried nearly every option for video transmission on the market besides the Connex HD system.  So far, I feel the[LaForge VTx](http://ubuyadrone.com/laforge-5-8ghz-25-200-400mw-switchable-video-transmitter/) system from UBAD gives the best option: It runs on 5V (super easy for wiring), is quite small, uses a pigtail (easy to mount), has a physical swith to disable transmission and supports really simple ways to change channels.
- Other Components:
  - Receiver: FrSky XSR
  - Power Distribution: BeeRotor PDB
  - Props: Dal Props 5", 3 rotor props
- A ton of required accessories (Radio, Goggles, Batteries, Chargers, Spares, etc).

  ## The Result

 The result is a stupidly quick quad.  Testing today I hit 78 MPH, and with different props could probably hit about 85.  Also, it rotates at about 900 degrees per second.

<<<<<<< HEAD
![]({{site.baseurl}}/images/IMG_0553.jpg)

So Colin - next time you are out here in the west I will teach you how to fly this thing (and maybe detune it just a touch while you learn to fly!).


![]({{site.baseurl}}/images/IMG_0554.jpg)
 
 



=======
![:width="700px"]({{site.baseurl}}/images/IMG_0553.jpg)

So Colin - next time you are out here in the west I will teach you how to fly this thing (and maybe detune it just a touch while you learn to fly!).

![:width="700px"]({{site.baseurl}}/images/IMG_0554.jpg)
>>>>>>> 0bf64b33d0e35ac4cf29e47dccef5acfe17fdc03
