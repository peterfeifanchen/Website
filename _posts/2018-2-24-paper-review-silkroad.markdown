---
layout: post
title:  "SilkRoad: Layer-4 Load Balancing with ASICs"
date:   2018-2-24 15:40:56 -0800
categories: systems
tags: [ Paper Review ]
---

<span style="color:red">Disclaimer:</span> All images, tables and graphs are sourced from 
the original paper itself and are not my own. The full link to the paper can be found
[here](https://eastzone.bitbucket.io/paper/sigcomm17-silkroad.pdf). Further, this post has
nothing to do with drugs or bitcoins - name collison was by pure coincidence. 

<h1>Problem</h1>

L4 load balancing is traditionally implemented in software. However, these 
software-based solutions are not ideal. They are high latency, 
jittery (50µs to 1 ms - poor performance isolation), and costly 
(3.75% of servers in a data center), due to the high computate overhead incurred 
during load balancing. The paper proposes to accomplish L4 load-balancing using 
programmable ASIC switches sitting at TOR instead, where it can be accomplished with 
two orders of magnitude in power and cost savings. 

In order to enable switches to perform L4 load balancing, two main problems must be 
overcome:

1. Per connection consistency (PCC). Packets belonging to one L4 flow must always be 
mapped to the same DIP.
2. The ConnTable and VIPTable for L4 load balancing (millions of entries) must be able
to fit in SRAM of switches. 

<h1>Background</h1>

For those, like me, that are not versed in the implementation minutiae of load 
balancers, what ConnTable, VIPTable, PCC, and the big deal about programmable ASICs is
all about may be a bit confusing. In short, L4 load balancers are designed to NAT 
virtual IPs (VIP) to a list of destination IPs (DIP). Each VIP represents a particular
service (e.g. data storage), and each DIP represents a physical location that performs
that service. The VIP is probably made avaliable through DNS servers. Incoming traffic
is evenly distributed across DIPs by using a hashing function. 

<figure>
<img src="/assets/Papers/SilkRoad/L4Arch.jpg">
<figcaption>VIPTable and ConnTable for L4 load balancer</figcaption>
</figure>


**VIPTable** contains VIP->DIP mappings, as DIPs change due to network activities such
as upgrades, failures, and additions etc., VIP->DIP mappings change over time. 

**ConnTable** contains the VIP->DIP decisions made by the VIPTable. This enables 
packets belonging to the same flow to be sent to the same DIP, even if VIPTable change.

**Programmable ASICs** such as [Barefoot](https://barefootnetworks.com)
(the author of this paper) are a type of new network ASICs. To understand what is
new, it is important to first understand the general architecture of network 
ASIC. 

<figure>
<img src="/assets/Papers/SilkRoad/matchaction.jpg">
<figcaption>Network ASIC Architecture</figcaption>
</figure>

As shown above, a network ASIC is a multi-stage pipeline. The first stage is 
the parser, which extracts specific range of bits at specific offsets from the 
input packet. This is then passed through a series of match+action tables. The matches
are done in SRAM (exact) or TCAM (wildcards). When a match (e.g., IP address) is found,
the match would contain an action pointer which indexes the desired action in an 
action table to perform (e.g., modify IP address). Overall, there is an ingress pipeline 
(packet in) and an egress pipeline (packet out) which is connected by a queue. Parallel 
pipelines with queued crossbar interconnect can be instantiated for greater throughput. 

This traditional design has a couple of shortcomings. First, the parser is fixed
to a specific parsing tree (e.g., MAC Address -> IP Address -> etc.,). Second, the 
SRAM/TCAM memory is fixed per stage. Third, it is hard to translate network logic 
semantics into match+action bits. Programmable network ASICs aim to solve all that. 
First, the [parser](http://klamath.stanford.edu/~nickm/papers/ancs48-gibb.pdf) is made 
programmable. Multiple parsers are instantiated to provide higher throughput. 
Second, physical match+action tables are replaced with logical match+action tables. 
This allows memory to be allocated depending on a stage's need. Details can be found 
[here](http://yuba.stanford.edu/~grg/docs/sdn-chip-sigcomm-2013.pdf). From my 
understanding (this may be completely wrong), this is achieved through the combination
of enabling the current stage to pick the next stage and the ability to compact/expand
a single logical match+action table across physical tables in multiple stages. Finally, 
in order to have any sanity in programming such ASICs, vendors, such as Barefoot
uses a higher level language (e.g. [P4](https://p4.org/) ) that obsfucates how 
network processing maps to hardware bits.

Finally, although not limited to programmable ASICs, modern network ASICs also support 
transactional memory and ever larger SRAM/TCAM space. With the help of clever techniques
as will be described later, it is now conceivable to store a complete set of the millions 
of L4 enries required for L4 load balancing at point-of-prescence (PoPs/front-end) and
back-end aggregate switches. 

**PCC** is defined on a per-flow basis. As flows are stateful, packets belonging to the same 
flow <u>must</u> be redirected to the same DIP. L4 load-balancers must have this property.
The particular PCC violation caused by ConnTable and VIPTable in the paper is not actually 
made apparent until Section 4.3. The problem here is that, for a particular flow that has been 
assigned to a DIP by current VIPTable<sub>i</sub>, the ConnTable would not be updated with such 
a entry until a later time. This window results in potential PCC violation change may occur 
before all the flows have been updated in the ConnTable. This window of uncertain can result in 
PCC violation if VIPTable was to change to VIPTable<sub>i+1</sub>. Subsequent packets of the 
flow may miss in ConnTable (as update has not reached it yet) and get assigned to another DIP
by the new VIPTable<sub>i+1</sub>. This is traditionally solved in software by locking the 
ConnTable and VIPTable until the load balancing update has propagated. 

Modern switching ASICs supports exact-matching SRAMs which can be used to implement
ConnTables. Insertion is triggered by hardware, upon encountering a new flow. These insertion events
are redirected to the control plane CPU (network OS e.g., Linux). SilkRoad optimizes this upcall by 
using the L2 MAC learning filter to batch a sequence of these new events and remove duplicates. The
CPU simulates cuckoo hashing in order to determine where to place the newly inserted ConnTable entry
in its downcall back into the ASIC. The delay between triggering the insertion and the udpating 
ASIC SRAM (the path is intermediated by CPU) results in PCC violations. There are also transactional
memories which insures statistics updated by a preceding packet are seen by the next packet. 

<h1>Contribution</h1>

1. **PCC** 3-step update process for ensuring PCC during VIPTable updates. Requires: bloom filter 
built using on-chip transaction memory. Modern ASICs have "tharry" which support hardware 
transactional semantics (update to tharry can be immediately seen by subsequent packet).

2. **Table Size** Naively storing L4 entries in match-action tables would take hundreds of
MB of SRAM (~10 million entries). A hash-based solution that reduces match field size with 
mitigation for false positives, allowing millions of L4 entries to fit in current network 
ASIC SRAM (50-100 MB).

<h1>Solution</h1>

<figure>
<img src="/assets/Papers/SilkRoad/newarch.jpg">
<figcaption>SilkRoad Architecture</figcaption>
</figure>

The final solution looks like this.


**Large Table Size**

1. Rather than storing IPv6 5-tuple (37 bytes), store a hash digest (16 bits) in ConnTable. 
This results in 0.01% false positives that is solved by punting it to software. False-positives
are detected when a TCP SYN packet matches in ConnTable. The software, moves the false positive 
match to another stage which uses another hash function. This solution seem a bit unnecessary
since the false positive rate is so low and the downside to false positive matches is a slightly
skewed load balance. Further this solution would not work for UDP and presents no solution when
successive collisions occur. 
2. Rather than storing Ipv6+Port DIPs (16 byte + 2 byte), store a DIP pool version (6bits). 
An extra level of indirection <span class="variable">DIPPoolTable</span> that translations
DIP pool table to a specific DIP.

**PCC**
SilkRoad uses transactional memory to implement a bloomfilter called <span class="variable">
TransitTable</span>. It breaks down a DIP pool update into a 3-step process. 

1. Request - when a DIP pool update is triggered, all connections that miss the ConnTable after
this time is recorded in the TransitTable. This stage finishes when all connTable updates prior
to the DIP pool update request has been sent to the connTable. The paper doesn't detail how we
know that all connTable updates prior to the request have been inserted, but I am guessing this
is a two step process where it first triggers the TransitTable and the first miss from 
TransitTable triggers the LearnTable flush. Then the set that was flushed would consist of all
the entries that need to be inserted into the ConnTable that arrived before the DIP update 
request. TransitTable is write-only in this stage. 
2. Execute - Once all the ConnTable insertion has happened, the DIP update is performed. Any
received packet would then be passed to the TransitTable. If the packet was previously seen,
then it will use the old DIP pool version in DIPPoolTable. If it has not, then it follows the
normal path. This stage ends when all the entries in transit table are inserted in connTable. 
Again, the paper elides how we know that all entries have been inserted. Again I believe there
must be a mechanism to flush the LearnTable once "Execute" finishes. Like the "Request" stage,
the set flushed would then constitute what needs to be inserted. In this stage, TransitTable 
is read-only.
3. Finish - Flows are no longer redirected to the TransitTable and normal operation occurs.

While the paper states that a key insight to be able to use a bloomfilter is that at any
given time there are only two DIP pools in use (old and new) during a transition. This seems
more like an implementation decision in software to not trigger another update while the 
state of the load balancer is between the "request" and "execute" stage. 

The 3-stage process fully ensures PCC as any flows that began before "Request" would have 
been inserted into the ConnTable before the DIP pool update is performed. Any flow arriving
after "Request" and before "Execute" would be in the TransitTable and be redirected to use
the old DIP pool. 

This solution may result in false-positives. Again this can either be ignored because it is
unlikely (SilkRoad uses 256 byte bloomfilter). In this situation, it can be ignored (if you
don't care about a perfect load-balancing) or detected and punted to software like the case
for hash digest false-positives.

<h1>Implementation</h1>

This uses the baseline [switch.P4](https://github.com/p4lang/switch). To this, an extra 
~400 lines of P4 code is added to implement the tables needed for L4 load-balancing.
All tables are implemented as exact-match (with word-packing for memory efficiency) except 
TransitTable which is a bloomfilter on transacitional memory. The software control plane
consists of ~1000 lines of C code performing connTable updates (cuckoo hasing), 3-step PCC 
and DIP pool updates. CPU is connected to the switch via PCI-E.  

<h1>Evaluation</h1>

**Additional Hardware Resources Required**
<figure>
<img src="/assets/Papers/SilkRoad/hardwareoverhead.jpg">
<figcaption>Additional usage above switch.P4 baseline</figcaption>
</figure>

Evaluation was performed by mapping P4 onto a Barefoot switch.

**Memory Savings**
<figure>
<img src="/assets/Papers/SilkRoad/memorysavings.jpg">
<figcaption>Memory savings and performance improvements over software load balancer</figcaption>
</figure>

<u>Left</u>: SilkRoad memory savings across ToR in different clusters (MB). 
<u>Middle</u>: Ratio of number of software load balancers needed to support
						               the same amount of load balancing.
<u>Right</u>: SilkRoad memory savings across ToR in different clusters (%)

Evaluation was done with a simulator using real traffic traces from a large web service 
provider for a ToR switch in each kind of cluster (PoP, Frontend, Backend). The paper did 
not elaborate on how the data was collected or the implementation details of the simulator,
but it sounds like traffic is driven through the actual switch and it seems Barefoot has
statistics collectors for its chips (from their website). The fact that a single switch 
can replace hundreds of software load balancers represents massive power and capital 
savings. 

**PCC**
<figure>
<img src="/assets/Papers/SilkRoad/pccresults.jpg">
<figcaption>Memory savings and performance improvements over software load balancer</figcaption>
</figure>


Evaluation was done comparing Duet (See Related Work), SilkRoad without TransitTable and
SilkRoad with TransitTable. Same simulator was probably used but with one-hour traffic
trace from one PoP cluster. The traffic contains DIP pool updates and traffic, which is
separated and independently generated from the dataset to mimic different combinations.

<h1>Related Work</h1>

Software load balancers lock both the ConnTable and DIPTable during an update to ensure PCC. 
Another system called <span class="variable">Duet</span> offloads the VIPTable into hardware
for low-DIP pool update but high volume VIPs. VIPTable now becomes a fast path, however, 
during a pool update, traffic is redirected into software where a ConnTable is built and 
VIPTable updated. The drawback of this solution is that Duet uses a timer to shift traffic
back to the VIPTable, this causes PCC violations for long running traffic and a wide window
for PCC violation (hence why SilkRoad is orders of magnitude less PCC violations even
without TransitTable). It is also insufficient as the SilkRoad author measured that DIP
pool updates occurs quite often. 

<h1>TidBits</h1>

Other than the technicals of this paper, what is interesting is that this paper provides 
a view into updates to DIP tables and number of active connections across clusters. Here
are some of the figures below.

<figure>
<img src="/assets/Papers/SilkRoad/activeconnections.png">
<figcaption>ConnTable entries per ToR switch (rationale for ~10 million entries)</figcaption>
</figure>

<figure>
<img src="/assets/Papers/SilkRoad/updatespermin.jpg">
<figcaption>DIP pool updates (rationale for frequent updates).</figcaption>
</figure>

Nature of DIP updates (rolling-upgrades) means that batching DIP updates to lower update 
frequency also would not work. Also this means, unlike ECMP at L3, at L4, the entire L4
DIP set could change. This means any clever consistent hashing methods also do not work 
(even if it did, it would skew the load-balancing of flows as L3 only deals with packets). 

<figure>
<img src="/assets/Papers/SilkRoad/newconnpermin.jpg">
<figcaption>New connections per min (no quiet period for DIP pool update without breaking PCC).</figcaption>
</figure>

The number of new connections can be up to 1M per minute. This means that for a MAC learning
filter time-out of 500µs (set by network operator), there will always be around 8 new connections
that have not been inserted yet into ConnTable. 

It is interesting to see all these new features that can be implemented directly in hardware,
especially with the help of the new P4 language. However, it would have helped to know what
percentage of total hardware resources are taken up already by switch.P4, especially when it 
is loaded with entries for its other services (e.g., VTEP stuff, ACL lists). This way we can
actually judge whether it is feasible to burden ToR switches with addtional task of L4 load
balancing. As right now, the table provided only gives incremental usage in additional to 
switch.P4 and not total resource usage. It is also interesting that as long more and more 
tasks can be implemented directly on hardware, switches may now be general purpose or 
specialized middleboxes without changing the underlying hardware, revealing interesting
and vast opportunities for new network orchestration. In one example as the paper 
outlines (see figure below), L4 load balancing can be flexibly aggregated or disaggregated
in the northbound and southbound direction based on need. 

<figure>
<img src="/assets/Papers/SilkRoad/deployment.jpg">
<figcaption>Load balancing can be flexibly moved across switching layers.</figcaption>
</figure>

While today's switches usually contains two supervisors for fault tolerance, they do not
contain two sets of ASICs (at least I don't think so). However if the ASIC can be partially
updated independently, then you can even aggregate or disaggregate or entirely change the
capabilities of a switch live. 
