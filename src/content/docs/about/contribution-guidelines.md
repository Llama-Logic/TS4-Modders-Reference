---
title: Contribution Guidelines
description: Guidelines to follow if you want to contribute to The Sims 4 Modders Reference
lastUpdated: 2025-09-18
---

**The Sims 4 Modders Reference** has the following guidelines for contributors and contributions alike. Please follow them to the best of your ability when submitting your contributions.

## Naming convention
Please name your files in lower-case, separating each word with an en-dash (-). Follow the example below.

```markdown
contribution-guidelines.md
```

## Title and description 
Pages need a title and a short description. The description won't be visible on the site, but it is visible for searching and embedding. 
These need to be at the top of the page.

Please see the example below:
```markdown
---
title: Scumbumbo's XML File Finder
description: A tutorial for using Scumbumbo XML File Finder
---
```

## Images

All images should have alternative text, also known as "alt text".

## Credits

Whether you submit through GitHub or other means, you can choose to remain anonymous on the site. We will respect your wishes.

If you want to be credited, please contact amethystlilac or Jimantha and provide them with a name and a link to credit you with. Otherwise, your username will be used, as well as any link that can be found (or none, if no link can be found).

The link can be of any website you host your mods in, as long as they're PG-13.

## Possible edits

Content may be edited or rejected for clarity, accuracy, appropriateness, and/or formatting if needed.

## Formatting

### If you're *not* submitting through GitHub

You don't need to format your submission with markdown, or format it in a specific place or website.

If you use markdown, we'll do our best to preserve as it is.

If you format it elsewhere, we'll try to copy the formatting to our best of our ability.

If you use plain text, we'll do our best with what we're given.

### If you're submitting through GitHub

Please use markdown and the guidelines above. If you have any doubts, please refer to the rest of the website for more examples, or contact amethyst lilac or Jimantha through Discord.

<!--Note: If other modders want to be reached out for solving these questions, let me know!-->

## Linking to other resources

### Resources in this website

Please use relative links. Refer to the examples below.

**Resources in the same folder.**

Let's say you're writing a tutorial inside the tutorials folder, and you want to link to xml-extractor.md

You need to do it like this:

```markdown
**[Scumbumbo's XML Extractor](../tutorials/scumbumbo-xml-extractor.md)**
```
The text inside the brackets is the name of the page, and the text inside the parenthesis is the relative link itself.

**Resources in a different folder**

Let's say you're writing contribution-guidelines.md inside the `About` folder, and you need to link to XML Scumbumbo's XML Extractor. To link these succesfully, you need to link it like so:

```markdown
**[Scumbumbo's XML Extractor](../../tutorials/scumbumbo-xml-extractor.md)**
```
And the link will look like the example below.

**[Scumbumbo's XML Extractor](../../tutorials/scumbumbo-xml-extractor)**

### Resources outside this website

To link to resources outside this website, please follow the example below.

```markdown
**[TS4 World Documentation](https://github.com/Kallixer/Ts4-World-Documentation)**
```