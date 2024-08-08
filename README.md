### Summary
This repository contains instructions and detailed results for reproducing the results presented in the `Advancements in Traffic Processing Using Programmable Hardware Flow Offload` paper.

### Software
The PF_RING and nProbe Cento packages used can be downloaded from https://packages.ntop.org. 
All the tests have been performed with software versions of May 20th, 2024.

### Testbed
nProbe Cento has been deployed on a Supermicro Super Server/X13SEI-F based on an Intel Xeon Gold 6526Y with a Napatech NT200A02 adapter.
The traffic generator was running on a Supermicro Super Server/X11SCA-F based on a 6 cores Intel Xeon(R) E-2136 with a Napatech NT100E3 adapter connected with a DAC (Direct Attached Cable) to the server running nProbe Cento.

### Traffic Generation
Traffic was generated with 12 pfsend instances, to take advantage of all physical and logical cores on the traffic generator, with different command line options to control the traffic at various rates. The maximum this machine was able to generate was 89 Mpps @ 60 Gbps (60-byte packets), or 10 Mpps @ 80 Gbps  (970-byte packets).

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

The Napatech streams have been configured with the below NTPL scripts, depending on the setup.

No-offload:
```
Delete = All
HashMode = Hash2TupleSorted
Assign[StreamId=(0..15)] = port == 1
```

Inline configuration with flow offload:
```
Delete = All

// Uplink and Downlink ports
Define UL_PORT = Macro("0")
Define DL_PORT = Macro("1")

// Streams / threads
Define STREAMS = Macro("(0..15)")

Define NUMA    = Macro("0")

Define ColorMiss      = Macro("1")
Define ColorUnhandled = Macro("2")

Define BlackList = Macro("3")
Define WhiteList = Macro("4")

Define isIPv4   = Macro("Layer3Protocol==IPv4")
Define isTcpUdp = Macro("Layer4Protocol==TCP,UDP")

KeyType[Name=KT_4Tuple] = {32, 32, 16, 16}

Define KeyDefProtoSpec = Macro("(Layer3Header[12]/32, Layer3Header[16]/32, Layer4Header[0]/16, Layer4Header[2]/16)")
KeyDef[Name=KD_4Tuple; KeyType=KT_4Tuple; IpProtocolField=Outer; KeySort=Sorted] = KeyDefProtoSpec

HashMode = Hash5TupleSorted

Setup[NUMANode=NUMA] = StreamId == STREAMS

Setup[TxDescriptor=NT; TxPorts=UL_PORT,DL_PORT; TxPortPos=112; TxIgnorePos=95] = StreamId==STREAMS

// New and unhandled flows are sent to the host
Assign[StreamId=STREAMS; Color=ColorUnhandled; Priority=1; Descriptor=DYN1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == UNHANDLED
Assign[StreamId=STREAMS; Color=ColorMiss;      Priority=1; Descriptor=DYN1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == MISS

// Drop blacklisted flows
Assign[StreamId=Drop; Priority=1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == BlackList

// Forward whitelisted flows between ports
Assign[StreamId=Drop; DestinationPort=DL_PORT; Priority=1] = Port==UL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == WhiteList
Assign[StreamId=Drop; DestinationPort=UL_PORT; Priority=1] = Port==DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == WhiteList

// These Assigns set NtStd0Descr_t->txPort
Assign[TxPort=DL_PORT] = Port==UL_PORT
Assign[TxPort=UL_PORT] = Port==DL_PORT
```

