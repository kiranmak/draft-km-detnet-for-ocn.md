---
abbrev: ocn-in-detnets
docname: draft-km-detnet-for-ocn-latest
title: Using Deterministic Networks for Industrial Operations and Control
date:  false
category: info
stream: independent

ipr: trust200902
area: "Internet"
workgroup: "Detnet Group"
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
-

    ins: K. Makhijani
    name: Kiran Makhijani
    organization: Futurewei
    email: kiran.ietf@gmail.com
-
    ins: R. Li
    name: Richard Li
    org: Futurewei
    email: richard.li@futurewei.com
-
    ins: C. Westphal
    name: Cedric Westphal
    org: Futurewei
    email: cedric.westphal@futurewei.com
-
    ins: L. Contreras
    name: Luis M. Contreras
    org: Telefonica
    email: luismiguel.contrerasmurillo@telefonica.com
-
    ins: T. Faisal
    name:  Tooba Faisal
    organization: King's College London
    email: tooba.hashmi@gmail.com

informative:
  FACTORY:   I-D.wmdf-ocn-use-cases
  VIRT-PLC:  I-D.km-iotops-iiot-frwk
  PCE-PAM:   I-D.contreras-pce-pam
  PTP-GRID: DOI.10.1109/IEEESTD.2016.7479438
  NIST-OT:  DOI.10.6028/NIST.SP.800-37r2
  TSN_IA_PROFILE:
    target: https://1.ieee802.org/tsn/iec-ieee-60802/
    title: IEC/IEEE 60802 TSN Profile for Industrial Automation

--- abstract

This document describes an application programming interface with a
deterministic network domain. These APIs are used to select (or map) services in
the Deterministic Network architecture.
--- middle

# Introduction {#intro}

Process automation systems involve operating equipment (such as actuating
and/or sensing field devices). The communication between the 'process
controllers' and field devices exhibit a well-defined set of behaviors and has
specific characteristics: delivering a control-command to a machine must be
executed within the time frame specified by a controller or an application to
provide reliable and secure operation. A low or zero tolerance to latency and
packet losses (among other things) is implied.

The endpoints ('process controllers' and field devices) embody
machine-to-machine communications to facilitate remote and local process
automation. Applications using deterministic networks are not aware of the
details of such networks and, therefore, require an interface with DetNets for
operations and control of field-devices. In this document such interfaces are
referred to as Operation and Control Network (OCN) interfaces for convenience.
OCN interfaces are used by applications to describe requirements for for
guaranteed delay-aware packet delivery, reliability, and packet loss mitigation.

Since Deterministic Networks provide extension to Time Sensitive Networks (TSN)
for large-scale domains, it is reasonable to utilize the
definitions developed by TSN for Industrial Automation (IA) traffic profile
{{TSN_IA_PROFILE}}. The IA endpoints will use this profile to access
appropriate service and treatment in a DetNet domain.

This document provides a background discussion on the endpoints and
connectivity in Industrial automation systems  ({{background}}),
IA traffic type APIs to be used at IP layer ({{ttypes}}). These APIs are
implementation and encapsulation agnostic. Applications may choose, either
in-band metadata method or P4 base datapath programmability. As an example,
IPv6 extension header based approach is covered in {{ocn-eh}}.


# Terminology

- Operational Technology (OT):
 : Programmable systems or devices that interact
with the physical environment (or manage devices that interact with the physical
environment). These systems/devices detect or cause a direct change through the
monitoring and/or control of devices, processes, and events. Examples include
industrial control systems, building management systems, fire control systems,
and physical access control mechanisms. Source: {{NIST-OT}}

- Industrial controller or process controller:
  : Is a logic control function used in process automation and control systems.
A process controller maintains the operational requirement of a process and
performs functions similar to programmable logic controllers (PLCs) but it can
be either a hardware or software component. The term process controller is used
through out to avoid confusion with 'network controllers' used in network
infrastructures.

- Industrial Automation:
  : Mechanisms that enable machine-to-machine communication by use of
technologies that enable automatic control and operation of industrial devices
and processes leading to minimizing human intervention.

- Control Loop:
 : Control loops are part of process control systems in which
desired process response is provided as input to the 'process controller', which
performs the corresponding action (using actuators) and reads the output values.
Since no error correction is performed, these are called open control loops.

