# iostats

```bash
iostat -dxm 1

# Memory
KEYS0=(Cached Buffers SwapCached)
KEYS1=(MemFree MemAvailable)
KEYS2=(AnonPages AnonHugePages Mapped Shmem
KEYS3=(Slab SReclaimable SUnreclaim)
KEYS4=(Dirty Writeback WritebackTmp Pgpgin Pgpgout PswpIn PswpOut)

KEYS=("${KEYS0[@]}" "${KEYS1[@]}" "${KEYS2[@]}" "${KEYS3[@]}" "${KEYS4[@]}")
PAT=$(printf '%s|' "${KEYS[@]}"); PAT=${PAT%|}

watch -n 1 "grep -E '^($PAT)' /proc/meminfo"
```

```bash
watch -n 1 '
awk "
BEGIN {
    headers = \"Maj Min Dev R_cmp R_mer S_rd TRd_ms W_cmp W_mer S_wr TWr_ms IO_inP Tio_ms WTio_ms\";
    split(headers, H);
    for (i in H) W[i] = length(H[i]);
}
{
    if (NF < length(H)) next;
    for (i = 1; i <= length(H); i++) {
        D[NR, i] = \$i;
        if (length(\$i) > W[i]) W[i] = length(\$i);
    }
}
END {
    for (i = 1; i <= length(H); i++) printf \"%-\" W[i]+1 \"s\", H[i];
    print \"\";
    for (r = 1; r <= NR; r++) {
        if (D[r,1] == \"\") continue;
        for (i = 1; i <= length(H); i++) printf \"%-\" W[i]+1 \"s\", D[r,i];
        print \"\";
    }
}
" /proc/diskstats
'

```

## CPU
```bash
watch -n 1 "
(
  echo 'CPU  user  nice  system  idle  iowait  irq  softirq  steal  guest  guest_nice'
  grep '^cpu' /proc/stat
) | column -t
"
```

## IRQ / SoftIRQ / Scheduler ###

```bash
watch -n 1 "
echo '### /proc/softirqs ###'
cat /proc/softirqs | column -t
echo
echo '### /proc/schedstat ###'
cat /proc/schedstat | column -t
"
```

```bash
watch -n 1 "
echo '### /proc/interrupts ###'
cat /proc/interrupts | column -t
"
```