Passive capture configuration with flow offload:
```
Delete = All

// Uplink and Downlink ports
Define UL_PORT = Macro("0")
Define DL_PORT = Macro("1")

// Streams / threads
Define STREAMS = Macro("(0..15)")

Define NUMA    = Macro("0")

Define ColorMiss      = Macro("1")
Define ColorUnhandled = Macro("2")

Define BlackList = Macro("3")
Define WhiteList = Macro("4")

Define isIPv4   = Macro("Layer3Protocol==IPv4")
Define isTcpUdp = Macro("Layer4Protocol==TCP,UDP")

KeyType[Name=KT_4Tuple] = {32, 32, 16, 16}

Define KeyDefProtoSpec = Macro("(Layer3Header[12]/32, Layer3Header[16]/32, Layer4Header[0]/16, Layer4Header[2]/16)")
KeyDef[Name=KD_4Tuple; KeyType=KT_4Tuple; IpProtocolField=Outer; KeySort=Sorted] = KeyDefProtoSpec

HashMode = Hash5TupleSorted

Setup[NUMANode=NUMA] = StreamId == STREAMS

// New and unhandled flows are sent to the host
Assign[StreamId=STREAMS; Color=ColorUnhandled; Priority=1; Descriptor=DYN1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == UNHANDLED
Assign[StreamId=STREAMS; Color=ColorMiss;      Priority=1; Descriptor=DYN1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == MISS

// Drop blacklisted flows
Assign[StreamId=Drop; Priority=1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == BlackList

// Forward whitelisted flows between ports
Assign[StreamId=Drop; DestinationPort=DL_PORT; Priority=1] = Port==UL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == WhiteList
Assign[StreamId=Drop; DestinationPort=UL_PORT; Priority=1] = Port==DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == WhiteList

// These Assigns set NtStd0Descr_t->txPort
Assign[TxPort=DL_PORT] = Port==UL_PORT
Assign[TxPort=UL_PORT] = Port==DL_PORT
```

The command to load the ntpl script is the below.

```
/opt/napatech3/bin/ntpl -f script.ntpl
```

Cento has been started as follows.

Passive capture (ndpi enabled)
```
sudo /opt/napatech3/bin/ntpl -f streams.ntpl
sudo cento -i nt:stream[0-15] --dpi-level 2 -v 4 -g 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --exporting-cores 31,31,31,31,31,31,31,31,31,31,31,31,31,31,31,31  --max-hash-size 40000000
```

Passive capture with offload (ndpi enabled)
```
sudo /opt/napatech3/bin/ntpl -f flow-offload-passive.ntpl
sudo cento -i nt:stream[0-15] --dpi-level 2 -v 4 --flow-offload -g 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --exporting-cores 31,31,31,31,31,31,31,31,31,31,31,31,31,31,31,31 --max-hash-size 40000000
```

Inline (ndpi enabled)
```
sudo /opt/napatech3/bin/ntpl -f streams.ntpl
sudo cento-bridge -i nt:stream[0-15],nt:0 --dpi-level 2 --bridge-conf rules.conf -v 4 -g 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --exporting-cores 31,31,31,31,31,31,31,31,31,31,31,31,31,31,31,31 --max-hash-size 40000000
```

Inline with offload (ndpi enabled)
```
sudo /opt/napatech3/bin/ntpl -f flow-offload.ntpl
sudo cento-bridge -i nt:stream[0-15],nt:stream[0-15] --dpi-level 2 --bridge-conf rules.conf -v 4 --tx-offload --flow-offload -g 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --exporting-cores 31,31,31,31,31,31,31,31,31,31,31,31,31,31,31,31 --max-hash-size 40000000
```

Passive capture (no nDPI)
```
sudo /opt/napatech3/bin/ntpl -f streams.ntpl
sudo cento -i nt:stream[0-15] -v 4 -g 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --exporting-cores 31,31,31,31,31,31,31,31,31,31,31,31,31,31,31,31 --max-hash-size 40000000
```

Passive capture with offload (no nDPI)
```
sudo /opt/napatech3/bin/ntpl -f flow-offload-passive.ntpl
sudo cento -i nt:stream[0-15] -v 4 --flow-offload -g 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --exporting-cores 31,31,31,31,31,31,31,31,31,31,31,31,31,31,31,31 --max-hash-size 40000000
```
