---
title: Internal Compression of DBPF Files
description: Internal Compression of Database Package (DBPF/.package) Files
lastUpdated: 2026-10-06
sidebar:
  order: 50
---

<sup><sub>A reference written by Fogity</sub></sup>

The internal compression format is a custom compression format available for [package](../dbpf-format/) files. It is used by Maxis for [string table(../stbl-format/)] resources, as it is optimised for text.

The format consists of a series of commands mixed with raw data.

## Format Specification

```python
- Binary
- Big Endian

- Most Significant Bit First

== FILE ==
HEADER      compression metadata
BLOCK[...]  a number of instruction and data blocks
END BLOCK   final instruction and data block

== HEADER ==
bit       extended size
bit[15]   unknown
IF 'extended size' set
  uint16  uncompressed size
ELSE
  uint12  uncompressed size

== BLOCK ==
ANY OF
  BLOCK A
  BLOCK B
  BLOCK C
  BLOCK D

== BLOCK A ==
bit                = 0b0  identifying bit
bit[2]                    highest bits of DO
bit[3]                    DN
bit[2]                    SN
byte                      lowest byte of DO
byte[source count]        data
# source count       = SN
# destination count  = DN + 3
# destination offset = DO + 1

== BLOCK B ==
bit[2]             = 0b10  identifying bits
bit[6]                     DN
bit[2]                     SN
bit[6]                     highest bits of DO
byte                       lowest byte of DO
byte[source count]         data
# source count       = SN
# destination count  = DN + 4
# destination offset = DO + 1

== BLOCK C ==
bit[3]             = 0b110  identifying bits
bit                         highest bit of DO
bit[2]                      highest bits of DN
bit[2]                      SN
byte[2]                     lowest bytes of DO
byte                        lowest byte of DN
byte[source count]          data
# source count       = SN
# destination count  = DN + 5
# destination offset = DO + 1

== BLOCK D ==
bit[3]             = 0b111  identifying bits
bit[5]                      SN
byte[source count]          data
# source count       = 4 * (SN + 1)
# destination count  = 0
# destination offset = 0

== END BLOCK ==
bit[5]             = 0b11111  identifying bits
bit[2]                        SN
byte[source count]            data
# source count       = SN
# destination count  = 0
# destination offset = 0
```

## Decompression

Each block variant can be identified by the starting bits (MSBs), there is no overlap except for the end block which overlaps with block D (the end block should take precedence when parsing blocks).

During decompression the source array will be read once in order, while the destination array may be accessed from anywhere up to (and including) the last written byte.

After parsing a block, the block data (given by the source count) should be appended to the destination array. Then a number of bytes (given by the destination count) should be copied from the destination array and appended to the array (the offset to copy from is given by destination offset). It is safest to do this byte by byte as the bytes to copy might overlap with the bytes being appended (for repeating data).

---

Originally written by [Fogity](https://www.patreon.com/c/fogity/) on GitLab.