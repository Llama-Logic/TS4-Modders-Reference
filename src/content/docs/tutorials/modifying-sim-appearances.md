---
title: Modifying Sim Appearances
description: A tutorial for modifying sim appearances in a mod you're creating for The Sims 4
sidebar:
  order: 50
---

<sup><sub>Originally posted on [Nexus](https://www.nexusmods.com/thesims4/articles/151) by [FellowFur](https://www.nexusmods.com/thesims4/users/42871565).</br>
May need minor updates</sup></sub>

This is a guide for how to modify sim's appearances in a mod you're creating.

This guide assumes a basic knowledge of Sims 4 Modding, including Tuning, Python, and Sims 4 Studio. It focuses primarily on editing appearances through script, since it's much more powerful, but tuning options are also mentioned when available. It's split up into individual sections based on what category of appearance changes you want to make. They are collapsed to be less overwhelming. There's also "Notes" section at the end which has some relevant info, in particular the "Sims with Occult Forms" section is **extremely important** when editing appearances, even if you don't intent to edit any occult forms themselves.

For all code snippets below, the `sim_info` variable refers to the `sim_info` object all sims have. I will not be explaining how to access this variable here, since it is an essential part of sim's script modding and I'm going to assume you already know how to access it.

Also, if you have any recommendations or corrections to make, please let me know, even if it's just something like how to format the code blocks better. Also feel free to ask questions in the comments, or even decompile the mod this article is attached to to see an example of these principles in actual use. (__Note__: the original article, mod, and comment section are located [here](https://www.nexusmods.com/thesims4/articles/151); please ask any questions or leave any comments there.)

## Appearance Categories

### CAS Parts:

<details>

<summary>Show</summary>

CAS Parts include most of the things you can change about a sims appearance, including clothes, hair, tattoos, body/face details, eye colors, and more. In S4S, they have the "CAS Part" type.

To apply a part to a sim, the first thing you need to know is the part's ID, which uniquely identifies it. If you open up a CAS part in S4S and go the the warehouse tab, you can find its ID in the 'Instance' column on the left, as well as under 'Key' -> 'Instance' in the Data tab on the right. Note that this number will be in Hexadecimal, you'll need to convert it to Decimal before using it. This can be done in S4S using 'Tools' -> 'Hash Generator', or in Python with `int({part_id}, 16)`.

You'll also want to make a note of what Category/Outfit Type/Body Type the part uses (I'll just call it Body Type here). It indicates which 'slot' the part occupies on a sim (ie the Pants slot, the Shoes slot, the Lipstick Slot, the Right Cheek Mole Skin Detail slot, etc.). Sims can only ever have 1 part from each Body Type in an outfit. This can be found in S4S in the 'Studio' Tab -> 'Categories' -> 'Outfit Type'. If you have the part's id, you can also get it in Python using `get_caspart_bodytype({part_id})` from `cas.cas`, which will return the Body Type's enum value from the `BodyType` enum in `sims.outfits.outfit_enums`. For reference, in the Notes section I've added a copy of that enum with some notes.

There are 3 methods you can change a sim's CAS Parts:

<details>

<summary>Tuning Method:</summary>

The built-in way of adding CAS parts is to use the `appearance_modifier` property of buffs. When the buff is applied to a sim, and `appearance_modifier` associated with it gets added to the sim, where it will take affect until overwritten or the buff gets removed. I won't go over every property of them since they are documented on TDESC, but here's the basics of how to use them:

- Pick `set`, `remove`, `replace`, or `randomize` `_cas_part` for the modifier, depending on what you want to do with the part
- Put in the part's ID in the `cas_part` field, or the Body Type for 'randomize'
- Use normal methods to apply the buff to the sim

**Notes:**
- If you want the appearance change to last after the buff expires, set the `update_genetics` field to true
- By default, this only affects the current outfit, and will affect the special 'bathing' outfit. Both of those and more can be changed using the `additional_flags` property
- The `priority` property can usually be left alone, but if you want you changes to overwrite other temporary changes you might need to set it

**Limitations of this method:**
- Can only affect the sim's current occult form
- Cannot apply parts to sim's mermaid forms, for some reason
- Cannot target a specific outfit the sim isn't currently wearing
- Cannot apply parts that would be invalid for the sim under normal gameplay
   - For example, you cannot apply masc frame only parts to sims with feminine frames, or toddler parts to adults
   - Some changes are always invalid, for example children can never be given tattoos using this method, even if you have a mod unlocking them in CAS

</details>

<details>

<summary>Hybrid Method:</summary>

This is basically just using the Tuning method, but calling it directly using a script. It has all the same limitations as the tuning method.

For the following examples, the variable `new_part_id` is the id of the part you want to apply to the sim.

For each part you want to add, you'll need to create an individual modifier for it. There are separate functions for each of the `set`, `remove`, `replace`, and `randomize` modifiers, this one is the Set. You can pass in arguments to set all the same properties the tuning method has:

```python
from buffs.appearance_modifier.appearance_modifier import AppearanceModifier, AppearanceModifierPriority

modifier = AppearanceModifier.SetCASPart(cas_part=new_part_id, update_genetics=True)
```

Then you need to apply each modifier to the sim. This function applies them all permanently, which is preferred in script since it won't time itself out if not connected to a buff. There are non-permanent functions, but I'm not familiar with them. Note that it takes a list of modifiers, not just one. The other arguments like you set the additional_flags and priority properties of the modifiers. You can also give the modifier a guid for reference later.

```python
from cas.cas import OutfitOverrideOptionFlags

sim_info.appearance_tracker.apply_permanent_appearance_modifiers(modifier_list, guid=0, priority=AppearanceModifierPriority.INVALID, apply_to_all_outfits=True, additional_flags=OutfitOverrideOptionFlags.OVERRIDE_ALL_OUTFITS)
```

</details>

<details>

<summary>Script Method:</summary>

This method is the most powerful, but also by far the most complicated. It doesn't have the same limitations as the previous, and can do a lot more, if you can get it working.

