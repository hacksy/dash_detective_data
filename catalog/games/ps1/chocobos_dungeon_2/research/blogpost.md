# Reverse Engineering the Save Checksum in Chocobo’s Dungeon 2 (PS1)

I recently reverse engineered the save file integrity check in *Chocobo’s Dungeon 2* for PlayStation.  
The goal was to understand how the game validates memory card saves and how the checksum is computed.

---

## 🧭 What triggered this investigation

While editing save files manually, I noticed something interesting:

- Small changes sometimes worked
- Other changes caused the save to be rejected or treated as corrupted

This strongly indicated a **checksum or integrity verification system**.

---

## 🔍 Finding the verification logic

Using **PCSX-Redux debugger**, I traced the save loading process.

Eventually, I reached the main save processing routine:

```
0x80020C00
```

This function:
- Loads save data from memory card
- Copies it into RAM structures
- Prepares internal game state
- Performs validation

---

## 🎯 The key discovery: checksum function

Inside this routine, I found a critical call:

```
jal 0x800209D0
```

This function immediately stood out as the checksum routine.

---

## ⚙️ Inside the checksum function

At `0x800209D0`, the logic is extremely simple:

- Read 32-bit values from memory
- XOR them into an accumulator
- Repeat until the end of the buffer

In pseudocode:

```c
uint32 checksum = 0;

while (start < end)
{
    checksum ^= *start;
    start++;
}

return checksum;
```

---

## 📦 Input data

From runtime analysis:

- `a0` = start of buffer
- `a1` = end of buffer

Important detail:

> The checksum is NOT computed directly on the raw memory card file.

Instead, it operates on a **reconstructed RAM buffer after loading and parsing**.

---

## 🧪 Verification

To confirm the behavior, I implemented the same logic in 010 Editor:

- XOR over 32-bit words
- Same range as the game

Result:
✔ Exact match with in-game checksum

---

## 📌 Conclusion

The save protection system in *Chocobo’s Dungeon 2* is:

- A simple 32-bit XOR checksum
- Computed over reconstructed save data in RAM
- Fully deterministic and reversible

There is no CRC, no encryption, and no obfuscation — just a basic integrity check.

---

## 🧠 Final takeaway

Once the correct buffer range is known, the entire save protection can be reproduced externally with a few lines of code.