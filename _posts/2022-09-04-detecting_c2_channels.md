---
title: "Detecting Command and Control Channels by Analysing Timestamps"
date: 2022-09-04
---

## Introduction

The following github repo includes Prolog PoC attempting to use this method to identify C2 channels. https://github.com/jordanjoewatson/swi2 
Although the code runs successfully, it takes too much time due to iterating over all possible combinations of packets.

## Steps

The following approach was used to collect and prepare data.

1. Use Wireshark to analyse traffic. 
2. Filter data to only include packets with the local hosts src IP and destination IP of the target application.
3. Iterate over consecutive packets and build a list of time differences between each packet.
4. Compute [Entropy](https://brilliant.org/wiki/entropy-information-theory/) of lists.

Milliseconds were removed from timestamps and duplicated rows were removed.

## Results

### Cobalt Strike HTTP Beacon (Sleep 10 99)

Amount of packets: 4820
Entropy value: 0.994
Diagram

### YouTube 

Amount of packets: 32
Entropy value: 0.877
Diagram

### Netflix

Amount of packets: 186
Entropy: 0.783
Diagram

### Reddit

Amount of packets: 558
Entropy: 0.348
Diagram

### RuneScape

Amount of packets: 748
Entropy: 0
Diagram

## Conclusion

High entropy can be an indicator for Command and Control channels. The Prolog code provided is also able to identify malicious C2 traffic masquerading in benign traffic, this is possible because of the Depth First Search algorithm built into Prolog. However, identifying channels using this approach is infeasable due to the time required when searching all possible combination of packets.