For the following examples, `new_part_id` is the id of part you are applying to the sim, or `new_part_ids` is a list of those ids.

First, you'll need to know the Body Type for each part you're applying (or removing, or replacing). If you just have a list of parts to add, getting a parallel list of Body Types for each part is simple:

```python
from cas.cas import get_caspart_bodytype

new_part_body_types = [get_caspart_bodytype(new_part_id) for new_part_id in new_part_ids]
```

Next, you'll need to get the sim's internal outfit data to start modifying it. This will give you a list of each outfit the sim has.

```python
from protocolbuffers import Outfits_pb2, S4Common_pb2

outfits_msg = Outfits_pb2.OutfitList()
outfits_msg.ParseFromString(sim_info._base.outfits)
outfits = outfits_msg.outfits
```

You can just loop through this to modify each outfit, or you can identify what each outfit actually is to target or skip specific ones (like the bathing outfit). If you don't care about individual outfits, skip this next block. Sadly the `outfits_msg` above does not track what each outfit is, only the outfits id. So you need to look up your target outfit id elsewhere, and compare it to the ids in the outfits list. To get the id of an outfit, you need to know its category (ie Everyday, Athletic, Special) and its slot (ie Everyday 1-5).

```python
from sims.outfits.outfit_enums import OutfitCategory

# Finds out which outfit in outfits is Everyday 5
# Safety check since some outfits won't have been created
if sim_info.has_outfit(OutfitCategory.EVERYDAY, 5):
    target_outfit = sim_info.get_outfit(OutfitCategory.EVERYDAY, 5)

    # Loop though outfits until we find the one with the matching outfit ID
    for outfit in outfits:
        if outfit.outfit_id == target_outfit.outfit_id:
            target_outfit_data = outfit
            break
```

It also might be useful to use `list(sim_info.get_all_outfit_entries())`, it'll give you a list of tuples with the outfit category and slot of each of the sim's outfits, which you can filter to be only the categories and slots you need, the use `sim_info.get_outfit` to actually get the outfits and their ids, and only edit outfits with those ids.


Once you have an outfit to edit, you need to get the actual parts info from it. Internally, an outfit looks something like this:

```python
"outfit_id": "0x0495108BA18C002B",
"parts": {
    "ids": [
        "0x0000000000050F2A",
        "0x00000000000524F3",
        "0x00000000000524F4",
        "0x000000000005135F",
        "0x000000000005311E",
        "0x0000000000051BB1",
        "0x000000000005520E",
        "0x0000000000053ED4",
        "0x0000000000053B59"
    ]
},
"body_types_list": {
    "body_types": [
        "0x00000003",
        "0x00000005",
        "0x00000008",
        "0x00000023",
        "0x0000003E",
        "0x00000069",
        "0x0000006A",
        "0x0000006B",
        "0x0000006C"
    ]
},
"match_hair_style": true,
"part_shifts": {
    "color_shift": [
        "0x4000000000000000",
        "0x4000000000000000",
        "0x4000000000000000",
        "0x4000000000000000",
        "0x4000000000000000",
        "0x4000000000000000",
        "0x4000000000000000",
        "0x4000000000000000",
        "0x4000000000000000"
    ]
},
"object_ids": {
    "object_id": [
        "0x0000000000000000",
        "0x0000000000000000",
        "0x0000000000000000",
        "0x0000000000000000",
        "0x0000000000000000",
        "0x0000000000000000",
        "0x0000000000000000",
        "0x0000000000000000",
        "0x0000000000000000"
    ]
},
"layer_ids": {
    "layer_id": [
        "0x00000000",
        "0x00000000",
        "0x00000000",
        "0x00000000",
        "0x00000000",
        "0x00000000",
        "0x00000000",
        "0x00000000",
        "0x00000000"
    ]
}
```

When we access them, all the Hexadecimal strings will just be actual numbers, and there are a lot more parts. `parts`, `body_types_list`, `parts_shifts`, and `object_ids` are all parallel lists containing information for each part. I'll mostly focus on `parts` and `body_types_list`, but here's what the others are for:
- `part_shift` really only matter for parts with custom opacity/hue/etc (ie makeup)
- `object_ids` matter for parts that are also objects (ie Crystal Creations jewellery)
- `layer_id` isn't implemented as of writing this, but will probably be used to the upcoming custom tattoo feature

Now, let's actually access the data and start editing it.

```python
target_outfit_part_ids = list(outfit.parts.ids)
target_outfit_body_types = list(outfit.body_types_list.body_types)
target_outfit_color_shifts = list(outfit.part_shifts.color_shift)
target_outfit_object_ids = list(outfit.object_ids.object_id)
target_outfit_layer_ids = list(outfit.layer_ids.layer_id)
```

If you're adding a part and a part with the same Body Type already exists, you can just swap the part_id out. Otherwise you'll have to add a new `part_id`, body type, etc.

```python
import bisect

if new_part_body_type in target_outfit_body_types:
    outfit_part_index = target_outfit_body_types.index(new_part_body_type)
    target_outfit_part_ids[outfit_part_index] = new_part_id

else:
    # Parts are ordered by body type, this inserts the new body type in order
    bisect.insort(target_outfit_body_types, new_part_body_type)
    outfit_part_index = target_outfit_body_types.index(new_part_body_type)

    # Add new part id in same index as body type to maintain parallel lists, add default values for others
    target_outfit_part_ids.insert(outfit_part_index, new_part_id)
        target_outfit_color_shifts.insert(outfit_part_index, 0x4000000000000000) # Hex value, not a string!
    target_outfit_object_ids.insert(outfit_part_index, 0x0000000000000000)
    target_outfit_layer_ids.insert(outfit_part_index, 0x00000000)
```

Similarly, if you want to remove a part or Body Type, make sure it exists and then just delete that entry from each parallel list.

```python
if delete_body_type in target_outfit_body_types:
    outfit_part_index = target_outfit_body_types.index(delete_body_type)
    del target_outfit_part_ids[outfit_part_index]
        del target_outfit_body_types[outfit_part_index]
        del target_outfit_color_shifts[outfit_part_index]
        del target_outfit_object_ids[outfit_part_index]
        del target_outfit_layer_ids[outfit_part_index]
```

