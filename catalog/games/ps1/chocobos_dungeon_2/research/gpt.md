# Chocobo’s Dungeon 2 (PS1) – Save Checksum Reverse Engineering

## Overview

This project documents the reverse engineering of the save file integrity check used in *Chocobo’s Dungeon 2* (PlayStation).

The game uses a simple **32-bit XOR checksum** to validate save data.

---

## Checksum Function

### Address
```
0x800209D0
```

### Behavior

The function computes a XOR checksum over a memory range:

```c
uint32 checksum_xor(uint32 *start, uint32 *end)
{
    uint32 checksum = 0;

    while (start < end)
    {
        checksum ^= *start;
        start++;
    }

    return checksum;
}
```

---

## Call Site

```
0x80020C00 → jal 0x800209D0
```

### Register usage

| Register | Meaning |
|----------|--------|
| a0       | start of buffer |
| a1       | end of buffer |
| v0       | checksum result |

---

## Data Flow

1. Save data loaded from memory card
2. Data copied into RAM structures
3. Internal save format is constructed
4. Checksum computed over RAM buffer
5. Result compared with stored checksum

---

## Important Detail

The checksum is computed on:

> The reconstructed RAM save buffer, not the raw memory card file.

---

## Algorithm Properties

- 32-bit word-based XOR
- No CRC or polynomial math
- No lookup tables
- No encryption or obfuscation
- Fully deterministic

---

## Implementation (010 Editor)

```c
uint CalculateXorChecksum(uint start, uint length)
{
    local uint checksum = 0;
    local uint pos;
    local uint value;

    for (pos = start; pos < start + length; pos += 4)
    {
        FSeek(pos);
        value = ReadUInt();
        checksum ^= value;
    }

    return checksum;
}
```

---

## Debugging Notes (PCSX-Redux)

Recommended breakpoints:

- `0x800209D0` → checksum function entry
- `lw` inside loop → data reads
- `xor` instruction → accumulator update
- function return → final checksum in `v0`

---

## Conclusion

The save integrity system in *Chocobo’s Dungeon 2* is a straightforward **32-bit XOR checksum over reconstructed RAM save data**.

Once the correct buffer range is identified, the checksum can be reproduced externally with minimal code.