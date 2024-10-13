---
title: "Three Body Problem"
date: 2024-10-12T23:32:45-04:00
draft: true
---

Cixin Liu

&#9734; &#9734; &#9734; &#9734; &#9734;

I can't even come close to understanding how this book won the Hugo award. Not only is it boring and difficult to follow in cultural translation, but it gets so many scientific facts wrong. It's on these demerits that I shall rip the book apart. The Hugo medallion on the cover is the only reason I not only finished reading it, but then didn't throw it directly in the trash. The myriad cross-discipline technical inaccuracies warrant my first 0-star rating.

1. Electromagnetism: You cannot shoot down a satellite with microwaves.

Microwaves work in the kitchen by heating the liquid water inside our food. This specifically works because the microwave wavelength is highly absorbed by liquid water. If there is no water, there is no heat. A glass and metal satellite would simply reflect all the microwaves that hit it. The best you could hope for would be to build up a surface charge that arcs like on a ball of aluminum foil.

2. Cooking: The author has no idea how microwave ovens work.

> Heats food from the inside out.

That's simply wrong, and easily proven. Full disadulation; this is the reason people don't trust scientists.

3. Electromagnetism: Using a ground-based source to microwave satellites would be extrordinarily ineffecient.

Remember that thing about how microwaves only heat water, and not metal? Well it turns out that there is a fantastic amount of water in the Earth's atmosphere. This means that microwave radios are limited to only 10's of miles of range. Increasing the frequency into the millimeter range limits the effective transmission range even further. Attempting to transmit through the first 100 miles of reasonably dense atmosphere would approach total losses. Should the transmitter experience rain, smog, or a high pollen count, the transmitted power would be further attenuated.

4. Computer Science: You cannot solve a chaotic system with a genetic algorithm.

The "three body problem" describes the difficulty in mathematically modeling a system where small input changes cause large observed effects. Genetic algorithms work by making small incremental changes to inputs that reliably move the model towards a solution that converges on a best fit. If the small incremental changes in input do not reliably move the model towards a better fit, the algorithm's scoring function will flail wildly like a (this is the technical term) wacky waving inflatable arm flailing tube man. The algorithm will think that none of the input changes make any difference, because they don't _approach_ a solution. Monte Carlo guessing would be more likely to eventually find an answer because it's not attempting to approach something it will never _approach_. The Monte Carlo method is decried as old, slow, and ineffective in the book.

5. Physics: A tri-solar syzygy would not cause anything to float off the surface of a planet.

If you imagine three suns all direcly in overhead alignment above a planet, they would pull on the whole planet _and_ everything on it _at the same rate_. The book describes objects being pulled up into the air, but if the planet itself were moving as well, there would be no relative motion between the planet and it's surface objects.

6. Computer Engineering: Modeling a whole-ass motherboard is a dumb way to perform calculations.

The analogy of humans holding flags to demonstrate logic gates is neat. I'd use this approach to teach middle schoolers. Using _30 million_ people within a few square miles to create an entire computer would be both logistically impossible and an incredibly dumb way to perform computations. Once the concepts are understood, it's only an exercise in incremental engineered improvements to reduce the example to something more practical, say a series of water buckets, mechanical levers, or maybe even electrical impulses. Further, an entire motherboard is not needed; a CPU and few memory registers would be totally sufficient. Does the author actually mean that the human-formation computer was ready to support a GPU or other PCI-Express cards, USB peripherals, and many types of long-term storage, not to mention all the power cleaning.
