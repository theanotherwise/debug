## iostats

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
  echo 'Load1 Load5 Load15 Running/Total PID'
  awk '{print \$1, \$2, \$3, \$4, \$5}' /proc/loadavg
) | column -t
"

```

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

## Interfaces

```bash
watch -n 1 '
awk "
BEGIN {
    headers = \"Iface RX_B RX_Pk RX_E RX_D RX_F RX_Fr RX_C RX_M TX_B TX_Pk TX_E TX_D TX_F TX_Co TX_Ca TX_Cm\";
    split(headers, H);
    for (i in H) W[i] = length(H[i]);
}
NR > 2 {
    gsub(\":\", \"\", \$1);
    if (NF < length(H)) next;
    for (i = 1; i <= length(H); i++) {
        D[NR, i] = \$i;
        if (length(\$i) > W[i]) W[i] = length(\$i);
    }
}
END {
    for (i = 1; i <= length(H); i++)
        printf \"%-\" W[i]+1 \"s\", H[i];
    print \"\";
    for (r = 3; r <= NR; r++) {
        for (i = 1; i <= length(H); i++)
            printf \"%-\" W[i]+1 \"s\", D[r, i];
        print \"\";
    }
}
" /proc/net/dev
'
```

```bash
watch -n 1 "
(
  awk '
    /^Tcp:/ {
      sub(/^Tcp:[[:space:]]*/, \"\")
      if (++n == 1) {
        hdr = \$0
      } else if (n == 2) {
        print hdr
        print \$0
        exit
      }
    }
  ' /proc/net/snmp
) | column -t
"

```

## Processes

```bash
ps -o pid,ppid,nlwp,cmd,%cpu,%mem,stat,start,time -p 4364 
```

```bash
watch -n 1 '
(
  echo USER PID PPID NLWP %CPU %MEM RSS VSZ STAT COMM OFILES
  for pid in $(ps -eo pid --sort=-%cpu | head -n 21 | tail -n +2); do
    ps -p $pid -o user=,pid=,ppid=,nlwp=,%cpu=,%mem=,rss=,vsz=,stat=,comm= 2>/dev/null | \
    while read u p pp nl c m r v s n; do
      of=$(ls /proc/$p/fd 2>/dev/null | wc -l)
      printf "%s %s %s %s %s %s %s %s %s %s %s\n" "$u" "$p" "$pp" "$nl" "$c" "$m" "$r" "$v" "$s" "$n" "$of"
    done
  done
) | column -t'
```

```bash
PROC="nginx"

watch -n 1 "
(
  echo USER PID PPID NLWP %CPU %MEM RSS VSZ STAT COMM OFILES
  for pid in \$(ps -eo pid,comm --no-headers | awk -v p=\"$PROC\" '\$2 == p {print \$1}'); do
    ps -p \$pid -o user=,pid=,ppid=,nlwp=,%cpu=,%mem=,rss=,vsz=,stat=,comm= 2>/dev/null | \
    while read u p pp nl c m r v s n; do
      of=\$(ls /proc/\$p/fd 2>/dev/null | wc -l)
      printf \"%s %s %s %s %s %s %s %s %s %s %s\n\" \"\$u\" \"\$p\" \"\$pp\" \"\$nl\" \"\$c\" \"\$m\" \"\$r\" \"\$v\" \"\$s\" \"\$n\" \"\$of\"
    done
  done
) | column -t"
```
