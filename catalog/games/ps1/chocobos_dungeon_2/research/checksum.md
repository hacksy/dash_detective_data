uint32_t checksum = 0;

for (uint32_t *p = (uint32_t *)(save + 0x20);
     p < (uint32_t *)(save + 0x1368);
     p++)
{
    checksum ^= *p;
}