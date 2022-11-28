---
layout: main
title: "Over-voltage protection"
date: 2022-11-27 17:32:23 -0700
---

_This blog post is dedicated to my Strymon Blue Sky, which died an untimely death from over-voltage. May others learn from my mistakes._

If you spend enough time in and around guitar pedals online you'll inevitably run across stories of people accidentally destroying their favorite effects with the wrong power supply. Every builder has at least a few stories, usually accompanied by a gruesome photo like this one with exploded electrolytic caps, burnt traces, and diodes turned fuses.

<figure>
  <img src="/assets/over-voltage/explosion.jpg" alt="Circuit board with an exploded electrolytic capacitor">
  <figcaption>aftermath of a 24V AC adapter on a 9V DC pedal, courtesy of Justus Gash</figcaption>
</figure>

Why do people not read the power requirements printed on the pedal? Why did they grab a random wall wart from their ever-growing pile (we all have one of these, let's be honest) and assume it would power their new pedal? It's difficult to not blame the end-user.

_We tried to warn them. If only they had taken the time to read._

## It's not their fault

We live in a world where a wide range of power supplies all share the same 2.1mm plug. A world with no written pedal power supply standard, where we for the most part adhere to an unorthodox (let's face it, center negative is weird) and unwritten design set forth by Boss in the 90s. [Don Norman](https://jnd.org/) would be furious.

Many pedal power supplies can output several voltages, and there's a long history of running pedals at higher voltages to change the sound. Sometimes this is encouraged by the manufacturer, sometimes it's a happy accident that it works at all. Many times it does not work, and bad things happen.

While you can design with higher voltages in mind, there's many circumstances where it's not possible. If you're using a charge pump, for example, you're already stepping things up and going above 9V can get out of spec very quickly!

We need a simple way to keep the pedal safe when powered outside of our desired range.

## Design criteria

Ideally our solution should:

- Use as few parts as possible and be affordable/easy to source
- Keep the pedal from operating until the issue is resolved
- Reset without user-intervention or repair
- Handle a voltage range output by common wall warts/pedal power supplies

It's rare to see a wall wart exceed 24V DC. Maybe 48V if you for some reason have old telecom equipment laying around. While it would be nice to accommodate even higher voltages, this is a decent bar, and will cover most innocent mistakes. We can operate under the assumption that there's inverse polarity protection upstream (I like using [this method](https://www.swampwitchpedals.com/post/i-put-my-thing-down-flip-it-and-reverse-it)).

Let's look at a few options.

## Crowbar

If you search for over-voltage protection circuits, odds are the crowbar is one of the first you'll come across. It's simple, cheap, and effective, but comes with a few catches.

<img src="/assets/over-voltage/crowbar.svg" alt="Crowbar circuit schematic">

At the core of the crowbar is a zener diode (D1) and a thyristor (Q1). The zener blocks current from flowing in reverse until its zener voltage is reached (9.1V in this case). Thyristors are a solid state gate of sorts, in which current doesn't flow until unless a positive voltage is present on the gate. In this configuration, once the 9.1V threshold is reached, current flows to the thyristor gate, which in turn shunts all current to ground. C1 and C2 act as low pass filters and keep stray transients from accidentally triggering the protection circuit, while R1 limits the current through D1 to avoid destroying it in the process.

The shunt to ground is essentially a short circuit, which causes excessive current to flow through the fuse, shattering it. This is a very effective means of protecting downstream electronics, but causes irreversible damage to the fuse. This is not ideal; replacing the fuse requires user intervention, and even though resettable fuses exist, they have numerous drawbacks once triggered.

## Improvement: MOSFET switch

While the crowbar doesn't meet our criteria, there's a few good ideas in there we can adapt to better suit our needs. With a few MOSFETs, we can effectively switch off the current flowing to the rest of the circuit when too much voltage is applied.

<img src="/assets/over-voltage/mosfet-switch.svg" alt="MOSFET switch schematic">

We're using P-channel MOSFETs as switches here; they'll remain "on" so long as the source voltage is higher than the gate voltage. MOSFET choice here is left as an exercise to the reader; a low R<sub>DS</sub> is desirable so as to avoid unwanted voltage drops (an advantage they have over BJTs), and sufficiently high V<sub>SD</sub> (which determines the max voltage we can protect against). If you're already using a MOSFET for polarity inversion protection odds are it'll work just as well in this application.

Q2 acts as our main switch, and remains on during normal conditions because its gate is grounded while the source sits at 9V. When the input voltage is 9V, D1 isn't conducting, and Q1's drain and source sit at similar voltages, keeping it in the "off" state. Once our input voltage exceeds D1's zener voltage and it begins to conduct, the gate of Q1 remains at 9V (with current limited by R1), while its source sits higher, turning it on. Once Q1 is on, Q2's gate becomes sufficiently positive (with R2 limiting current) and it turns off, protecting our circuit.

Try out this [interactive simulation](https://tinyurl.com/2hnllp5m) to get a better idea of what I mean. You might want to play around with the values; I'm using a 10V zener, and the actual cutoff voltage will be V<sub>zener</sub> + V<sub>th</sub> for the MOSFET of choice. For my purposes a strict cutoff at 9V isn't necessary, and this margin of error works fine in practice, but it depends on what's downstream.

## Regulation

This is outside the scope of our initial design criteria, since we want the pedal to fail open when presented with excess voltage, but it's worth talking about. There's something to be said for making as many inputs as possible valid ones, but because of the wide range of possible inputs this is much more difficult and beyond the scope of what myself and most builders are after.

### Linear regulator

If you've messed with electronics very long, you've probably encountered a linear regulator of some sort, such as the venerable L78-series. Why not just slap an L78L09 on the input and call it a day? Unfortunately, most linear regulators require an input voltage a few volts higher than their target voltage. Low-dropout regulators exist and improve upon this but can still have a margin as low as 100mV, which is still iffy if you plan on supporting a wide variety of unknown power supplies.

Large input/output voltage differentials can significantly reduce the efficiency of linear regulators and dissipate heat. Plugging a 24V power adapter into a 9V regulator will draw more current and could potentially heat up to undesirable levels if not designed carefully.

### Switching DC-DC converter

There's a pretty wide swath of devices under the "DC-DC converter" category, with numerous topologies and pros and cons. There tends to be a continuum where the more affordable parts are simpler internally and require more external parts as well as careful planning, while the more complex ones can handle a wider variety of situations with less supporting circuitry. Because oscillators are involved PCB layout becomes extremely important so as to not introduce noise into the audio path.

If you're looking to support a wide variety of input voltages this is a great place to start. Most of the time I don't think this is worth the added complexity and cost.
