### Summary
This repository contains instructions and detailed results for reproducing the results presented in the `Advancements in Traffic Processing Using Programmable Hardware Flow Offload` paper.

### Software
The PF_RING and nProbe Cento packages used can be downloaded from https://packages.ntop.org. 
All the tests have been performed with software versions of May 20th, 2024.

### Testbed
nProbe Cento has been deployed on a Supermicro Super Server/X13SEI-F based on an Intel Xeon Gold 6526Y with a Napatech NT200A02 adapter.
The traffic generator was running on a Supermicro Super Server/X11SCA-Fbased on a 6 cores Intel Xeon(R) E-2136 with a Napatech NT100E3 adapter connected with a DAC (Direct Attached Cable) to the server running nProbe Cento.

### Traffic Generation
Traffic was generated with 12 pfsend instances, to take advantage to all physical and logical cores on the traffic generator, with different command line options to control the traffic at various rates. The maximum this machine was able to generate was 89 Mpps @ 60 Gbps (60-byte packets), or 10 Mpps @ 80 Gbps  (970-byte packets).

The below command lines have been used. Please note they depend on the CPU, memory and NIC characteristics of the machine running the tool.

1K flows 100 new-fps 10Mpps 80Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 1000 -U 8500 -K 1 -l 970 -O &
done
```

10K flows 1K new-fps 10Mpps 80Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 10000 -U 850 -K 1 -l 970 -O &
done
```

100K flows 10K new-fps 10Mpps 80Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 100000 -U 85 -K 1 -l 970 -O &
done
```

1M flows 100K new-fps 10Mpps 80Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 1000000 -U 9 -K 1 -l 970 -O &
done
```

10M flows 1M new-fps 10Mpps 80Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 10000000 -U 1 -K 10 -l 970 -O &
done
```

1K flows 100 new-fps 89Mpps 60Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 1000 -U 70000 -K 1 -l 60 -O &
done
```

10K flows 1K new-fps 89Mpps 60Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 10000 -U 7000 -K 1 -l 60 -O &
done
```

100K flows 10K new-fps 89Mpps 60Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 100000 -U 700 -K 1 -l 60 -O &
done
```

1M flows 100K new-fps 89Mpps 60Gbps

```
for i in $(seq 1 12);
do
 pfsend -i nt:0 -b 1000000 -U 70 -K 1 -l 60 -O &
done
```

10M flows 1M new-fps 89Mpps 60Gbps

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
