---
title: Creating an injector
description: A tutorial for injecting into CareerTuning
sidebar:
  order: 50
---

<sup><sub>A tutorial by januksenkosketus</sup></sub>

This is a tutorial for creating a simple injector script. This tutorial assumes that you have a basic understanding of tuning mods. Previous python and/or programming experience is helpful but not mandatory.

Tools I used:

- Sims 4 Studio
- [Sims 4 Python Script Workspace](https://sims4studio.com/thread/15145/started-python-scripting)

## Let's get started!

So, what's a tuning injector? Tuning injector essentially does the same thing as a tuning mod without having to override the tuning files. Instead of editing the tuning XML you edit the tuning in python.
Typically you don't need to create your own injectors. The Lot51 Core Library has injectors for most use cases, for example for adding interactions to objects. However, sometimes what you want to do is so specific that the core library can't help you.

In this tutorial, we'll create a mod that allows sims with a specific trait start a career at a higher level. Let's take a look at `16051256355318702971<!--careers.career_tuning-->`. You can find a list called `TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS` which contains, well, trait-based career level entitlements. For example, if your sim has a degree in bartending, they can start a culinary career at the first level of the bartender track.

```
<U>
    <T n="benefits_description">0x7836EE61<!--<font color='#ff0000'>Having a relevant university degree means this position will come with higher pay, easier promotions, more vacation days, and a signing bonus. Only one signing bonus can be awarded per Sim every 7 days, regardless of career.</font>\n--></T>
    <L n="career_entitlements">
        <T>10138<!--career_Culinary_Bartender_Level1--></T>
    </L>
    <T n="trait">219807<!--trait_University_BartenderDegree--></T>
</U>
```

In theory, we _could_ just override the career_tuning XML and call it a day. We shouldn't do that though, because it's a huge file that affects all careers in the game. A much safer option for both maintainability and compatibility with other mods is to inject our values into the career tuning.

We need two things for our mod to work: the injector script and the tuning snippet. Let's start with the script.

First, we need to create a new project. I'm using andrew's Sims 4 Python Script Workspace in PyCharm but if there's another tool you're more comfortable with, that's also fine. My file structure looks like this:

```
My Script Mods/
├── career_entitlement_injector/
| ├── Scripts/
| | ├── januksenkosketus_career_entitlement_injector.py
| ├── compile.py
```

You can name your python files whatever you want, but the file name + class name combo has to be unique, otherwise you'll run into problems. It's generally good practice to include your creator name somewhere, and that's why I named mine januksenkosketus_career_entitlement_injector.

First thing our script file needs is the class definition:

```
from sims4.tuning.instances import HashedTunedInstanceMetaclass

class CareerEntitlementInjector(metaclass=HashedTunedInstanceMetaclass,
manager=services.get_instance_manager(Types.SNIPPET)):
```

CareerEntitlementInjector is the name of the class (again, you can use any name you want). Metaclass and manager tell that this is a class for snippet tuning.

Next, we need to define our instance tunables, meaning we have to define the structure of our XML snippet. We're going to make the structure identical to `TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS` in `16051256355318702971<!--careers.career_tuning-->`, which is shown in the example above.

Looking at the snippet, you'll see a `<U>` tag. Inside the tag are two `<T>` tags and one `<L>` tag containing a `<T>`. Dominic M has written a thorough article explaining what each of these tags means and how they work. You can read it if you're interested in more technical details. [Link to the article](https://leroidetout.medium.com/sims-4-tuning-101-a-deep-dive-into-how-tuning-is-generated-from-python-part-2-1a1f5f147c30). For this tutorial, you only need to know that for each of these tags there's a class in python and we need those when defining the snippet structure.

This may seem intimidating and overwhelming at first, but you don't really need to understand the different data types to create mods. Here's a quick explanation:

- U is a TunableTuple, which is a collection that can contain basically any values.
- T usually contains a reference to a tuning file or a string. It's a little bit more complex than that, but that's not relevant in the scope of this tutorial. In this case, `benefits_description` is a TunableLocalizedString and `trait` a TunableReference.
- L is a TunableList.

The structure of our snippet is going to look like this, with example values to make it easier to understand:

```
  <L n="trait_based_career_entitlements">
    <U>
      <T n="benefits_description">0x7836EE61</T>
      <T n="trait">27916<!--trait_Bookworm--></T>
      <L n="career_level_entitlements">
        <U>
          <L n="career_entitlements">
            <T>29946<!--career_Writer_Level3--></T>
          </L>
        </U>
      </L>
    </U>
  </L>
```

In python, it looks like this:

```
    INSTANCE_TUNABLES = {'trait_based_career_entitlements': TunableList(
        tunable=TunableTuple(benefits_description=TunableLocalizedString(
            description='A localized string for entitlement reason'),
            trait=TunableReference(
                manager=services.get_instance_manager(Types.TRAIT),
                allow_none=False),
            career_level_entitlements=TunableList(
                description='A list of career entitlements',
                tunable=TunableTuple(
                    career_entitlements=TunableList(
                        description='A list of career starting levels',
                        tunable=TunableReference(
                            manager=services.get_instance_manager(Types.CAREER_LEVEL),
                            allow_none=False))),
                unique_entries=True),
        ))}
```

It's scary, I know, but let's go through it line by line. First, there's a TunableList called `trait_based_career_entitlements`, just like in our snippet. Inside there's a TunableTuple (`<U>`), that contains `benefits_description`, `trait` and `career_level_entitlements`. I added descriptions to some of the tunables, but that's not mandatory. It's just documentation that could be useful in the future when you're trying to remember what this injector even does. I set `allow_none=False` for `trait` and `career_level_entitlements` which means that the value is required and can't be empty. `benefits_description`, however, can be empty.

The last line has `unique_entries=True`, which means that you can't add the same value to the `career_entitlements` more than once. In this case, it wouldn't really matter if you did, but I just wanted to mention it to show that the option exists.

So this is how our injector looks like now. Note that we need to import the necessary tuning classes for our script to work.

```
import services
from sims4.localization import TunableLocalizedString
from sims4.resources import Types
from sims4.tuning.instances import HashedTunedInstanceMetaclass
from sims4.tuning.tunable import TunableList, TunableTuple, TunableReference


class CareerEntitlementInjector(metaclass=HashedTunedInstanceMetaclass,
                                manager=services.get_instance_manager(Types.SNIPPET)):
    INSTANCE_TUNABLES = {'trait_based_career_entitlements': TunableList(
        tunable=TunableTuple(benefits_description=TunableLocalizedString(
            description='A localized string for entitlement reason'),
            trait=TunableReference(
                manager=services.get_instance_manager(Types.TRAIT),
                allow_none=False),
            career_level_entitlements=TunableList(
                description='A list of career entitlements',
                tunable=TunableTuple(
                    career_entitlements=TunableList(
                        description='A list of career starting levels',
                        tunable=TunableReference(
                            manager=services.get_instance_manager(Types.CAREER_LEVEL),
                            allow_none=False))),
                unique_entries=True),
        ))}
```

Now, on to the injection part. First, we need a method that runs once the tuning is loaded. This method will run every time you boot the game. Note that unlike earlier, you can't rename this method, it has to be called `_tuning_loaded_callback`.

```
    @classmethod
    def _tuning_loaded_callback(cls):
```

We need to inject the data into the career tuning as an ImmutableSlots object. It's a widely used object type in The Sims, and it's basically an object that can't be changed after it's been created. If you want to understand how ImmutableSlots objects work at a deeper level, [here's an old but still relevant guide by Scumbumbo](https://modthesims.info/showthread.php?t=575118). It's not really in the scope of this tutorial though.

Here's the whole injector method:

```
    @classmethod
    def _tuning_loaded_callback(cls):
        create_career_level_entitlement = make_immutable_slots_class(
            ['benefits_description', 'trait', 'career_entitlements'])

        for tunable_entry in cls.trait_based_career_entitlements:
            for entry in tunable_entry.career_level_entitlements:
                entitlement_to_add = create_career_level_entitlement(
                    dict(benefits_description=tunable_entry.benefits_description, trait=tunable_entry.trait,
                         career_entitlements=entry.career_entitlements))
                entitlements = getattr(Career, 'TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS')
                new_entitlements = entitlements + (entitlement_to_add,)
                setattr(Career, 'TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS', new_entitlements)
```

First, we create the immutable slots class. Then, we go through every entry in our `trait_based_career_entitlements` list.

`entitlement_to_add = create_career_level_entitlement(dict(benefits_description=tunable_entry.benefits_description, trait=tunable_entry.trait, career_entitlements=entry.career_entitlements))` creates an ImmutableSlots object with our data.

`getattr(Career, 'TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS')` fetches the entitlement list in `16051256355318702971<!--careers.career_tuning-->`. We don't want to change that directly, instead we create a new list called `new_entitlements` that contains both the original list and our new value. Then, we set the new list as the new value for `TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS`.

And that's it! Here's the whole code with imports:

```
import services
from careers.career_tuning import Career
from sims4.collections import make_immutable_slots_class
from sims4.localization import TunableLocalizedString
from sims4.resources import Types
from sims4.tuning.instances import HashedTunedInstanceMetaclass
from sims4.tuning.tunable import TunableList, TunableTuple, TunableReference


class CareerEntitlementInjector(metaclass=HashedTunedInstanceMetaclass,
                                manager=services.get_instance_manager(Types.SNIPPET)):
    INSTANCE_TUNABLES = {'trait_based_career_entitlements': TunableList(
        tunable=TunableTuple(benefits_description=TunableLocalizedString(
            description='A localized string for entitlement reason'),
            trait=TunableReference(
                manager=services.get_instance_manager(Types.TRAIT),
                allow_none=False),
            career_level_entitlements=TunableList(
                description='A list of career entitlements',
                tunable=TunableTuple(
                    career_entitlements=TunableList(
                        description='A list of career starting levels',
                        tunable=TunableReference(
                            manager=services.get_instance_manager(Types.CAREER_LEVEL),
                            allow_none=False))),
                unique_entries=True),
        ))}

    @classmethod
    def _tuning_loaded_callback(cls):
        create_career_level_entitlement = make_immutable_slots_class(
            ['benefits_description', 'trait', 'career_entitlements'])

        for tunable_entry in cls.trait_based_career_entitlements:
            for entry in tunable_entry.career_level_entitlements:
                entitlement_to_add = create_career_level_entitlement(
                    dict(benefits_description=tunable_entry.benefits_description, trait=tunable_entry.trait,
                         career_entitlements=entry.career_entitlements))
                entitlements = getattr(Career, 'TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS')
                new_entitlements = entitlements + (entitlement_to_add,)
                setattr(Career, 'TRAIT_BASED_CAREER_LEVEL_ENTITLEMENTS', new_entitlements)
```

You can now compile the script into a .ts4script file by running `compile.py`.

One final thing we need is the package with our XML snippet. My snippet looks like this:

```
<?xml version="1.0" encoding="utf-8"?>
<I c="CareerEntitlementInjector" i="snippet" m="januksenkosketus_career_entitlement_injector" n="januksenkosketus:career_entitlement_injector_snippet" s="17429251746300160708">
  <L n="trait_based_career_entitlements">
    <U>
      <T n="benefits_description" />
      <T n="trait">27916<!--trait_Bookworm--></T>
      <L n="career_level_entitlements">
        <U>
          <L n="career_entitlements">
            <T>29946<!--career_Writer_Level3--></T>
          </L>
        </U>
      </L>
    </U>
  </L>
</I>
```

Note that the names of the class and the module must be exactly the same as those of your python class and script file, otherwise the script won't find your snippet.

Now just place your ts4script and package files in your Mods folders and tada! Your sim with the correct trait should now be able to start the specified career(s) at a higher level!

---

Originally written by [januksenkosketus](https://www.curseforge.com/members/januksenkosketus/projects) for this site.
