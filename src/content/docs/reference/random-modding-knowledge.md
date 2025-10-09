---
title: Collection of Random Modding Knowledge
description: A collection of bits of random modding knowledge too small to become their own page
sidebar:
  order: 99
---

Things you just need to *know* somehow, despite it not being told to you.

Disorganized chaos.

## EA vs. custom animations in interactions

When making a custom interaction cloned from an EA interaction, the outcome being an animation_ref doesn't work. You have to manually set the constrait for the interaction to the animation_ref for the game to handle it properly, otherwise the Sim will do the animation wherever they stand, even if you copied EA's tuning exactly.

Example: Copy the mail checking interaction. EA's version simply has animation_ref as the outcome, and the game handles it. But your cloned, identical version won't work until you add the animation_ref as the constraint.

![An example of the above, showing mailbox_GetMail with an animation_ref as a constraint.](~/assets/mailbox_GetMail-waffle.png)