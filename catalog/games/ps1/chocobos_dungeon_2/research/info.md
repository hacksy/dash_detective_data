Chocobo’s Dungeon 2 – Save Checksum Reverse Engineering
Overview

The PlayStation version of Chocobo’s Dungeon 2 uses a simple save integrity check based on a 32-bit XOR checksum.
No CRC, no encryption, no tables.

Checksum Type
Type: 32-bit XOR accumulation
Method: word-wise XOR over save buffer
Complexity: O(n)
No transforms (no shifts, no rotates, no polynomial math)
Core Checksum Function

Address:
0x800209D0

Pseudocode
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
Function Call Context

Caller:
0x80020C00

Call:
jal 0x800209D0

Register Usage

a0 = start of RAM save buffer
a1 = end of RAM save buffer
v0 = returned checksum

Data Flow
Save is loaded from memory card
Data is copied into RAM buffer
Save structure is reconstructed
XOR checksum is computed over RAM data
Result is compared with stored checksum
Included in Checksum
32-bit aligned save data
Final reconstructed RAM block
Excluded
checksum field itself
temporary loading buffers outside final range
Key Observations
Loop uses lw + xor only
No CRC or hashing complexity
Fully deterministic
Very fast verification
Debugging (PCSX-Redux)

Breakpoints:

0x800209D0 → checksum function
lw inside loop → data fetch
xor instruction → accumulator update
return → final checksum in v0

Watch:

a0 / a1 range
running XOR value
010 Editor Implementation

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