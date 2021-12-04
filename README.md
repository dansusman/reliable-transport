# Reliable Transport Protocol

This repo holds the source code for Project 4 of Networks and Distributed Systems course at Northeastern University. We were tasked to implemented something similar to TCP, i.e. build some reliability guarantees on top of UDP.

Authors: John Henry Rudden, Daniel Susman

## High Level Approach

This code module comprises two major components: 3700recv and 3700send (sender and receiver applications, respectively). These apps talk to each other sending data back and forth, attempting to manage communication reliably and in-order. In this way, our code mimics TCP. After some experimentation with TCP Reno-like functionality, we chose a Sliding Window approach to Reliable Transport that doesn't check for three duplicate ACK's.

### The Sender
Our sender utilizes a `SEQ_BUFFER` object, which is a dictionary from sequence number of a packet to a `SeqInfo` object that holds all the information about that packet that we may need in the program's execution. We delete entries in `SEQ_BUFFER` as we receive correct ACK's for specific packets, and add to it as we send things out. Our Sliding Window is set to 20, which we landed on after some experimentation with larger and smaller values. This means we send out in batches of 20 if possible, and respond to dropped packets/disturbed ACK's from there.

Using a sliding window approach increases the efficiency of our code's execution and allows us to handle a lot of dropped packets, high latency, and large payloads. At the time of writing, all tests are passing, including performance tests, since we opted to optimize time/space complexity passed our original batch approach.

### The Receiver
Our receiver utilizes a `DATA_BUFFER` object, which holds any buffered information that we may need later in the execution of the program. Whenever we receive **in order** packets, we add to the buffer and send an ACK back to the sender. We sort the `OUT_OF_ORDER` list of packets and loop through to see if we can increase our `NEXT_IN_ORDER` packet (the next packet our receiver expects to see). This happens in the `find_next_ack()` function. When we receive an **out of order** packet, we add to the `OUT_OF_ORDER` list of packets and send an ACK to the sender.

We use a SACK approach to inform the sender of which packet is expected next in order. For instance, if sequence number 34000 just came in, if we need to append a SACK to the ACK, we would append 35000 (assuming `DATA_SIZE = 1000`). 

## Challenges Faced

Throughout this project, we ran into a few problems, as outlined below:
1. Our initial approach with a congestion window and sending "batches" of packets at a time did not perform as well as we wanted. Since we didn't have to worry about fairness in the network, we shifted to an alternate, faster implementation that utilizes a Sliding Window. Both of these approaches came with their fair share of challenges and tiny bugs that were tricky to catch.
2. To properly implement SACKs, we needed a way to ensure our `NEXT_IN_ORDER` sequence number was always the sequence number of the correct packet. After running through a few approaches and printing multiple pieces of information, we landed on the approach of saving out of order packets in a sorted list and updating `NEXT_IN_ORDER` accordingly as we received packets from the sender.

## Testing Our Code

To test our code, we made use of netsim, nettest, and testall religiously. Once we had our sender and receiver talking to each other, we started start and worked up to more complex examples. We also made **heavy** usage of the provided log() function in both 3700send and 3700recv. This allowed us to see the state of very aspects of the code at any point we desired. This was integral in solving some of our smaller, more annoying bugs.

As a final sanity/litmus test, we ran testall 15 times in succession to ensure we were passing every test regularly. This increased our confidence in our code's robustness and performance.
