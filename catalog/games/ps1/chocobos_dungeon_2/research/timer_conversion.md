string ShowTimer(uint32 v)
{
    local uint32 total;
    local uint32 minutes;
    local uint32 seconds;
    local uint32 hours;

    /* convert ticks → seconds */
    total = v / 32;

    hours = total / 3600;
    minutes = (total / 60) % 60;
    seconds = total % 60;

    return Str("%02d''%02d'%02d",
        hours,
        minutes,
        seconds);
}