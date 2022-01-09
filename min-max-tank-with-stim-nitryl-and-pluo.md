---
description: By mirrorcult
---

# Min-Max Tank with Stim, Nitryl, and Pluo

As the title suggests, this is a guide to min-maxxing an oxygen tank containing stimulum, nitryl, and pluoxium--gaining the effects of all three.

Stimulum: Stun/sleep immunity

Nitryl: Speedy lad

Pluoxium: Oxygen

<details>

<summary>How Breathing Works</summary>

Assuming you're breathing out of an internals tank, it's ~~fairly~~ simple, but a little unintuitive at first.

First, a humans life() proc is called.&#x20;

Afterwards, handle\_breathing() is called, which calls breathe() every four ticks.

Breathe() handles getting gas from the environment, or in our case an internals tank. It calls get\_breath\_from\_internal(0.5) on the internals tank, which returns a quantity (mol) of air from the tank to your breath equal to distribute\_pressure / (8.31 \* 293.15) \* 2.

Using this gas mixture, breathe() calls check\_breath(), which does all the checks for stim/nitryl/pluox and what not.

</details>

<details>

<summary>Partial Pressure</summary>

Partial pressure is the weirdest part of breathcode.

To get partial pressure (abbreviated as PP from now on), get\_breath\_partial\_pressure(gas\_pressure) is called.

But, this proc's signature is a little misleading; The input isn't actually pressure, its the moles of a certain type of gas--you can see this because the equation for the output takes the form of P = nRt/V.

Partial pressure is the total 'pressure' that one gas type accounts for, which is used for many calculations.

</details>

<details>

<summary>Gas Needed (Long)</summary>

**Stimulum**:

Stimulum requires at least 0.002 mol per breath cycle.

Stimulum's PP doesn't matter (and is actually never calculated), just the mole count. So, we want to be as close to 0.002 mol as possible.

**Nitryl**:

Nitryl also requires at least 0.002 mol per breath cycle, BUT we want to make nitryl\_pp as low as possible to avoid burn damage.

**Pluoxium**:

Pluox is where it gets kind of interesting.

It actually factors into Oxygen's PP, not its own (which leads to an interesting situation where pluoxium is actually never garbage collected, and causes a small memory leak)--but it's 8x as powerful as oxygen in that regard.

So, we need to look at how O2's PP is handled. Weirdly enough, safe\_oxygen\_max is actually equal to 0, and there's a check for if(safe\_oxygen\_max), so you can never have too much oxygen, apparently. However, you need to have O2\_PP be > 16, or, in other words, pluoxium's PP should be >2.

Now we have a number. Since Pluox\_PP should be >2, then we know that nRT/V should be >2--or, in other words,

moles \* 8.31 \* 293.15 / 0.5 = 2

Solving for this in Wolfram Alpha, we see that we need at least 0.00041 mol of pluoxium in our breath to be able to breathe.

</details>

<details>

<summary>Gas Needed (Short)</summary>

So, in total, we need 0.002 mol of Stimulum (at least) and Nitryl, and 0.00041 mol of pluoxium (at least) per breath. This is a total of 0.00441 mol per breath.

We can plug this value into the get\_breath\_from\_internal() function mentioned before like so.

0.00441 = distribute\_pressure / 4872.153

This means that we need a distribute\_pressure of \~21.5 to have enough gas--with exact numbers.

With leeway added at the end, this will be 26.6 kPa (see Ratios section)

</details>

<details>

<summary>Ratios</summary>

Since we know that we need a release pressure of 21.5 kPa, let's work out some ratios.

This is simple--we have 0.00441 mol needed, and we know the mole count for each gas.

But, let's also add some leeway for each gas--stimulum/nitryl get 0.0005 extra, and pluox will get 0.00005 extra.

This totals up to 0.00546 (which is a distribute pressure of 26.6 kPa)// Some co

**Stimulum** = 0.0025 / 0.00546 = \~45.78% -- PP = (0.0025 \* 8.31 \* 293.15) / 0.5 = **12.18 PP**&#x20;

**Nitryl** = 0.0025 / 0.00546 = \~45.78% -- PP = (0.0025 \* 8.31 \* 293.15) / 0.5 = **12.18 PP**&#x20;

**Pluoxium** = 0.00046 / 0.00546 = \~8.44% -- PP = (0.00046 \* 8.31 \* 293.15) / 0.5 = **2.24 PP**

</details>

<details>

<summary>Other Calculations</summary>

I'd also like to know the Nitryl burn damage per tick.

We know we have 12.18 PP of Nitryl, and the formula for burn damage per breath cycle works out to be:

nitryl\_pp / 4, or 12.18 / 4, or 3.045 per 4 life cycles

Because breathing happens every 4 ticks, this works out to be 0.76\~ avg burn damage per tick. This is easily counterbalanced by taking any kind of burn medicine, so we don't really need to worry about it at all.

</details>

<details>

<summary>Summary (gas-mix)</summary>

This is what your end tank should look like:

2533 kPa (or less) 293.15 K (20 deg. Celsius)

Stimulum: 45.78% Nitryl: 45.78% Pluoxium: 8.44%

Release Pressure: 26.6 kPa

</details>
