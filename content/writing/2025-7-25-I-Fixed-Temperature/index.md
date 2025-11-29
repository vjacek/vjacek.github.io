---
title: "I Fixed Temperature"
date: 2025-07-25
draft: false
---

All 3 temperature systems [1], [2] have some major problems:

**Kelvin**

Astronomically, the Kelvin scale makes a lot of sense. Given the vast general emptiness of space, you might have a pretty reasonable need to use the low end of the spectrum, and absolute zero is a wonderful place to start. But for temperatures that humans might experience, it's pretty far off from the Most Useful Numbers (0-100).

**Celcius**

As we move to Celcius, we lose absolute truth at absolute zero, and make the critical improvement of becoming based on water, the Most Important Molecule. For humans, nothing could make more sense than basing our discussion and measurements around water, upon which our existence is already naturally based. The problem is that we don't get to use very much of the scale in everyday life. You might be able to experience the range from -25C to 45C in a lifetime, but that's only a total difference of 70 degrees. In common usage, 10C would be considered cool, and 27C warm. This gives an absolutely pathetic resolution of 17 degrees! The difference of a degree is so large that decimal points become common. This silliness must be another major reason the United States has not largely adopted Celcius.

**Fahrenheit**

The strength of Fahrenheit is that it has a high resolution for temperatures that humans might experience. Anywhere from -20 to 120 is pretty reasonable for an average person to feel throughout their life; this gives us 140 degrees to work with, and here in New England is's entirely reasonable to see -10 to 90 in a single year. High resolution is so important because it gives precision and allows us to easily distinguish nearby ranges. We generally think anything under 50F is cool, and over 80F is quite warm, that's the most used 30 degrees.

**V**

This brings us to the hero of our story, degrees V. Similar to the Radian's unitlessness, V is self-complete and therefore has no other full name. V is simple: it is 2 \* Celcius. It retains the all-important relation to water; freezing is still 0, and boiling becomes the no-harder-to-remember 200. Most usefully, we also gain resolution at the human scale. Using the same examples from Celcius and Fahrenheit above, -50V to 90V become entirely reasonable, with 20V to 55V being cool to warm. That's a whopping 35 degrees of range that we can use to describe comfortable indoor temperatures!

Enjoy the precision of `F` but you're not a fucking moron? Start using `V`s in your home today!

For readers in the United States, this table based on some selected Fahrenheit values gives clear comparisons:

|                  | Kelvin | Fahrenheit | Celcius | V   |
| ---------------- | ------ | ---------- | ------- | --- |
| Water: Boiling   | 273    | 212        | 100     | 200 |
| Human: Too Hot   | 305    | 90         | 32      | 64  |
| Human: Warm      | 297    | 75         | 24      | 48  |
| Human: Cool      | 283    | 50         | 10      | 20  |
| Water: Freezing  | 373    | 32         | 0       | 0   |
| Human: Very Cold | 250    | -10        | -23     | -46 |

My home grafana dashboard is another fun way of viewing the differences. Both scales show the same high resolution 30-ish degree range, but `V`s are based on physics instead of random local conditions.

![Grafana](2025-7-25-I-Fixed-Temperature-Screenshot-1.png)

1. I'm absolutely not going to even justify the insanity of the Rankine scale by including it in the discussion.
2. There appear to be a few other temperature scales as well: Delise, Newton, Reaumur, and Romer. Although historically interesting, they were not purposefully designed but a byproduct of specific measurement techniques.
