---
title: Database Package File (DBPF) format
description: A Format of a .package/DBPF file
lastUpdated: 2025-10-06
sidebar:
  order: 50
---

<sup><sub>A reference written by Fogity</sub></sup>

This page documents the Database Package File (DBPF) format.

The format is a key-value store used to store all the game resources using resource keys. The file consists of a header followed by each resource and finally the resource index (technically, the index could be stored anywhere after the header). The resources can be compressed using one of the supported compression types.

## Format Specification

```python
- Binary
- Little Endian

- Least Significant Bit First

== FILE ==
HEADER  file metadata
...     resource data, possibly compressed
INDEX   resource index with resource metadata

== HEADER ==
byte[4]  = "DBPF"  file identifier
uint32   = 2       major format version
uint32   = 1       minor format version
uint32             major file version (unused by Maxis and modders)
uint32             minor file version (unused by Maxis and modders)
uint32             unknown/unused
uint32             creation time (unused by Maxis and modders)
uint32             update time (unused by Maxis and modders)
uint32             unknown/unused
uint32             index count
uint32             index offset, short (absolute) [1]
uint32             index size
byte[12]           unknown/unused
uint32   = 3       unknown/unused
uint64             index offset, long (absolute) [1]
byte[24]           unknown/padding

== INDEX ==
INDEX HEADER        index options and constant values
ENTRY[index count]  index entry for each resource

== INDEX HEADER ==
uint32    index flags [2]
            0x1 constant type
            0x2 constant group
            0x4 constant instance extension
IF 'constant type' set
  uint32  resource key type
IF 'constant group' set
  uint32  resource key group
IF 'constant instance extension' set
  uint32  resource key instance upper 32 bits

== ENTRY ==
IF 'constant type' not set
  uint32      resource key type
IF 'constant group' not set
  uint32      resource key group
IF 'constant instance extension' not set
  uint32      resource key instance upper 32 bits
uint32        resource key instance lower 32 bits
uint32        resource offset (absolute)
uint31        compressed size
bit           extended entry (always set by Maxis and modders)
uint32        uncompressed size
IF 'extended entry' set
  uint16      compression type
                0x0000 uncompressed
                0x5a42 zlib
                0xFFE0 deleted
                0xFFFE streamable
                0xFFFF internal
  uint16 = 1  committed (does not have any effect in the game)
```

1. If the short index offset is set to 0, then the long index offset is used instead. Maxis always sets the short index offset to 0.
2. If the type, group, or extended instance is the same for all keys in the index, then these values can be included in the index header instead of in each index entry.

## Compression Types

**uncompressed**: The resource is not compressed at all. Usually avoided.

**zib**: The [zlib](https://www.zlib.net/) compression format. Most common compression type.

**deleted**: Not a compression type, used to mark that a resource should be unloaded from the game. Appears in delta packages, uncommon in mods or modding tools.

**streamable**: Unknown format. Not used.

**internal**: [Custom compression format](../internal-compression-dbpf/), used by Maxis for string tables.

---

Originally written by [Fogity](https://www.patreon.com/c/fogity/) on GitLab.