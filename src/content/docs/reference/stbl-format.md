---
title: String Table File (STBL) format
description: A Format of a .package/DBPF file
lastUpdated: 2025-10-06
sidebar:
  order: 50
---

<sup><sub>A reference written by Fogity</sub></sup>

This page documents the String Table (STBL) format.

The format is a key-value store used to store all the game strings.

## Format Specification

```python
- Binary
- Little Endian

== FILE ==
HEADER               file metadata
ENTRY[string count]  key-string entries with metadata

== HEADER ==
byte[4] = "STBL"  file identifier
uint16  = 5       format version
byte    = 0       compressed (unused by Maxis and modders)
unit64            string count
byte[2]           unknown/unused
uint32            total string size if null terminated

== ENTRY ==
uint32                 string key
byte              = 0  flags (unused by Maxis and modders)
uint16                 string size
byte[string size]      string data, UTF-8 (not null terminated)
```

---

Originally written by [Fogity](https://www.patreon.com/c/fogity/) on GitLab.