**Be careful** when removing parts! Some parts can be freely removed, like makeup or earrings, but some parts will break if you remove them, like hair or shoes. Those parts have specific 'blank' parts that are automatically added in CAS when you remove them, like a 'bald' hairstyle, or barefeet 'shoes'. To remove those parts, you'll want to instead replace their id's with the 'blank' versions appropriate to that sim's frame and age. I've marked which Body Types need to be replaced in the 'Body Types List' under 'Notes'.

Finally, we need to save our modified outfit data back into the game.

```python
outfit.parts = S4Common_pb2.IdList()
outfit.parts.ids.extend(target_outfit_part_ids)

outfit.body_types_list = Outfits_pb2.BodyTypesList()
outfit.body_types_list.body_types.extend(target_outfit_body_types)

outfit.part_shifts = Outfits_pb2.ColorShiftList()
outfit.part_shifts.color_shift.extend(target_outfit_color_shifts)

outfit.object_ids = Outfits_pb2.ObjectIdsList()
outfit.object_ids.object_id.extend(target_outfit_object_ids)

outfit.layer_ids = Outfits_pb2.LayerIdsList()
outfit.layer_ids.layer_id.extend(target_outfit_layer_ids)

sim_info._base.outfits = outfits_msg.SerializeToString()
```

Unfortunately, we are not actually done yet. While the above modifies the sim's outfits, the game also tracks a list of 'genetic' parts. These don't actually have anything to do with genes, they're just parts that are the same for every outfit (ie tattoos, skin details, etc.) If you are modifying a part with a genetic Body Type, it also needs to be changed in the genetic list. While the tuning method handles this for us, the script method requires doing it manually.

I've marked which Body Types are genetic under the 'Body Types List' in the 'Notes' section.

Like before, start by parsing the genetic info.

```python
genetic_msg = Outfits_pb2.GeneticData()
genetic_msg.ParseFromString(sim_info._base.genetic_data)
genetic_parts = list(genetic_msg.parts_list.parts)
```

Genetics parts use a completely different formatting from outfit parts, here's an example of what some of genetic_msg look like internally:

```python
"sculpts_and_mods_attr": "Cjyxx/vyjNCzro8B17yg/dCUud+6AaO",
"physique": "0.000,0.630,0.526,0.000,0.000,0.000,0.000,0.000,0.000,",
"voice_pitch": -0.08,
"voice_actor": "0x6B772518",
"parts_list": {
    "parts": [
        {
            "id": "0x00000000000547CD",
            "body_type": "0x00000002",
            "color_shift": "0x4000000000000000",
            "object_id": "0x0000000000000000",
            "layer_id": "0x00000000",
            "PartName": "yfHair_EP14BraidLoose_NeutralBlack",
            "BodyTypeName": "Hair"
        },
        {
            "id": "0x0000000000024237",
            "body_type": "0x00000004",
            "color_shift": "0x4000000000000000",
            "object_id": "0x0000000000000000",
            "layer_id": "0x00000000",
            "PartName": "yuTeeth",
            "BodyTypeName": "Face"
        },
        {
            "id": "0x000000000000699F",
            "body_type": "0x0000001C",
            "color_shift": "0x4000000000000000",
            "object_id": "0x0000000000000000",
            "layer_id": "0x00000000",
            "PartName": "ymFacialHair_Bald_Black",
            "BodyTypeName": "FacialHair"
        }
    ]
},
"growth_parts_list": {
    "parts": [
        {
            "id": "0x000000000000699F",
            "body_type": "0x0000001C",
            "color_shift": "0x4000000000000000",
            "object_id": "0x0000000000000000",
            "layer_id": "0x00000000",
            "PartName": "ymFacialHair_Bald_Black",
            "BodyTypeName": "FacialHair"
        }
    ]
}
```

The principle is the same as with the outfit parts, if you want to add a part for a Body Type your sim already has, just change the ID, otherwise create a new part. I personally find it convenient to extract all the Body Types to make the logic simpler:

```python
existing_body_types = [part.body_type for part in genetic_parts]

if new_part_body_type in existing_body_types:
    genetic_part_index = existing_body_types.index(new_part_body_type)
    genetic_parts[genetic_part_index].id = new_part_id
else:
    new_part_data = Outfits_pb2.PartData()

    new_part_data.body_type = new_part_body_type 
    new_part_data.id = new_part_body_id
    new_part_data.color_shift = 0x4000000000000000
    new_part_data.object_id= 0x0000000000000000
    new_part_data.layer_id= 0x00000000
    # PartName and BodyTypeName seem to be autofilled

    # Add to parts list, order doesn't matter
    genetic_parts.append(new_part_data)
```

And deleting is again relatively simple, just be careful again to only delete parts that don't have an default 'blank' version.

```python
if delete_body_type in existing_body_types:
    genetic_part_index = existing_body_types.index(delete_body_type)
    del genetic_parts[genetic_part_index]
```

Finally, save the modified genetic info:

```python
sim_info._base.genetic_data = genetic_msg.SerializeToString()
```

Now all the part editing is done! The last step is just to let the game know you've changed the sim's outfit and it should update their appearance in game with the changes.

```python
sim_info.resend_outfits()
```

</details>

</details>

### Growth Parts:

<details>

<summary>Show</summary>

These are what is set in CAS as a sim's 'preferred' length for facial and body hair, and what the hair is reset to if they shave. They can be editted, but I haven't found any way to turn on/off hair growth itself outside of CAS (if you figure it out, let me know!).

To edit it, follow all the steps in the script method for CAS Parts, above, for the genetic parts, except instead of selecting the genetics part list with `genetic_parts = list(genetic_msg.parts_list.parts)`, you'll want to select the growth parts list with `growth_parts = list(genetic_msg.growth_parts_list.parts)`. Otherwise they're treated identically to genetic parts.

</details>

### Skin Tones:

<details>

<summary>Show</summary>

