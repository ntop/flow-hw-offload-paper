### Summary
This repository contains instructions and detailed results for reproducing the results presented in the `Advancements in Traffic Processing Using Programmable Hardware Flow Offload` paper.

### Software
The PF_RING and nProbe Cento packages used can be downloaded from https://packages.ntop.org. 
All the tests have been performed with software versions of May 20th, 2024.

### Testbed
nProbe Cento has been deployed on a Supermicro Super Server/X13SEI-F based on an Intel Xeon Gold 6526Y. 
The traffic generator was running on a Supermicro Super Server/X11SCA-Fbased on an Intel Xeon(R) E-2136 connected with a DAC (Direct Attached Cable).

### Traffic Generation

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 10000000 -U 7 -K 1 -l 60 -O &
done
```

### Flow Processing

The Napatech streams have been configured as follows
```
/opt/napatech3/bin/ntpl -e "Delete = All"
/opt/napatech3/bin/ntpl -e "HashMode = Hash2TupleSorted"
/opt/napatech3/bin/ntpl -e "Assign[StreamId=(0..15)] = port == 1"
```

and cento has been started as follows
```
cento -i nt:stream[0-15] --dpi-level 2 -v 4 -g 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --exporting-cores 31,31,31,31,31,31,31,31,31,31,31,31,31,31,31,31 --max-hash-size 40000000

```
