# 100W Hybrid tube/transistor amp

This document exists to record the project history and my thought process behind the design. It exists just because I like being wordy and describing how I come up with solutions to the problems I have.

## Background

A few years ago, I picked up a copy of Morgan Jones' Valve Amplifiers book. I studied EE in University, but modern curriculums breeze past vacuum tubes as a completely outdated and supplanted tech. Yet they still find use in some audio devices, and I wanted to know more about how they work.

I went through the whole book in a weekend. And in the first few chapters, I was really excited to try and apply the contents and do my own design... but by the end, I was really apprehensive of the whole thing. Tubes are a pain to work with, usually requiring very high voltage supplies, expensive and awkward output transformers that result in peaking in the frequency response, and for all the effort that goes into designing with tubes you end up with an amp that has worse THD+N and lower output power than a much cheaper transistor-based amp.

I shelved the idea of doing a DIY amp for a while after that, until I also got a copy of Douglas Self's Audio Power Amplifier Design book. That book I also devoured in just a few weeks, and started on doing my own DIY "Blameless" inspired amplifier design. Serendepitiously, while I was doing research on DIY audio forums for info about transistors to use in my design, I came upon a thread about "space charge" vacuum tubes. These tubes were commonly used in car radios and operate at MUCH lower voltage; they were designed for car batteries so most worked at 12-24V, and they're also pretty cheap. From here I had the idea to design a hybrid amp using this type of tube. I could have a tube *somewhere* in the signal path, but still get most of the performance and design simplicity of working with transistors and lower voltages. Thus, this project was born.

## Project Definition

I have three strict requirements for this project:

1. Tube somewhere in the signal path: that's the whole inspiration for the project :)
2. Off-the-shelf power supply: I really don't want to mess with mains wiring (especially if I want to be a good citizen and not conduct emissions..), and besides that power transformers are bulky, expensive, and I hate working with them. I just want some DC voltage(s) that I can use out of the box.
3. 50 W into 4-8 Ohm: my speakers technically are rated to 100 W, but I don't think I'd ever listen to them that loud. They're also 4 Ohms, but in case I ever get some 8 Ohm speakers I figure it's worth designing the amp to be capable of both.

Otherwise, my priorities are just that I just want to minimize THD+N as well as have a maximally flat frequency response in the audio band.

I also want to have a volume control knob here as well, meaning I probably need a op-amp preamplifier in the box. For IO, I'll only have one output, my speakers, but I'll have at least 3 or 4 inputs: my turntable, my PC, and maybe one or two other devices (game console or laptop or mp3 player or whatever). The turntable uses RCA jacks, my PC has a few output options but all 1/8 inch jacks, and the other channel will use 1/8 inch jacks as well. However, I don't know if it's typical to just use that kind of jack for inputs and outputs to an amplifier.

## Architecture

Starting with a pretty standard 3-stage amplifier topology based on what I learned in Self's book, I wanted to first answer the question of where I could insert the tube. The typical 3-stage amplifier uses a PNP differential pair transconductance input stage, an NPN transimpedance stage (VAS, for "voltage amplification stage"), and then for a Class AB amplifier, a complementary output stage using emitter-followers. This whole architecture could just as easily be mirrored: NPN diff pair inputs, PNP VAS, and the complementary output remains the same. The reason the "standard" topology starts with a PNP input stage is that they have lower noise (though I couldn't really find any measurements showing this, and I haven't tested it myself).

The way that vacuum tubes work, they don't have a complementary version - they all work like NPNs or NMOS (actually closest analogy per my understanding is an N-type JFET as tubes are enhancement mode devices). Anyway all this means is that the tube has to go in place of an NPN transistor in the signal path, and I thus can't use it in the output stage which requires complementary devices. So that leaves either the input stage as a diff pair, or the VAS stage.

From here, I took a look at my available options. Turns out, most available car radio tubes are probably not ideal for this design. A lot of them are pentodes, which in a pure tube design with a tapped output transformer can be really linear and sound fine, but for this hybrid amp I will basically require just a basic triode regardless of where I put it. So I just picked the highest transconductance tube I could find, which is the ECC86 twin triode. This also basically locks in the architecture. Since it's a twin triode, I may as well use it as a differential pair. Hence, it'll be the input diff pair that uses the tube, and the VAS will be a PNP transistor.