Technically a sim's skin tone can be set in tuning, but you can only randomly select from a set, so I won't both explaining it here (look at the `randomize_skintone_between_tags` appearance_modifier on buffs if you need that). Luckily setting a sim's skin tone with script is trivial.

First, you need to know the skintone's ID, which uniquely identifies it. If you open up a skintone in S4S and go the the warehouse tab, you can find its ID in the 'Instance' column on the left, as well as under 'Key' -> 'Instance' in the Data tab on the right. Note that this number will be in Hexadecimal, you'll need to convert it to Decimal before using it. This can be done in S4S using 'Tools' -> 'Hash Generator', or in Python with `int({skintone_id}, 16)`.

Once you have the skintone id (refered to here as the variable `new_skintone_id`), you can literally just set it.

```python
sim_info.skin_tone = new_skintone_id
```

You can also set the tone shift if you know want it to be, otherwise just setting it to 0 is fine.

```python
sim_info.skin_tone_val_shift = 0.0
```

</details>

### Sculpts/Presets:

<details>

<summary>Show</summary>

These are what show up when you click on a sim's eyes, ears, nose, etc, as quick options to pick between to edit your sim without changing the sliders. They can only be edited through script.

First you need to know the sculpt's ID, which uniquely identifies it. Note here that sculpts and presets usually go together, and **you specifically need the id of the sculpt, not the preset**. If you open up a sculpt in S4S and go the the warehouse tab, you can find its ID in the 'Instance' column on the left, as well as under 'Key' -> 'Instance' in the Data tab on the right. Note that this number will be in Hexadecimal, you'll need to convert it to Decimal before using it. This can be done in S4S using 'Tools' -> 'Hash Generator', or in Python with `int({skintone_id}, 16)`. In these examples, the variable new_sculpt_id will refer to the id of the sculpt.

Next you'll need to parse the list of sculpts the sim currently has:

```python
appearance_attributes = PersistenceBlobs_pb2.BlobSimFacialCustomizationData()
appearance_attributes.ParseFromString(sim_info.facial_attributes)
current_sculpts = list(appearance_attributes.sculpts)
```

That gives just a direct list of the ids of every sculpt currently applied to the sim. Even though only one sculpt can apply to each part of a sim at once, (ie they can only have 1 eye sculpt, 1 mouth sculpt, etc) there is no way to tell what part each sculpt in the list. If you add multiple sculpts that affect the same part of a sim, only the last in the list will take affect. Due to this it is good practice to try and identify if there are any sculpts in the list that would conflict with the one you're adding, and remove it. This can really only be done by tracking what sculpts apply to each part.

To make this easier, I recommend tracking what type of sculpt your sculpt is, and then use that type to pick from a dict of vanilla sculpts, and filtering anything in that list out of your sims sculpts before applying your sculpt. You can skip this but it could cause unexpected behaviors. I've including in the 'Notes' a 'Vanilla Sculpts Dict' that includes all the vanilla sculpts that you can use.

```python
new_sculpt_type = 'chin'

if new_sculpt_type in vanilla_sculpts:
    filtered_sculpts = [sculpt for sculpt in current_sculpts if sculpt not in vanilla_sculpts[new_sculpt_type]
```

With that done, you can just add your sculpt:

```python
filtered_sculpts.append(new_sculpt_id)
```

Now save the updated list back to the game. Due to the data structure the sculpts are saved it, we can't easily remove a specific sculpt from the internal list, or replace it directly with our new list, so the easiest method is just to clear the internal list and add our new list to the now empty internal list.

```python
del appearance_attributes.sculpts[:]
appearance_attributes.sculpts.extend(filtered_sculpts)
sim_info.facial_attributes = appearance_attributes.SerializeToString()
```

Finally, tell the game to instantly update the sim's in-game appearance with their new appearance data.

```python
sim_info.resend_facial_attributes()
```

</details>

### Sliders:

<details>

<summary>Show</summary>

These are all the mouse movement you make on sims to adjust their appearances. I don't actually know how these work, so for now just reference [this thread](https://modthesims.info/showthread.php?t=642941).

</details>

### Fit/Fat Levels:

<details>

<summary>Show</summary>

These are just the Fit and Fat sliders next to your sims, they can be easily edited directly.

```python
sim_info.fit = new_fitness_level
sim_info.fat = new_fatness_level
```

</details>

## Notes:

### Sims with Occult Forms

<details>

<summary>Show</summary>

If you are editing a sim with an occult form via script, even if you aren't editing that form, you need to follow these extra steps. They also tell you how to target a sim's specific form.

Basically, the game keeps track of 1 `sim_info` for the sim's current form, plus 1 occult `sim_info` for each of the sim's forms, **including** the current form. When the sim's form changes, the occult `sim_info` for the form basically replaces the sim's current form. The problem is that if you edit the sim's current form, it does not automatically sync those changes to the equivalent occult `sim_info` for that form. So you need to edit both `sim_infos`, or else when they changes forms and change back, your modifications will be gone.

I recommend keeping track of all the `sim_infos` for forms you want to edit, and then just looping through them, like so:

```python
target_infos = # get editable infos here

for sim_info in target_infos:
    # make appearance modifications
```

For getting the occult forms, there's several important methods to use. This gets form the sim is currently in, returning a **single** value from the `OccultType` enum.

```python
current_occult_type = sim_info.occult_tracker.get_current_occult_types()
```

This method return `True/False` depending on if a sim has an occult type. Note that not all occult type have occult forms.

```python
sim_info.occult_tracker.has_occult_type(OccultType.WITCH)
```

This method returns the actual occult form `sim_info` for a given occult type:

```python
sim_info.occult_tracker.get_occult_sim_info(OccultType.VAMPIRE)
```

As an example, here's how you could get all the infos you need if you wanted to edit only the werewolf form of sims

```python
from sims.occult.occult_enums import OccultType

target_infos = []

# Make sure sim is a werewolf
if sim_info.occult_tracker.has_occult_type(OccultType.WEREWOLF):
    # Always add the werewolf form info
    target_infos.append(sim_info.occult_tracker.get_occult_sim_info(OccultType.WEREWOLF))

    # If the sim is currently in werewolf form, also edit the current form
    if sim_info.occult_tracker.get_current_occult_types() == OccultType.WEREWOLF:
        target_infos.append(sim_info)
```