- Feedback Control Loop:
  : A feedback loop is part of a system in which some portion (or all) of
the system's output is used as input for future operations.

- Industrial Control Networks:
 : Industrial control networks are the
interconnection of equipment used for the operation, control, or monitoring of
machines in the industrial environment. It involves a different level of
communication - between fieldbus devices, digital controllers, and software
applications.

- Human Machine Interface (HMI):
: An interface between the operator and the machine.
The communication interface relays I/O data back and forth between an operator's
terminal and HMI software to control and monitor equipment.

## Acronyms

- HMI: Human Machine Interface
- OCN: Operations and Control Networks
- PLC: Programmable Logic Control
- OT: Operational Technology
- OC: Operation and Control
- OCN: Operation and Control Networks

# Background on Industrial Control Systems {#background}

An industrial control network interconnects devices used to operate, control and
monitor physical equipment in industrial environments. {{icn-arch}} below shows
such systems' reference model and functional components. Closest to the
physical equipment are field devices (actuators and sensors) that connect to
the Programmable Logic Controllers (PLCs) or other types of controllers (Note:
in this memo term 'process controller' will be used to differentiate
from other meanings of controller) using serial bus technologies (and now
Ethernet).  Above those 'process controllers' are Human Machine Interface (HMI)
connecting different PLCs and performing several controller functions along
with exchanging data with the applications.

A factory floor is divided into cell sites. The PLCs or other types of
controllers are physically located close to the equipment in the cell sites.
Monitoring, status, and sensing data are collected on the site
and then transmitted over secure channels to the data applications for
aggregation and further processing. These applications can be hosted
in remote cloud infrastructure but are often hosted within a
limited domain environment, controlled by a single operator, like
on-premise, at the edge, or in a private cloud. Both options gain
from infrastructure that scales out and has elastic computing and storage
resources so they will be referred to as cloud in the following sections.

~~~~~drawing

        +-+-+-+-+-+-+
     ^  | Data Apps |....            External business-logic
     :  +-+-+-+-+-+-+   :                Network
     :        |         :
     v  +-+-+-+-+-+-+  +-+-+-+-+--+
        | vendor A  |  |vendor B  |  Interconnection of
        | controller|  |controller|  controllers
     ^  +-+-+-+-+-+-+  +-+-+-+-+-+   (system integrators)
     :       |         |
     :   +-+-+-+-+  +-+-++-+
     :   | Net X |  | Net Y|
     v   | PLCs  |  | PLCs |--+    device-controllers
     ^   +-+-+-+-+  +-+-+--+  |
     :      |        |        |
     :   +-+-+    +-+-+    +-+-+
     v   |   |    |   |    |   |   Field devices
         +-+-+    +-+-+    +-+-+