Before I start running calculations, I also wanted to lock in a few other aspects of the design: For the input stage, I'll use just a really basic current mirror load, and a NPN constant current source. For the VAS, I'll use the buffered emitter-follower stage from Self's book, and also I'll include the current limiter safety mechanism from his book, just in case. Then for the output stage, my original plan was just a regular EF output stage (though I'll come back to that later in the design process... :) )

Also, the design will be for a stereo amp. I don't want to mess with a subwoofer or 5.1 surround or anything like that just yet, and I think having separate mono amps for each speaker sounds really overkill for me.

### Power Supplies

With the overall architecture in place, first order of business is to figure out what I'm going to do about power supplies.

I originally started designing with a 48V laptop supply. This couldn't do the full 50W output power, but I didn't mind that too terribly, I could still get >25W which was still quite good for my listening environment. However after playing with it a bit, and running some simulations, I started to worry about the shortcomings, namely that the unipolar voltage involved a lot of weird compromises in the feedback network and required a massive output capacitor to block DC to the speakers. The design technically worked but I couldn't quite get the bandwidth and stability I wanted. So I scrapped that and decided to go with a more traditional bipolar supply.

50W into 8 Ohms is the worst case for voltage, requiring a sqrt(50W \* 8Ohm) = 20 V RMS voltage swing at the output. This is a peak of ~28V, and I'll need some headroom, so I'll design for +/-32V or higher.

Then for current, 20V RMS into 4 ohms is the worst case. Peak current is at that peak voltage of ~28V / 4 Ohms, giving 7.08 A per channel, max. This'll actually be 100W into 4 Ohms which is more than I wanted to design for, but as stated my speakers go to 100W so at least it won't damage anything.

After some searching, seems the most accessible option (and really the only possible fit for this design of all the options I found) is the Connex SMPS300RE. It's a little module that has a bipolar +/-36V output and can do 300W continuous. Given I've got 100W output per stereo channel plus a bunch of aux electronics, this should cover everything worst case. There's two versions, and I decided to go with the one that has a heatsink for a little extra peace of mind. It also has an extra +/-18V aux rail at up to 500 mA, which will be useful as I'll need it for the volume control op-amps. And it's only 100x100x45mm, so it should fit well into the enclosure. It also fortunately needs only an IEC inlet to bring in mains and I just wire it up with some spade terminals so I don't need to do anything complicated with mains (which is something I wanted to avoid!).

I know I'm going to have a few auxiliary electronics components (pre-amp, output relays, LED indicators) which I think I'll power using a linear regulator off of each of the +/-18V supplies as well. I'll pick the component later once I have a better idea of the power draw on these rails.

Lastly, I need some kind of power supply for the heater. This is pretty simple, I need a 6.3V supply capable of 660 mA (330 mA per channel). Since it's just powering a filament, I don't think it needs to be perfect - actually a lot of tube designs just directly use a 6.3V RMS sine wave out of a transformer, so any regulated voltage I make will already be more performant than that. I've got an MC34063 switching regulator chip on-hand already, so I'll just design that to step one of the 36V rails down to 6.3V for the heater.

### Input stage

This one's pretty easy to design, as the tube's performance basically restricts the entire thing, but I did hit one snag during the process. Basically, I originally wanted to run this off of the +/-18V rails and then use a capacative coupler and a level shifter to go into the VAS stage (was inspired for this by some stuff I saw in Jones' book, where he does that kind of thing a lot in the all-tube amps). Unfortunately I just couldn't get it to work right in the simulation, so I had to scrap the idea, and instead I decided to just make it really simple and use a cascode to protect the tubes from overvoltage.

Thus, for the actual design, I'm running it as follows: the tubes form a differential pair, with a current mirror load, and a cascode set to around 25V. Looking at the graphs, if I run 5 mA through each branch at this anode voltage, I'll have about -0.7V for my V_GK. I think this is okay, it looks like it's right in the center of the linear region for this device, and it gives me a gm of about 5-6 mA/V per triode, or 10-12 mA/V for the whole diff pair.

For the current mirror load, I added in 50 Ohm emitter resistors which add a 250 mV drop at the current operating point (side note, I've never properly learned how to size these resistors; I always just pick a value doesn't eat up too much voltage headroom while still being larger than the expected VBE tolerance between the transistors, and that seems to work okay. I don't know if there's a better way to fine-tune them.). For the mirror, I'm using KSA992 transistors, since they're considered low-noise and have a high enough voltage rating to handle the worst case full 72V if there's any weird start-up glitches.

For the cascode, I copied this strategy from Self's book of making a resistive divider to the common emitter node of the diff pair, rather than to each rail (Figure 6.16 in the book). Self uses a cascode for some distortion reduction effect, but I'm basically forced to use it here to protect the anodes from going over their voltage spec. I guess though this will be double benefit for me. I'm using KSC1845 transistors here, and biasing them to about 25-26V, so that the tube V_AK will be about 25V, which is the highest voltage specced out in the datasheet.

Then on the cathodes, I included some "emitter" degredation resistors from each cathode to the common node. In the transistor based designs in Self's book, these help to linearize the input stage through a sort of negative feedback process. The tradeoff is, for a fixed bias current, adding these resistors reduces the effective gm of the pair. With transistors you can manage this by just increasing the bias current, but I'm already at the maximum reasonable bias current for these tubes. Still I'm including the footprint for these in the board and will just fill them with 0 Ohm shunts, and can always test out once I have the physical board if there's any benefit for me to do this.

Then lastly, for the tail current, it is just a pretty standard transistor constant current source, I'm using KSC1845 as I'm already using them for the cascodes. I also stuck a cascode in here too as it's trivially easy to do and there's a whole section in Self's book about how it is a performance improvement.

I was fortunately able to find a spice model for the tube online (https://www.diyaudio.com/community/threads/vacuum-tube-spice-models.243950/page-146) and at least get a rough idea of if the operating point was right. It seemed to follow the predictions quite well, other than that the simulated gm was much lower than the datasheet's curves. However other than that, everything behaved like I expected, so I felt confident to move on to the next stage.

### Transimpedance stage

... aka Voltage Amplification Stage.

I ended up almost completely to a T copying the designs in Self's power amplifiers book.

For a calculation that I will show in here, there's the compensation capacitor. I didn't do anything funky here like 2 pole compensation, I just calculated for a unity gain frequency using the formula from Self's book: H = gm/(w\*C), where gm is the input stage gm and C is the compensation cap. I went for a 4 MHz unity gain, and got C ~= 400 pF. I used that as a starting point in the simulation and decreased the value while staying within the phase margin, and ended up using 100 pF, which gave me a phase margin of 87 degrees in the simulation (side-note: like I said above, the simulation gave an input stage gm that was about half of what I designed for, so what I did for this was to use degenerated transistors to get the same gm and Ic that I should have with the tubes.. there's definitely a lot of inaccuracies with doing this, but it was the best solution I could come up with). I'll be sure to have multiple possible values for this so I can test different values on the final product.

In this stage we also find the VBE multiplier, which I'll talk more about in the output stage section as it's used to bias the output transistors.

I'm using BD140 transistors for the VAS, and a MJE15034 for the bias generator (to make it the same transistor as is used in the drivers); the constant current source for this stage uses the same KSC1845 I used in the input stage. I'm operating this stage at 10 mA bias.

Overall it's pretty basic in this stage. In the book, there's a lot of advanced technique stuff you can do here for performance, but I don't know if it's all overkill for my design. I'm hopeful that my heatsinking is enough that I can get away with a basic VBE multiplier, and I hope that straightforward single capacitor Miller compensation is all that's needed here. I think that by keeping it very basic, I should hopefully be able to see how well the amplifier works, and if I can measure any weird behavior after I actually build it, then I can adjust my design.

### Output stage

Lastly is the output stage. I went for the 3EF design from the book, and am running two output devices in parallel. I did the 3-level emitter follower design because I expected that with multiple output devices in parallel, the current draw on the drivers will be high enough that we need the extra pre-driver stage. Ultimately it's very easy to add this as all I need to do is slightly adjust the VBE multiplier from the book to account for the extra base-emitter drops from the 3rd layer.

I designed the first stage with a pair of MJE15034 and MJE15035 targeting 5-10 mA quiescent current, the second stage with the same transistors targeting 10-15 mA quiescent current, and the output stage using NJW3281G and NJW1302G pairs in a dual/parallel output configration. According to the book, the goal is to set the voltage drop across the emitter resistors to about 25 mV each (so that the total drop in each branch is around 50 mV, which is optimal). This is actually a pretty huge amount of current for 0.1 Ohm emitter resistors, meaning that we've got a standing current of about 250 mA in each output pair. In idle, this works out to about 9W dissipated by each transistor, which feels like a lot (2 channels with 4 devices each means 72W just in idle!), but looking at some example designs it's pretty normal.

Anyway, to get this bias, I need the output of the driver stage to have this 50 mV + 2\*VBE of the output stages across it, which works out to about 1.35V. So for a quiescent 15 mA, I need a 90 Ohm resistor. Then, the pre-driver stage needs that voltage plus 2\*VBE of the driver transistors, which is about 2.65V -> for a 10 mA standing current, I need about 265 Ohms. Then lastly, my VBE multiplier needs to generate the voltage to turn these transistors on, which is about 3.95V. Since the output transistors have a pretty wide spread of turn-on voltages, but also the Vbe_on drops with increasing temperature, I set it up with a potentiometer that lets me select in a range of 2x VBE up to 8x VBE, which I think should be pretty good (in total, the design target is 6xVBE plus 50 mV, so this gives me some headroom).

The output stage has a usual Zobel network as well as a series inductor/resistor pair. I didn't calculate these myself, just used the standard values from the book.

I also kept the feedback very simple, just a pair of resistors to give a gain of about 20 V/V overall. There's also a capacitor to ground in the feedback network, and I calculated the value to give me a low frequency cutoff of around 1 Hz.

Lastly of note is amplifier protection. I don't think I need I-V limiting, as my amplifier's max power is already approaching the max power that the power supply can put out. So I would think the PSU fuse should blow before any protection circuit would kick in anyway. So all I did was put some diodes reverse biased to the rails to prevent from overvoltage flyback in case something weird happens (like if the relay shuts off)

### Auxiliary electronics

Firstly, I wanted a pre-amp to let me use balance and volume controls. To keep this simple, I decided on a really basic op-amp based circuit. I more or less just copied the circuit from here: https://sound-au.com/project88.htm, the logarithmic approximation circuit is particularly cool. The circuit on the website has a selectable gain, but I don't think that's necessary for me. I just put in 2x gain in the second stage to balance out any losses in the volume/balance controller.

The op-amps are powered from the 18V rails through some simple linear regulators.

Another very important auxiliary piece is the delay circuit. Tubes have a warm-up period of about 30 seconds, to give the heaters time to actually get to temperature. Before that, the tube might not conduct at all, or at much lower current. This would result in a large voltage from the anode to the cathode as the other components won't drop the voltage if they aren't conducting; and all this can damage the tube.

There's a lot written both online and in books about these kind of switches, as one of the hard parts is using a relay to switch high current branches of a circuit; you can get arcing, it's hard to disengage a relay at high current, etc., but I had a bit of a realization during the design: if I only use the relay on the first 2 stages, which draw comparatively little current in operation, I shouldn't actually even need one for the output stages. Without current flowing through the VAS branch, the voltage across the VBE multiplier is 0V, and so the drivers and output transistors will all be off. This makes it a LOT easier to manage. I just used a simple timer circuit with a huge resistor, capacitor, and a darlington transistor to set the delay time to about 30 seconds, and the darlington then drives the relays and LEDs.

For the relays themselves, I just found a relay that was good for the power supplies and one that was good for the audio. I've never actually designed with a relay outside of some basic school projects, but I hope the ones I selected are good enough.

I did also include a DC protection circuit. This one I based on one of the circuits on this page: https://sound-au.com/project33.htm. I didn't want to mess around with speaker safety so I wanted something that's actually tried and tested.

### Mechanical components

We're getting past the limits of what I'm used to designing here, and this is less of mathy engineering and more design, so I had to wing it.

For the audio inputs, I wanted to just use a basic 1/8 or 1/4 inch jack but in the end I decided I'd just get standard RCA jacks for everything. I went with 3 total pairs.

For the audio outputs, this was something I knew nothing about; seems looking at some other amplifier designs they just use those screw posts for speakers. That kind of surprised me, even really high end ones seem to use these, but I guess if it's standard it works for me. Got two pairs of red and black, one for the left and one for the right channel.

I've already got two potentiometers on hand. One is 10k, one is 100k, and they slot into the potentiometer slots in the preamp design.

For power, I'm going with a fused IEC inlet with a built-in toggle switch.

Last for the simple components: the pushbuttons. I want to be able to select my input channel with a 1-on radio button array, like was common on old car audio decks and stuff. I really like the tactile effect from those kinds of buttons. I've not finalized the choice of buttons yet, but I found this C&K F-Series of pushbuttons that seems to be exactly what I want, I just have to actually assemble it myself.

### Heatsinking

Things got really tricky here. I had a tough time determining whether it's best to rate the heatsinking for the worst-case peak power, or for the average power of music playing (since music already has a wide dynamic range and won't constantly burn the max power even at max volume). Average power is a lot lower, of course, but I'm also not sure how quickly the temperatures can rise in the event of a short-lived higher volume spike. Also, speakers are not purely resistive loads, and reactive loads result in more power dissipation in the transistors, but I don't have a good model of my speakers to find out.

So here's what I did: I did my best to calculate it with the worst case sine wave for a resistive load driven by a class AB amplifier, and then applied a 2x factor to cover the realistic reactive load that a speaker might possibly have.

The rule of thumb for a class-B amplifier is that the worst power efficiency happens when 40% of the peak instantaneous power is being delivered to the load, at which point half of that amount of power is being burned, on average, by the output stage transistors. This comes out to 36V*36V/4Ohm * 0.2 = 64.8W (the maximum instantaneous power is at the peak of a sine wave at the full 36V amplitude, which is actually a way higher output power to the load than I initially targeted. This is foreshadowing). This power gets combined with the quiescent dissipation of the class AB stage, which is the 260 mA across 36V for each transistor; when there's voltage swing the voltage shifts from one transistor to another but regardless it'll be 260 mA * 36V = ~9W on average in each transistor. So, in total we're dissipating 9W*4 (quiescent total) plus 64.8W (signal total), in each channel. Or another way to think about it is that each transistor has to dissipate 9W plus 64.8W/4 = 25.2 W, into a purely resistive load.  Then to account for reactive loads, I multiply the signal total by 2, so actually the heatsink should be designed so each transistor can handle 9W + 2*64.8W/4 = 41.4 W each; for a total transistor dissipation of 165.6 W per channel.

This is massive and not really feasible for a single passive heatsink. It also, as it turns out, is REALLY close to the limits on the PSU (which can do 300W continuous, 400W peak; for this worst case signal burning 165W in the transistors, we're delivering 33W to the load, times 2 channels and that's 396W, barely under the max).

So at this point, I was basically forced to lower the supply, there's no other option. If I drop the supply voltage from +/-36 to +/-30V, the power number drops quite a bit: max power burned by the output stage transistors becomes 30V*30V/4 * 0.2 = 45W, and the quiescent power draw drops to 260 mA * 30V = 7.8W per device; the total worst-case dissipation drops to 121.2 W per channel. The problem with this approach is that I wouldn't actually get my 100W of power to the load, because I can't actually swing rail-to-rail.

Even if reducing the supply to 30V fixes the total output power problem, that's still a lot of heat to dissipate. So the solution to that is forced air cooling. I could get a quiet fan for each channel and force air over the heatsink blades. It'll be a bit annoying to design the chassis around this as I'll want a cleanable filter to prevent dust from getting in. But this is probably the best option to ensure that the devices run cool all the time without sacrificing any performance.

I went for the hybrid approach. Turns out, the SMPS module I decided on is adjustable, so can actually get the 30V module and turn it up slightly to about 32V, which gets me to that 100W output that I wanted to design for. I also decided to just add 2 fan drive circuits to the board, and use a thermostat switch on each channel to detect temperatures above some threshold to turn on the fans. I'll work out the details of the actual dimensions later when I finalize the PCB layout, but I think the idea will be to have a heatsink that's about 80mm tall and has ~80mm fins, so I can use an 80mm computer fan.

(there is a spreadsheet with accurate and up-to-date calculations in the project folder)

## PCB Layout

Physically, the design is such that the left and the right output channels are separated on their own wings of the board, the pre-amp is in the front towards the center to keep the sensitive traces away from anything high current, the power stuff is in the back center, and the input and VAS stages are sort of at the interface between the center section and the left and right output channels.

The tubes are going to be mounted to the top of the chassis because the whole point of the design is the tubes, and I want them visible :)

In mixed-signal design, the star ground topology is considered extremely dated and quite poor for performance (particularly EMC performance). But apparently for an audio amplifier, it's still the standard way to do it. So the plan is to have 5 branches from the main ground, radiating out of the main power connector: one for the pre-amp, one for each channel's input and VAS, and one for each channel's output.

I'm going to design with a 6-layer board. This way, I can have 2 ground planes, 2 power plane layers, and 2 signal layers. It's more expensive but it makes the design way easier and should massively improve signal integrity. I can route the + and the - voltages each on their own power plane layers to make it extremely simple, and then for the ground planes just be smart about where currents are going and flowing to keep output stage return current away from the pre-amp and input stage.