Here's another example, to edit all the forms of mermaid sims

```python
target_infos = []

# Make sure sim is a mermaid
if sim_info.occult_tracker.has_occult_type(OccultType.MERMAID):
    # Add current info
    target_infos.append(sim_info)

    # Add both occult forms infos
    target_infos.append(sim_info.occult_tracker.get_occult_sim_info(OccultType.MERMAID))
    target_infos.append(sim_info.occult_tracker.get_occult_sim_info(OccultType.HUMAN))
```

A final note, the `sim_infos` gotten from `get_occult_sim_info` are **not** full `sim_infos`. They can be used for appearance modifications, but if you're going to also be modifying skills, traits, etc, you need to make sure the only use the initial `sim_info`.

</details>

### Plant Sims

<details>

<summary>Show</summary>

Plant sims will not be immediately affected by any appearance changes, made from script or from tuning. I'm not sure why. Any changes you make will happen, but won't be visible until the sim stops being a plant sim. The best workaround I've found for this is to just briefly make them not a plantsim after you're done making your changes. This will reset the buff timer and needs, so be careful.

```python
import services
from sims4.resources import Types

trait_manager = services.get_instance_manager(Types.TRAIT)
plantsim_trait = trait_manager.get(162668)
sim_info.remove_trait(plantsim_trait)
sim_info.add_trait(plantsim_trait)
```

</details>

### Getting Related Sim Attributes

<details>

<summary>Show</summary>

<details>

<summary>Getting the Sim's Current Hair Color:</summary>

If you're going to change a sim's hair, you probably want to match their current hair color. The easiest way I've found to do this is to use the hair's tags.

First, get a representative outfit to grab the sim's hair color from. Since all outfits should use the same hair color, I just pick one. Then get the tags for that outfit, filtering to only get tags from the sim's hair.

```python
from sims.outfits.outfit_enums import BodyType

(outfit_category, outfit_index) = list(sim_info.get_all_outfit_entries())[0]
hair_tags = list(get_tags_from_outfit(sim_info._base, outfit_category, outfit_index, body_type_filter=BodyType.HAIR).values())[0]
```

Then you need to see which hair color tag is in the tags list. The tag numbers don't make sense, so I'll provide them here, in the order they appear in CAS:

```python
hair_colors = {
    2528: "NEUTRAL_BLACK",
    131: "BLACK",
    133: "DARK_BROWN",
    2529: "WARM_BROWN",
    132: "BROWN",
    2530: "LIGHT_BROWN",
    136: "RED",
    896: "AUBURN",
    135: "ORANGE",
    2531: "NEUTRAL_BLONDE",
    2532: "LIGHT_BLONDE",
    94: "BLONDE",
    900: "DIRTY_BLONDE",
    96: "PLATINUM",
    905: "WHITE",
    2533: "WHITE_BLONDE",
    134: "GRAY",
    903: "PURPLE_PASTEL",
    902: "HOT_PINK",
    899: "DARK_BLUE",
    904: "TURQUOISE",
    901: "GREEN",
    897: "BLACK_SALT_AND_PEPPER",
    898: "BROWN_SALT_AND_PEPPER"
}

for hair_color in hair_colors.keys():
    if hair_color in hair_tags:
        return hair_colors[hair_color]
```

</details>

<details>

<summary>Getting the Sim's Gender Attributes:</summary>

To make sure you're applying the correct parts, you'll probably want to know what the sim's gender, frame, and style preferences are. These are pretty easy to obtain:

```python
import services
from sims4.resources import Types
from sims.sim_info_types import Gender

def get_gender_tags(sim_info):
    trait_manager = services.get_instance_manager(Types.TRAIT)
    masculine_frame = trait_manager.get(136877)

    # Check if sim's gender is male or female
    if sim_info.gender == Gender.MALE:
        gender = "MALE"
    else:
        gender = "FEMALE"

    # Check if Sim's Frame is masculine or feminine
    if sim_info.has_trait(masculine_frame):
        frame = "MASCULINE"
    else:
        frame = "FEMININE"

    # Check if Sim's Style Preference is masculine or feminine
    if sim_info.has_trait(GlobalGenderPreferenceTuning.MALE_CLOTHING_PREFERENCE_TRAIT):
        style = "MASCULINE"
    else:
        style = "FEMININE"

    return {gender, frame, style}
```

</details>

<details>

<summary>Getting the Sim's Age:</summary>

You may need to know a sim's age to apply the correct parts. For this just get their age and compare it to the Age enum:

```python
from sims.sim_info_types import Age

if sim_info.age == Age.TEEN:
    # etc
```

</details>

</details>

### Body Types List

<details>

<summary>Show</summary>

Here's the game's body type enum, used to differentiate all the 'slots' a sim can have a part in. It may become out of date

Markers:<br>

&nbsp;&nbsp; * = A genetic body type, it must be the same part for all outfits and in the sim's **genetic** parts list<br>
&nbsp;&nbsp; ^ = A **growth** body type, it can be used as a growth part<br>
&nbsp;&nbsp; ~ = A body type that should **never be removed**, always substitute it for a 'blank' part

