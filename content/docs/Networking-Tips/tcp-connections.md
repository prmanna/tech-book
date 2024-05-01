---
title: "TCP Connection"
bookCollapseSection: true
weight: 10
---

# Intro

## TCP Connection Establishment Handshake
1. Sync
The initiator that is establishing a connection with a target generates a
random sequence number (5,045 for this example) and sends a TCP packet
with its sync flag set to 1 and its sequenceNumber set to the just-defined sequence
number.
2. Sync/Ack
Upon receipt of the TCP Sync packet from the initiator, the target sets its ack
number value to the received sequenceNumber + 1 (5,046 in this example). The
target responds by setting its own sequence number to a random value
(17,764 in this example) and sending a TCP packet whose sync and ack flags
are both set to 1 and whose sequenceNumber is set to the just-defined sequence
number value and whose ackNumber is set to the target’s just-set ack number.
3. Ack
Upon receipt of the TCP Sync/Ack packet, the initiator sets its ack number to
the received sequenceNumber + 1. The initiator then sends a TCP Ack packet to
the target whose ack flag is set to 1 and whose sequenceNumber and ackNumber
are set to the initiator’s corresponding internal values.
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/tcp-conn/tcp-3way-handshake.png)

## TCP Connection Termination Handshake
The steps in a four-way TCP connection termination handshake are thus.
1. Initiator Finish
One side of the connection (the initiator) sends a TCP packet to its connection
partner whose finish bit is set to 1.
2. Target Ack or Finish/Ack
The target of the initial termination message responds by sending a TCP
packet to the initiator whose ack bit is set to 1. This acknowledges that the
initiator-to-target data connection is closed and that the initiator will no
longer send TCP data packets to the target. The target responds by sending
two TCP packets to the initiator: one whose ack bit is set to 1 and one whose
finish bit is set to 1. If the target has no more data to send to the initiator, it
may combine these two TCP packets into a single TCP packet where both ack
and finish are set to 1. If the target does have more data to send, the connection
may remain “half open” until the data transmissions are complete.
3. Initiator Ack
Finally the initiator sends a TCP packet whose ack bit is set to 1 to complete
the connection termination process.
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/tcp-conn/tcp-conn-termination.png)

## Reliable Data Delivery
For a data connection to be reliable, three types of problems must be detected and
corrected. These are:
    * out-of-order data
    * missing data
    * duplicated data

TCP’s sequence numbers address all three of these problems.

### TCP Sequence Number Behavior
Briefly: The initial sequence number is chosen randomly. As the TCP sender
transmits bytes to the receiver, each TCP packet includes a sequenceNumber value
that indicates the cumulative sequence number of the last byte of each packet. The
TCP receiver responds by transmitting TCP Ack packets whose ackNumber value
identifies the highest-numbered byte received without any gaps from the first byte
of the TCP session. Sequence numbers roll over through zero upon exceeding the
maximum value that a 32-bit number can hold.

### TCP Data Reorder
You can see the presence of sequenceNumber in each packet means that a TCP
receiver can position each received packet within a TCP reorder buffer where
the original sequence of data bytes may be reassembled. Let’s consider a simple
scenario where the initial sequence number is 0. If the first packet received by the
TCP receiver has a sequenceNumber value of 1,000 and the packet has a TCP data
payload of 1,000 bytes, then this packet neatly fits within the buffer at offset 0.
The next packet received by the TCP receiver has a sequenceNumber value of 3,000
and carries 1,000 bytes of TCP data. The receiver subtracts the data byte count
from the received sequenceNumber value to arrive at a sequence number of 2,000
for this packet’s first data byte. Thus, the data is written to the buffer starting at
offset 2,000, leaving a 1,000-byte gap between the first and second received packets.
Finally, a TCP packet with 1,000 data bytes and a sequenceNumber of 2,000
is received, meaning the sequence number of the first byte of this packet is 1,000.
This third packet’s data is written into the 1,000-byte gap between the first two
packets’ data, completing the in-order data transfer.
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/tcp-conn/tcp-data-reorder.png)

### Missing Data Retransmission
Using the data reordering example from above as a starting point, let’s assume that
the TCP packet whose sequenceNumber value is 2,000 wasn’t simply delivered out
of order, but was lost due to any of a variety of the usual things that happen on
networks. From the TCP receiver’s perspective, the order in which the TCP data packets are received and its responses to the data packets are the same regardless
of whether a packet was lost in transmission (and later retransmitted) or simply
delivered out of order by the network.
In response to the receipt of the first TCP data packet, the TCP receiver transmits
a TCP Ack packet back to the sender with an ackNumber value of 1,000, indicating
that is has successfully received bytes 0 through 999. The second TCP data packet
received by the TCP receiver indicates that a gap exists from byte 1,000 through
1,999. To inform the TCP sender of this, the TCP receiver transmits another TCP
Ack packet with the exact same ackNumber value of 1,000. This informs the TCP
sender that, while the first 1,000 bytes have been successfully received, the next
packet (and possibly more) is currently missing. This duplicate acknowledge (the
sender has already received an acknowledge for the first 1,000 bytes) motivates
the sender to retransmit the first unacknowledged packet (i.e., the packet whose
sequenceNumber is 2,000).
The TCP sender, in turn, keeps track of the TCP Ack messages that have been received
and maintains an awareness of the highest-numbered data byte that’s been
successfully transmitted. Every unacknowledged byte is subject to retransmission.
Re-transmissions and acknowledges may also get lost. Hence, TCP maintains timers
to detect these cases and automatically retransmit either data or Ack packets as
necessary to keep the process moving along. 

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/tcp-conn/tcp-missing-data-behavior.png)

### Duplicate Data Discard
Duplicate data may be received by a TCP receiver if a packet that is assumed missing
is simply delayed and delivered out of order. The TCP receiver may request another
copy of the packet through the use of a TCP Ack message before the delayed
data packet is received. If both copies of the data packet are eventually successfully
received, then the receiver must deal with redundant data.
If a TCP receiver receives a TCP data packet whose sequenceNumber corresponds
to a data packet that had previously been received (regardless of whether or not it
had been acknowledged), the TCP receiver may either discard the packet or simply
overwrite the data in its receive buffer with the newly received data. Since the data
is the same, the data in the buffer does not change.

**References:**
* 
