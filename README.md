# debug

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

# CPU
```bash
watch -n 1 "(echo 'CPU  user  nice  system  idle  iowait  irq  softirq  steal  guest  guest_nice'; grep '^cpu' /proc/stat) | column -t"
```