```python
NONE = 0
HAT = 1
HAIR = 2 *~ # Despite being genetic, hair can be different in different outfits if that outfit's match_hair_style attribute is false.
HEAD = 3 *~
TEETH = 4 *~
FULL_BODY = 5 ~ # Cannot exist at same time as UPPER or LOWER BODY. This part should deleted if those both exist, and vise versa.
UPPER_BODY = 6 ~
LOWER_BODY = 7 ~
SHOES = 8 ~
CUMMERBUND = 9
EARRINGS = 10
GLASSES = 11
NECKLACE = 12
GLOVES = 13
WRIST_LEFT = 14
WRIST_RIGHT = 15
LIP_RING_LEFT = 16
LIP_RING_RIGHT = 17
NOSE_RING_LEFT = 18
NOSE_RING_RIGHT = 19
BROW_RING_LEFT = 20
BROW_RING_RIGHT = 21
INDEX_FINGER_LEFT = 22
INDEX_FINGER_RIGHT = 23
RING_FINGER_LEFT = 24
RING_FINGER_RIGHT = 25
MIDDLE_FINGER_LEFT = 26
MIDDLE_FINGER_RIGHT = 27
FACIAL_HAIR = 28 *^~
LIPS_TICK = 29 # in-game typo lol
EYE_SHADOW = 30
EYE_LINER = 31
BLUSH = 32
FACEPAINT = 33
EYEBROWS = 34 *~
EYECOLOR = 35 *~
SOCKS = 36
EYELASHES = 37
SKINDETAIL_CREASE_FOREHEAD = 38 *
SKINDETAIL_FRECKLES = 39 *
SKINDETAIL_DIMPLE_LEFT = 40 *
SKINDETAIL_DIMPLE_RIGHT = 41 *
TIGHTS = 42
SKINDETAIL_MOLE_LIP_LEFT = 43 *
SKINDETAIL_MOLE_LIP_RIGHT = 44 *
TATTOO_ARM_LOWER_LEFT = 45 *
TATTOO_ARM_UPPER_LEFT = 46 *
TATTOO_ARM_LOWER_RIGHT = 47 *
TATTOO_ARM_UPPER_RIGHT = 48 *
TATTOO_LEG_LEFT = 49 *
TATTOO_LEG_RIGHT = 50 *
TATTOO_TORSO_BACK_LOWER = 51 *
TATTOO_TORSO_BACK_UPPER = 52 *
TATTOO_TORSO_FRONT_LOWER = 53 *
TATTOO_TORSO_FRONT_UPPER = 54 *
SKINDETAIL_MOLE_CHEEK_LEFT = 55 *
SKINDETAIL_MOLE_CHEEK_RIGHT = 56 *
SKINDETAIL_CREASE_MOUTH = 57 *
SKIN_OVERLAY = 58 *
FUR_BODY = 59 *
EARS = 60 *
TAIL = 61 *
SKINDETAIL_NOSE_COLOR = 62 *
EYECOLOR_SECONDARY = 63 *
OCCULT_BROW = 64 *
OCCULT_EYE_SOCKET = 65 *
OCCULT_EYE_LID = 66 *
OCCULT_MOUTH = 67 *
OCCULT_LEFT_CHEEK = 68 *
OCCULT_RIGHT_CHEEK = 69 *
OCCULT_NECK_SCAR = 70 *
FOREARM_SCAR = 71 *
ACNE = 72 *
FINGERNAIL = 73
TOENAIL = 74
HAIRCOLOR_OVERRIDE = 75
BITE = 76 *
BODYFRECKLES = 77 *
BODYHAIR_ARM = 78 *^~
BODYHAIR_LEG = 79 *^~
BODYHAIR_TORSOFRONT = 80 *^~
BODYHAIR_TORSOBACK = 81 *^~
BODYSCAR_ARMLEFT = 82 *
BODYSCAR_ARMRIGHT = 83 *
BODYSCAR_TORSOFRONT = 84 *
BODYSCAR_TORSOBACK = 85 *
BODYSCAR_LEGLEFT = 86 *
BODYSCAR_LEGRIGHT = 87 *
ATTACHMENT_BACK = 88
SKINDETAIL_ACNE_PUBERTY = 89 *
SCARFACE = 90 *
BIRTHMARKFACE = 91 *
BIRTHMARKTORSOBACK = 92 *
BIRTHMARKTORSOFRONT = 93 *
BIRTHMARKARMS = 94 *
MOLEFACE = 95 *
MOLECHESTUPPER = 96 *
MOLEBACKUPPER = 97 *
BIRTHMARKLEGS = 98 *
STRETCHMARKS_FRONT = 99 *
STRETCHMARKS_BACK = 100 *
SADDLE = 101
BRIDLE = 102
REINS = 103
BLANKET = 104
SKINDETAIL_HOOF_COLOR = 105 *
HAIR_MANE = 106
HAIR_TAIL = 107
HAIR_FORELOCK = 108
HAIR_FEATHERS = 109
HORN = 110
TAIL_BASE = 111
UNUSED = 112
```

</details>

### Vanilla Sculpts Dict

<details>

<summary>Show</summary>

Use these to see if a sim has any sculpts that need to be removed when adding yours:

