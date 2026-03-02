---
title: Collection of Random Modding Knowledge
description: A collection of bits of random modding knowledge too small to become their own page
lastUpdated: true
sidebar:
  order: 99
  badge:
    text: UPD
    variant: note
---

Things you just need to *know* somehow, despite it not being told to you.

Disorganized chaos.

## EA vs. custom animations in interactions

When making a custom interaction cloned from an EA interaction, the outcome being an animation_ref doesn't work. You have to manually set the constrait for the interaction to the animation_ref for the game to handle it properly, otherwise the Sim will do the animation wherever they stand, even if you copied EA's tuning exactly.

Example: Copy the mail checking interaction. EA's version simply has animation_ref as the outcome, and the game handles it. But your cloned, identical version won't work until you add the animation_ref as the constraint.

![An example of the above, showing mailbox_GetMail with an animation_ref as a constraint.](~/assets/mailbox_GetMail-waffle.png)

## Special Characters in In-Game Text

Be very careful in using special characters in in-game text.

Using curly backets (`{}`) in the name of a build/buy item will cause the game to crash when a player searches for anything starting with a letter in the display name. 

Using < on its own, not as part of a tag, will cause the text to break off -- `Earn a < Bachelor's Degree >` becomes `Earn a `. Using `<>` with text in between will result in extra space and no text (`Bachelor's <test> Degree` gets `Bachelor's  Degree`), most likely because < and > are used for formatting (`<i>Bachelor's Degree</i>` will give you <i>Bachelor's Degree</i>).

You may be able to use coding to add those special characters, but pay attention to the rare cases where there's a character limit, because each part of the code will add to the total characters used, and be aware that it seems to be inconsistent if it will work at all, so test in game. If it works in one place, it should work everywhere, but some things just don't work.