~~~~~
{: #icn-arch title="Functions in Industrial Control Networks"}

Data applications can integrate softwarized process control
functions to improve automation and make programmatic real-time decisions. The
equipment control and collection of data generated by the sensors should be
possible over small or large-scale deterministic networks as illustrated in
{{new-arch}}.

~~~~~drawing

               +-+-+-+-+-+-+-+-+
               |     Data Apps |      Integrated Apps with
               | c1 | c2  | c3 |      Remote process control
               +-+-+-+-+-+-+-+-+
                \   ,-----.   /
                 +-[  Det- ]-+
                   [Network]
                    `-----'
               +-+-+-|  |-+-+-+-+
               |        |       |
             +-+-+    +-+-+   +-+-+
             |   |    |   |   |   |   Field devices
             +-+-+    +-+-+   +-+-+
~~~~~
{: #new-arch title="Converged Cloud based Industrial Control Networks"}

One particular motivation is to provide the behavior of a field bus between the
cloud and the actuators/sensors. i.e., with the same assurance of reliability
and latency, albeit over wide area networks (WAN). Many industrial control
applications, such as factory automation {{FACTORY}}, PLC virtualization
{{VIRT-PLC}}, power grid operations {{PTP-GRID}}, etc.,  are now expected to
operate in the cloud by leveraging virtualization and shared infrastructure
wherever possible.

## Connected Process-Controllers, Sensors and Actuators

{::comment}
## Reference Points for Connecting Controllers, Sensors and Actuators
{:/comment}

Control systems comprise 'process controllers', Sensors and Actuators. The data
traffic essentially carries instructions that cause machines or equipment
to move and do things within or at a specific time. The connectivity exists in
the following manner:

- A 'process controller' interfaces with the sensors and actuators. It knows an
application's performance parameters which are expressed in terms of network
specific requests or resources such as tolerance to packet loss, latency limits,
jitter variance, bandwidth, and specification for safety.  The 'process controller' knows
all the packet delivery constraints.

- An actuator receives specific commands from the 'process controllers'. The
  Deterministic network between them should support control of actuating
devices remotely from the 'process controller' while meeting all the
requirements (or key performance indicators - KPIs) necessary for successful
command execution. The actuator participates in a closed control loop as needed.

- A sensor emits periodic sensor data. It may intermittently provide
asynchronous readings upon request from the 'process controller'. Sensors may report
urgent messages regarding malfunctioning in certain equipment, cell sites, or
zones.

In many control systems, there is at least one 'process controller' (or server) entity on
one end and two other entities - the sensors and actuators on the other end.
The communication with sensors and actuators is through a 'process controller' application;
as such data applications do not directly interact with the field devices.
Neither actuators nor sensors perform decision-making tasks. This
responsibility belongs to the 'process controller'.

## Generalized Communication Model

To describe networked process control behavior, a conceptual communication model
is used so that the data applications do not concern with the details of the
networks realizing operations and control. We refer to this model as an operation
and control network (OCN) model, with the following components:

- Logical reference points: identify an endpoint's role or function as
  sensor-point, actuation-point, or operation & control point (oc-point for
short). Note: the term 'oc-point' is used to avoid confusion with the network
controllers and the term 'fd-point' is used when both types of field devices are
referred to.

- Interface specification: in terms of associated traffic patterns between the
  endpoints as described below in {{ocn-pattern}}. The interface may be any
type of network (Ethernet, IP, wireless, etc. The model assumes that the
network is capable of providing network services and resources necessary of the
application specific operations and control.

Depending on the design of the usecase, the 'process controller' functionality
(oc-point) may reside as a software module in the data application or as a
separate module. When deployed as a separate module, another connectivity
the interface between the data application and oc-point will be needed and is out
of the scope of this document.

The applications will use a communication interface between oc-point and
sensor-point to receive sensory data and similarly interface between oc-point
to actuation point to execute a single or a sequence of control instructions.

This abstraction provides an additional layer of  protection in the sense that
the traffic patterns between the reference points are well defined so any
exceptions can be easily caught.

## Traffic Patterns {#ocn-pattern}

For either local or wide areas, the process automation activities over the
network can generate a variety of traffic patterns between the oc-point and
field devices such as:

### Control Loops {#c-loop}

The equipment being operated upon is sensitive to when a command request
actually executes. An actuator, upon receiving a command (say a function code) will
immediately perform the corresponding action.
{::comment}
It is the responsibility of network
and 'process controller' to ensure that behavior of the sensor and actuator follows the
expectations of applications.
{:/comment}

For several such applications, the knowledge of a successful operation is equally
critical to advance to the next steps; therefore, getting the response back in
a specified time is required, leading to a knowledge of timing. These types of
bounded-time request and response mechanisms are called control loops.

Unlike general-purpose applications, commands cannot be batched; the
parameters of the command that will follow depends on the result of the previous one.
Each request in the control loop takes up a minimal payload size (function code,
value, device or bus address) and will often fit in a single short packet.

In Detnet-enabled network, it can be imagined as a small series of packets with
the same flow identifier, but with different latency constraints.

It is required to support control loops where each request presents its own
latency constraints to the network and where commands are small sized packets.

###  Periodicity {#ocn-intervals}

Sensors emit data at regular intervals; i.e., there may be more tolerance to
variations in jitter between the measurement intervals. Usually, 'process
controllers' or applications listening to sensor data are programmed to
tolerate and record intermittent losses or delay variations upto certain number
of times. Therefore, time criticality is not always high.

Notably, industrial software now increasingly rely on sensor data collection to
monitor the state and behavior of the entire shop floor.  Thus, the number of
sensors are growing and the combined traffic volume generated by sensors is
expected to be very high. In fact will contribute to a large percentage of ocn traffic.
Moreover, the periodicity of each sensor will also vary based
on the equipment.

It is required that network capacity is planned appropriately for the periodic
traffic generated from the different sensors. The periodic interval should also
be preserved in the network because any variations could provide false
indications that the equipment is misbehaving.

### Ordering

In real-time process control communications, processing of out-of-order related
messages will lead to costly operations failures.  For example, messages
such as request and reply, or a sequence of commands to different endpoints may
be related in the application work flow, therefore, both time constraints and order must be preserved.

The network should be capable of supporting sporadic on-demand short-term flows.
This does not imply instantaneous resource provisioning, instead it would be
more efficient if the provisioned resources could be shared for such
asynchronous traffic patterns.

Another consideration with ordering is that both actuators and sensors are
low-resource devices.  They can not buffer multiple packets and execute them in
order while maintaining the latency bounds of each command execution.  This
means the network must pace packets that may arrive early.


### Urgency

Besides latency constrained and periodic messages, sensors also report failures
as fault notifications, such as pressure valve failure, abnormally high
humidity, etc. These messages must be delivered immediately and with the utmost urgency.

### Co-existence with non-deterministic flows

A DetNet-enabled network could support traffic other than
deterministic flows. Thus a graceful co-existence of any type of flows
can be expected. Notably, the characteristics of the deterministic
flows can influence or impact on the non-deterministic flows, as well
as the decisions taken by the control loops.


## Communication Patterns

Control systems follow a specific communication discipline. The field devices
(sensors and actuators) are always controlled, i.e., interact with the system
through 'process controllers' in the following manner:-

- Sensor to 'process controller': data emitted at periodic intervals providing
  status/health of the environment or equipment. The  traffic volume for this
communication is determined by the payload size of each  sensor data and the
interval. These are a kind of synchronous Detnet flows but with much higher time intervals; still the inter-packet gap should be minimal.

- Process controller to/from actuator: the commands/instructions to write or read.
  Actuators generally do not initiate a command unless requested by the
'process controller'. Actuators will often execute a command, read the corresponding
result, and send that in response to the original write command.  The traffic
profile will be balanced in both directions due to requests/ response behavior. These are like asynchronous flows but without the observation interval constraint.


## Industrial Control Application Interfaces to DetNets

Current industrial automation solutions utilize a split approach.
Industrial-controllers are placed close to the equipment to achieve
operational accuracy, whereas actual process instructions are received
through other means possibly involving human interface. Similarly,
sensor data is first acquired on-site then transmitted in bulk to the
enterprise cloud or remote site for further processing. Such
approaches lead to increase in IT infrastructure costs on the shop
floors.

This document assumes that the
deterministic networks are deployed between enterprise sites and shop
floors. They have resources available to provide latency guarantees,
reliability, and link capacity over known physical distances. Thus,
they have the capabilites to deliver process control and operation
instructions remotely from an application to field devices over larger
distances or the Wide Area Networks (WAN) thereby reducing the need
for IT infrastructure on shop floors.

Moreover, it may even be the case in which non-deterministic domains are
traversed by industrial automation applications. In such situations,
mechanisms will be in place to provide guarantees in terms of the
Service Level Objectives (SLOs) defined for the deterministic flows.
Means for that could the use of IETF Network Slice services
{{?RFC9543}}, and/or the selection of paths characterized by precision
metrics {{PCE-PAM}} and {{!RFC9544}}.

The OCN APIs are defined to  leverage such capabilites from diverse variety of
networks in uniform manner. These APIS are discussed next.

# Operation & Control Traffic Types {#ttypes}

## Overview

{{fig:detnet-ind}}, shows application interface to DetNet domain. Note that the
interface or APIs are needed for DetNet-unaware endpoints. The PC-App and FD-GW
below do not carry DetNet encapsulations, instead they interact using OCN APIs with
traffic type information with the DetNet  edges to be mapped to DetNet flows.


TODO: Synchronization aspects, sync clock details for time-triggered operations.

~~~~~
DetNet
End System
   _
 / PC\     +-----+      +-----------+            DetNet
| App |<-->|MGMT |<====>|DETNET-CTRL|          End System
/-----\    +-----+      +---+-------+          +------+
| NIC |                /   |       \           |FD-GW |
+--+--+ De|tNet       /    |        \          +----+-+
   |    UN|I   +----+    +----+       +----+ DetNet |
   |      v    |    |    |    |-+     | PE |  UNI(U)|
   +-----------U PE +----+ P  | |     |    U--------+
               |    |    |    | |-----|    |
               +----+    +--+-+ |     +----+
                            +---+
             |<------DetNet ----------->|

    PC APP: Process Controller Application
    FD-GW:  Field device gateway
    NSP entity: Network service provider controller
                e,g, DetNet Controller
~~~~~
{: #fig:detnet-ind title="A Realistic DetNet Based Industrial Application Network"}


## OCN Traffic Type Equivalence {#ttypes-equ}

The table below is equivalent to TSN IA traffic profile; however, these
are defined as traffic-type code, and the parameters that applications would
use it to interface with the DetNet.

|---
| Traffic Type | TT-Code | Param | Param | Description |
|---
|Isochronous| ISOC_TT| 0x08| DL_TIME<br> DL_UNIT |Deadline limit between Src and Dst <br>Optional clock src info|
| Cyclic-<br>Synchronous | CSYNC_TT |0x07 |DL_TIME <br>DL_UNIT |-same- |
| Cyclic-<br>Asynchronous | CASYN_TT | 0x06 | DL_TIME <br>DL_UNIT | -same- <br> No clock source needed|
| Network Control | NWCTL_TT | 0x05 |-as above-| |
| Alarms and Event | ALEV_TT | 0x04 | DL_TIME <br> RETRANS | Retransmission flag|
| Conf. Diag | CFDG_TT | 0x03 |||
| Best Effort<br> High | BEHI_TT | 0x02 |||
| Best Effort<br> Low | BELO_TT | 0x01 |||

 In the following sections, a brief definition and corresponding metadata is explained which is similar to the description in {{TSN_IA_PROFILE}}.

 In some cases, endpoints may require details of the synchronization clock which maybe managed separately but the DetNet would need to know the source for synchronization.

### Isochronous traffic-type

These are the most time-sensitive cyclic traffic. This type of traffic has specific latency requirements. This type of traffic is normally used in control loop tasks. The applications specifies:

- cycle times in the range of upto tens of milliseconds. Maybe even microseconds.
- Synchronized common clock source identifier: optional.
- Network must be engineered to offer zero-congestion

API format:

~~~~
   +-- tt_code = ISOC_TT
   +-- dl_time = value
   +-- dl_tmunite = ms |us
   +-- app-flow-ref
   +-- clock-src : ip address
~~~~

### Cyclic-synchronous traffic-type
This type of traffic is also transmitted cyclically with latency requirements.
These can be specified as a flow (or stream in TSN)

- Cycle times in the range of hundreds of microseconds to hundreds of milliseconds.
- Synchronized common clock source identifier: optional.
- Network must be engineered to offer zero-congestion

API format

~~~~
     +-- tt_code = CSYNC_TT
     +-- dl_time = value
     +-- dl_tmunit = ms |us
     +-- app-flow-ref
     +-- clock-src : ip address
~~~~

### Cyclic-Asynchronous traffic-type

This type of traffic is transmitted acyclically and bounded by the application clock.
These can be specified as a flow (or stream in TSN)

- Cycle times are in the range of milliseconds to seconds.
- tolerates congestion loss to specified limit (number of packets lost per unit of time).

API format

~~~~
     +-- tt_code = CSYNC_TT
     +-- dl_time = value
     +-- dl_tmunit = ms |sec
     +-- app-flow-ref
~~~~

### Alarms and Events traffic type

This type of traffic is acyclic. This traffic expects bounded latency and should follow bandwidth constraints.

- Bounded latencies are in the range of milliseconds to hundreds of milliseconds.
- Allocated bandwidth limits.
- Support for retransmission in response to packet loss is expected.

Networks should be engineered to handle bursts of frames, up to a provisioned number of frames.

API format

~~~~
     +-- tt_code = ALEV_TT
     +-- dl_time = value
     +-- dl_tmunit = ms |sec
     +-- restrans = yes |no
~~~~

### Configuration and diagnostics traffic type

Traffic profile is same as above.

API format

~~~~
     +-- tt_code = CFDI_TT
     +-- dl_time = value
     +-- dl_tmunit = sec
     +-- restrans = yes |no
~~~~

It is expected that application controller endpoint will periodically transmit diagnostics packets and field device configurations.

### Network Control

Network control traffic will be generated by application controller and is comprised of services required to maintain network operation such as time synchronization, loop prevention, and topology detection.

API format:

~~~~
     +-- tt_code = CFDI_TT
     +-- dl_time = value
     +-- dl_tmunit = sec
     +-- restrans = yes |no
~~~~

It is expected that the controller endpoint can transmit network control packets.

# IANA Considerations

None

# Security Considerations

Application flows can be protected at the network layer as described in the
{{!RFC9055}} Section 10. In case applications provide additional data (metadata)
to the network layer, the integrity of metadata has to be protected from  the
application endpoint to the DetNet edges using existing layer 3 admission control and encryption mechanisms.

# Acknowledgements

This work is supported by by the European Commission Horizon Europe SNS JU project DESIRE6G (GA 101096466).

--- back

# Appendix: Potential OCN-EH Extension Header Approach {#ocn-eh}

An interface from application to network using IPv6 operation and control
Extension header (EH) option is proposed as means for app-flow to express
network resources with a fine granularity. Other options as YANG based
provisioning do not scale, nor are easy to change dnamically. Since
applications generating app-flows use IP, an IPv6 EH option provide are a more
natural fit than other encapsulations and is specifically suitable for DetNet
unaware end systems.


OCN-EH solution is an in-band interface to the DetNets from OT
applications with a programmable and dynamic process automation capabilities.
Once the network is engineered for DetNet services, it can map
the incoming traffic with OCN-EH with in its(DetNet Domain) capabilities.

## Operation and Control Network Option (OCNO) {#ocno}

   The OCN Option (OCNO) is a hop-by-hop option that can
   be included in IPv6 for OCN traffic control specification.

~~~~drawing

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
                                   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                   |  Option Type  |  Opt Data Len |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | OCN Code                    |   OCN-TC-Flowlet nonce         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | OCN Param | len |       value                                |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | OCN Param | len |       value                                |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | OCN Param | len |       value                                |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~
{: #ocn-detnet title="Explicit Traffic Control HBH Options"}

{: vspace="0"}
  Option Type:
  :   8-bit identifier of the type of option.  The option identifier
   for the OCN Option (0x??) to be allocated by the IANA. First two bits
  will be 00 (skip over this option and continue processing the header.)

  Option Length:
  :  8-bit unsigned integer. Multiple of 8-octets.

  OCN Code:
  :  16-bit Traffic type code

Flowlet nonce:
:     16-bit. Identifies that a packet is associated with a group of
      packets and shares fate. For example, an application can set the
      same nonce for a set of actuators and sensors. When set to 0,
      flow-id is set to the same value in related flows. When flow-id is
      also 0, no relationship exists.

OCN Parameters:
:    described as parameter code, length and value


The workflow of traffic with EH option happens in the following steps:

1. An end system (industrial controller)  uses the format described in
  {{ocno}} to provide traffic type and constraints.  It fills option type, len fields along with OCN parameters if needed.

1. Platform logic related deterministic processing is not part of the
  network latency in EH; Packet is tranmitted on interface connected to
  DetNet relay node.

1. DetNet relay node processes parameters, and source/destination addresses
  associate an app-flow to DetNet flow. It may or may not remove EH
  see {{encap_pre}}, and inserts its own DetNet encapsulation (technology specific).

1. In case of known exceptions or errors, the relay node could reply to application with traffic type ALEV_TT.

1. DetNet delivers the packet with guarantees of traffic type
   requested to the endsystem gateway connecting to field devices.

1. Field device gateway performs protocol translation and deliver packet to
   the field device.

1. Observable errors, such as late delivery or inconsistent OCN header may
  be sent to the application from the gateway.

1. Similarly, gateways insert new OCN headers for messages originating from
   field devices, such as alarms or other sensor data.


## OCNO EH Processing {#encap_pre}

 - OCNO EH  can be extended for conveying errors from DetNet to the industrial controller application. For example, when a service violation
happened in the DetNet, relay node will set an error flag in OCNO EH.
- Field devices are considered resource-constrained and are not expected to insert or process extension headers.

Two different approaches of hop-by-hop options processing are feasible.

 1. EH is inserted by the application. The relay node performs mapping to DetNet flow.
 2. if the DetNet data plane is IPv6 end to end, then EH can be carried and processed on each hop to the last relay node, which
    acts as a gateway for the fld device and performs EH processing.

Currently only the first option is assumed.