```python
vanilla_sculpts = {
    "chin": [117692097307221872, 1438038647623687400, 1438038647623687404, 2297992739914861240, 2558726282824493165, 2660230991275541721, 2898984971974438224, 2898984971974438226, 2898984971974438230, 2898984971974438231, 2898984971974438234, 2898984971974438235, 2898992668555835761, 2961585556583616931, 2961585556583616935, 3190611621388663303, 3925418774101364443, 4301422848605443192, 4301422848605443195, 4332180867297008160, 4332180867297008163, 5025015685225125297, 5025015685225125298, 5360166786821062476, 5360166786821062479, 5535018121905246717, 5535018121905246718, 5784550092563161389, 6982312531431875427, 6982312531431875431, 6982312531431875438, 6982312531431875439, 6982313630943503673, 6982322427036529304, 7062968458617826185, 7308323871249907877, 7581611612133458742, 7899827251050373029, 7899827251050373030, 8243651520970184946, 8372833322794741838, 8563412084251004794, 8563412084251004798, 8839489795197666173, 8839489795197666174, 9852811741130565368, 9852811741130565371, 10064004328033505394, 10330358697894536113, 10330358697894536114, 10331272775345222773, 10331272775345222774, 10365449816364155705, 10365449816364155706, 10378310597421793703, 11867418350232785849, 11867418350232785850, 12365099576978437174, 12427324639630435858, 12427325739142064096, 12427325739142064100, 12427325739142064108, 12427325739142064109, 12427333435723461507, 12533701423956997602, 12533701423956997603, 12533701423956997605, 12533701423956997607, 12533701423956997614, 12533701423956997615, 12533711319561651480, 12924250253825214257, 12924250253825214258, 13120774838270434702, 13759634936619347528, 13759634936619347531, 14020054972589786992, 14020054972589786995, 14744757478812188986, 15025940610436408359, 15103223933728929825, 15103223933728929826, 16307676186282117263],
    "ear": [62168790478451008, 868258496292115654, 1134838611062166580, 1449277173737049436, 1514966768795050819, 2296589626869061925, 2346105232467418973, 2362172328730911872, 2708300619047110758, 3323230150645375198, 4976810773657562396, 5754056021135194814, 6800609332098539644, 8331015431018670122, 9332358256906476792, 9552856982167370268, 11181875726695239450, 12617304263478911608, 13056491412561988595, 14920139386521715666, 15780762292721114726, 16279455045538566271, 16715245800433173918, 17251909202474675796, 17522448945014517463, 18411646883143309513],
    "eye": [66406458203726796, 66406458203726799, 487446320007593365, 2336268759045615913, 2407209954549704379, 3204880945776777458, 3204880945776777462, 3204880945776777471, 4010727962690370868, 4129115414536984628, 4129115414536984629, 4129115414536984632, 4129115414536984633, 4129115414536984637, 4129115414536984639, 4129116514048612744, 4129116514048612745, 4129116514048612746, 4129116514048612747, 4129116514048612748, 4129116514048612749, 4129116514048612750, 4129116514048612751, 4129125310141638510, 4733190823672668849, 4733190823672668850, 4733190823672668851, 4733190823672668852, 4733190823672668853, 4733190823672668854, 4733190823672668855, 4733190823672668856, 4733190823672668857, 4733191923184296962, 4733191923184296963, 4733193022695925200, 4733193022695925201, 4733193022695925202, 4733193022695925203, 4733193022695925204, 4733193022695925205, 4733193022695925207, 4733193022695925214, 4980263598919600593, 4980263598919600594, 4980263598919600595, 4980263598919600596, 4980263598919600597, 4980263598919600598, 4980263598919600599, 4990329973420779684, 5115099185947268697, 5115099185947268698, 5115099185947268699, 5115099185947268700, 5115099185947268701, 6589298452656506072, 6589298452656506075, 8091266986525136216, 8091266986525136219, 8712944678862036536, 8712944678862036537, 8712944678862036538, 8712944678862036539, 8712944678862036540, 8712944678862036541, 8712944678862036542, 8712944678862036543, 8712945778373664646, 8712945778373664647, 8712945778373664650, 8712945778373664654, 8712953474955062173, 9353770290370648499, 9513394640659455484, 9633017368160744688, 9633017368160744689, 9633017368160744690, 9633017368160744691, 9633017368160744692, 9633017368160744693, 9633017368160744694, 9633017368160744695, 9633018467672372800, 9633018467672372801, 9633018467672372808, 9633018467672372810, 9633018467672372812, 9633018467672372813, 9633026164253770343, 9660376488029049490, 9660376488029049491, 9660376488029049496, 9660376488029049497, 9660376488029049499, 9660376488029049500, 9660376488029049501, 9660376488029049502, 9660376488029049503, 9660377587540677741, 10223441795989241664, 10223441795989241665, 10223441795989241666, 10223441795989241670, 10223441795989241671, 10441827725774618528, 10441827725774618531, 10522334869931611256, 10522334869931611257, 10522334869931611260, 10522334869931611262, 10522334869931611263, 10666488078472390873, 10666488078472390874, 10666488078472390875, 10666488078472390876, 10666488078472390877, 12499006405223370804, 12499006405223370807, 12716048452999884692, 12814612211226432399, 12842395012146674778, 12842396111658303014, 12842396111658303016, 12842396111658303017, 12842396111658303018, 12842396111658303019, 12842396111658303020, 12842396111658303021, 12842396111658303022, 12842396111658303023, 12842397211169931232, 12842397211169931234, 12842397211169931235, 12842397211169931236, 12842397211169931237, 12842397211169931238, 12842397211169931239, 12842397211169931240, 12842397211169931241, 13136343190256107881, 13196283141317935050, 13701163535234106880, 13701163535234106881, 13701163535234106883, 13701163535234106884, 13701163535234106885, 13701163535234106886, 13701163535234106887, 13746299399067037984, 13746299399067037987, 13961577316780196048, 13961577316780196052, 13961577316780196057, 14662748918721623470, 14662750018233251680, 14662750018233251682, 14662750018233251683, 14662750018233251684, 14662750018233251685, 14662750018233251686, 14662750018233251687, 14662750018233251688, 14662750018233251689, 14752796088517234341, 14752796088517234342, 15423678878695424519, 15559924304215836944, 15559924304215836946, 15559924304215836947, 15559924304215836948, 15559924304215836949, 15559924304215836950, 15559924304215836951, 15619857689526980881, 15619857689526980885, 15619857689526980888, 15722568144577697678, 17024470595721414068, 17024470595721414069, 17024470595721414073, 17024470595721414077, 17024471695233042184, 17024471695233042185, 17024471695233042186, 17024471695233042187, 17024471695233042188, 17024471695233042189, 17024471695233042190, 17024471695233042191, 17024480491326067950, 17680426911461394685, 17680426911461394686, 17975702381477803132, 18153815860146515088, 18153815860146515090, 18153815860146515091, 18153815860146515092, 18153815860146515093, 18153815860146515094, 18153815860146515095, 18185272126674156269],
    "jaw": [2129757580972141163, 7241889737660708883, 7459717523308791650, 11493362667950488780, 13711950134733530765, 14412437684349186308, 15275824882801884758, 15381688981853324507, 16109921372881073920, 16109921372881073925, 16109921372881073926, 16109921372881073927, 16557733007180520746, 18187937993343627328, 18187937993343627329, 18187937993343627330, 18187937993343627335],
    "mouth": [379056256518597520, 379056256518597522, 379056256518597523, 379056256518597524, 379056256518597525, 429244643946849533, 429244643946849534, 633201343211631121, 633201343211631122, 1487999643398556736, 1488703778123389912, 1488703778123389915, 2310257688802915366, 2860716707493549851, 3021361008934657207, 3115767106076086184, 3115767106076086185, 3115767106076086187, 3707613566269521539, 4048522222662448601, 4048522222662448602, 4048522222662448603, 4096674233328182044, 4096674233328182045, 4096674233328182047, 5362039432929585377, 6958730976638068840, 7033881458708592970, 7260097754451707040, 7260097754451707042, 7260097754451707043, 7543980117540515964, 7543980117540515965, 7543980117540515966, 7570036349915951537, 7570036349915951541, 7573571646053834172, 7573579342635231570, 7573579342635231571, 7573579342635231577, 7573579342635231579, 7573579342635231580, 7573579342635231581, 7573579342635231582, 7573579342635231583, 7573580442146859820, 7573580442146859821, 7573580442146859822, 7573580442146859823, 8153810080370072614, 8762066288320337184, 8762066288320337186, 8762066288320337187, 9045501290640066770, 9045501290640066771, 9045501290640066772, 9045501290640066773, 9045501290640066774, 9516715000894760166, 9627773014639271114, 10498487101789570740, 10498487101789570743, 10572182501445524094, 10876671820320670515, 10876671820320670519, 11058760378229283547, 11058769174322309192, 11058769174322309193, 11058769174322309194, 11058769174322309195, 11058770273833937428, 11058770273833937429, 11058770273833937432, 11058770273833937433, 11058770273833937434, 11058770273833937435, 11058770273833937436, 11058770273833937438, 11256586282779211720, 11256586282779211724, 11831473014233127208, 11831473014233127211, 12145902685038450172, 12145902685038450175, 12350048413334316946, 12350048413334316947, 12350048413334316948, 12350048413334316949, 12350048413334316951, 12560728912097913691, 12560737708190939336, 12560737708190939337, 12560737708190939338, 12560737708190939339, 12560738807702567572, 12560738807702567573, 12560738807702567576, 12560738807702567578, 12560738807702567579, 12560738807702567580, 13825800934616964704, 13825800934616964707, 13898842818803550875, 14612273809102734499, 14872644518450091572, 14872644518450091575, 15957893632436593972, 15957893632436593973, 15957893632436593977, 15957893632436593978, 15957893632436593979, 15957893632436593981, 15957894731948222092, 15957894731948222093, 15957894731948222094, 15957894731948222095, 15957903528041247854, 16460849439366560313, 17057741180663826232, 17901437305859439122, 17901437305859439123, 17901437305859439124, 17901437305859439125, 17901437305859439127, 18177116191742511028, 18177116191742511031],
    "nose": [516356642587071624, 830194205405788516, 830194205405788519, 1490788059236643847, 1490788059236643848, 1490788059236643849, 1490788059236643850, 1490788059236643852, 1490788059236643853, 1490788059236643854, 1636873390000360772, 1636873390000360775, 1636874489511989016, 1636874489511989019, 1636874489511989020, 1636874489511989022, 1636874489511989023, 1636882186093386551, 1999672946923655816, 3963163148131381147, 4380849005468914643, 4498159596685839777, 5486784528100077152, 5486784528100077155, 5486784528100077156, 5486784528100077157, 5486784528100077159, 5486785627611705396, 5486785627611705399, 5486794423704731044, 5813152306607531696, 5813152306607531697, 5813152306607531699, 5813152306607531700, 5813152306607531701, 5813152306607531703, 5813152306607531706, 6059117517342944863, 7081283718366432208, 7081283718366432210, 7081283718366432211, 7188263382037111192, 7188263382037111194, 7188263382037111195, 7188263382037111196, 7188263382037111197, 7188263382037111198, 7188263382037111199, 7188271078618508727, 7574357815859427722, 7574357815859427723, 7574357815859427724, 7574357815859427725, 7574357815859427727, 7926677853397587489, 8034946617061286984, 8034946617061286985, 8034946617061286986, 8034946617061286987, 8034946617061286989, 8034946617061286990, 8034946617061286991, 8034956512665940990, 8407043862538209840, 8407043862538209841, 8407043862538209843, 8407043862538209844, 8407043862538209845, 8407043862538209846, 8407043862538209847, 8407043862538209850, 8765679565765271357, 8765679565765271358, 8916114449426112780, 8916114449426112783, 9704568545458066838, 10468671514095880459, 10638630900194048786, 10638630900194048787, 10638630900194048788, 10638630900194048789, 10638630900194048790, 10897730083345805801, 10973149658662749680, 12480196085545660108, 12480196085545660111, 12634835355276987940, 13009040218750486294, 13601293640444981016, 13601293640444981019, 14006486307747924553, 14183860894988772993, 14183860894988772997, 14467503341951234956, 14604237694810509640, 14604237694810509641, 14604237694810509643, 15056386833425195506, 15557784064877064382, 16424590501873830088, 16424590501873830089, 16424590501873830091, 16529155481437568040, 16529155481437568042, 16529155481437568043, 16529155481437568044, 16529155481437568045, 16694923907520595431, 17255209339530779126, 17917404598944467328, 17917404598944467330, 17917404598944467331, 17917404598944467332, 17917404598944467333, 17917404598944467334, 17917404598944467335, 17917404598944467337, 18106897049908301300, 18106897049908301301, 18106897049908301302, 18196892441389225575, 18385643681745311843]
}
```

</details>

## Credits

- Thanks to MizoreYukii for help setting parts via tuning 
- Thanks to Lynire's Unify Hair, Makeup, and Tattoos mod for help setting non-genetic parts in script, and managing outfits
- Thanks to Deaderpool for help with setting genetic parts
- Thanks to TurboDriver for help modifying sculpts
- Thanks to Scumbumbo's 'Change Sim Name or Gender' mod for help determining Gender Attributes

---

Originally posted on [Nexus](https://www.nexusmods.com/thesims4/articles/151) by [FellowFur](https://www.nexusmods.com/thesims4/users/42871565).