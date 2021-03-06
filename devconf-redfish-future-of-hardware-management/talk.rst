
Redfish - future of hardware management
=======================================

*by Dmitry Tantsur and Ilya Etingof, Red Hat*

Why hardware management?
========================

Imagine you are sysadmin. It's late night, you are in your deep sleep.
Suddenly, you wake up - a shift engineer calls you notifying that company's
core application has become unresponsive.

You jump out of your bed, reach out for your computer, hop on the
company's network to realize that the system is down.

Desperate, you are trying to understand the problem when the phone
rings again. Uh, that's your boss. They are very concerned,
almost freaking out. They explain the enormous pressure they experience
from all the angry customers...

In the end you got 10 more minutes to figure it out or...

What do you do next?

Hardware management to rescue
=============================

Luckily, you are a wise sysadmin! You have done your disaster recovery
homework. You got some form of hardware management at your disposal.

So you used it to get on remote console of the server to figure out that
it's a data corruption problem. So you went ahead booting server from
network, restoring the data and booting back to normal.

End of story.

The purpose of this is to show one of the cases when hardware management
makes sense.

Management of scale
===================

The other use case when hardware management can become a game changer is a
large-scale deployment e.g. the situation when you got to run a large farm
of servers perhaps at a DC.

A hardware management system capable to boot the servers, hook on the
console, update the firmware, monitor physical parameters etc. can
significantly improve data centre automation.

Keep in mind that a single company as large as Microsoft can have about
a million hardware servers in operation. When you have so many units,
they tend to come and go almost all the time.

This is where server automation may bring huge savings of all sorts.

How it works
============

Typically, the hardware management system is based on a small satellite
computer known as BMC (baseboard management controller). It is frequently
a SoC running alongside the main board. It has direct access to the parts
of the main system what allows it to power up/down the main system, change
BIOS settings etc.

The fact that it is an independent computer makes so-called out-of-band
access to the main system possible. Out-of-band here means that the
main system takes no part in the management operations. It can be up and
running or completely dead.

The BMC is running a software agent which acts as a frontend to all the
features of the management platform. This software agent implements a
protocol over which the clients can talk to the BMC.

What is Redfish
===============

This talk is about the Redfish protocol, however many others exist now
days or existed in the past.

The problem with so many standards is that you can't easily choose one.
More often than not, you are forced to support some of them to fully manage
your fleet.

It is anticipated, that the legacy protocols will be eventually phased
out in favour of the Redfish.

Before the Redfish
==================

The IPMI (Intelligent Platform Management Interface) is the direct
predecessor of the Redfish. It was designed 20 years ago to run on
the weak computers of the time.

Now days IPMI turned out to be difficult to deal with because of:

* it's hard to script with it
* does not scale up due to UDP transport
* not quite secure
* lagging behind functionally

The latter probably explains why almost all hardware vendors eventually
came up with their own proprietary hardware management protocols.

By the way...
=============

Why do they call it 'lights out' manager? What does it mean 'lights out'
in this context?

Any ideas?

Even earlier
============

But it used to be worse in the past!

Before the BMC era, sysadmins used bulky, remotely controlled
keyboard-video-mouse switches, serial console servers (which aggregate the
consoles of the servers to a single, networked server).

For hard power-cycling people used remotely controlled circuit breakers.

Way less reliable in-band technologies were also used for emergency system
access. Such as VNC or RDS. The problem with the in-band access is that
it relies on the main system being in a reasonably good health what may
not be the case in case of emergency.

When it all started
===================

But it used to be even worse in the past!

Before all these hardware hacks were invented, the businesses have to
keep a living soul at the DC at all times.

In the early years of my career in computing, I carried out night
shifts at the DC. That involved sleeping on a folding bed between the
roaring racks hugging pager tightly...

So much for the history!

Redfish design
==============

Now, what's Redfish?!

The Redfish protocol is quite new. It has been released as an official
standard of the Distributed Management Task Force organization in 2015.
And it keeps developing!

To put it simple, Redfish is a web service exposing REST API communicating
JSON-serialized data.

It implements synchronous and asynchronous calls and it is designed for
future extensibility. That allows, for example, hardware vendors to
seamlessly support their proprietary features within the Redfish server.

Redfish benefits
================

From functionality standpoint, Redfish is not a groundbreaking development.

Its usability lies in its wide adoption in the future and the very well
understood technologies Redfish is based on. That makes it easy for
the the operators to integrate Redfish into their existing workflow
and tooling.

Redfish core components
=======================

Redfish models all manageable physical components of the computer. The models
are exposed through the REST API as resources. So models and resources are
roughly the same things.

Clients request operations to carry out on resources. The operations that
can be done in CRUD manner are mapped to HTTP methods.

Besides resource state changes, Redfish implements a handful of high-level
applications that may not be directly relevant to the hardware management.
We will touch services later in this talk.

Redfish resources
=================

As the current core Redfish schema goes, a Redfish agent exposes Systems
branch where it has configuration, inventory and state information for all
the computers being managed.

At the DC, individual computers are normally mounted in the racks. Or blades
are mounted in an enclosure. The Chassis branch references all racks or
enclosures being managed, the inventory information, rack configuration and,
most importantly, it links-in the computers mounted in each rack by
referencing them in the Systems branch.

Finally, there is the Managers branch that exposes capabilities, state,
configuration and actions related to the BMC, enclosure manager,
rack e.g. the out-of-band management system being controlled by this
Redfish agent.

What's important to note here is that all these JSON documents that
the Redfish agent serves, they hyperlink each other. That makes it possible
for the clients to reconstruct the physical layout of the DC by simply
crawling the web service.

Redfish operations
==================

Redfish uses vanilla HTTP for many things. For example, if you want to
read current state of a resource, you just do HTTP GET. To create some
new configuration entity you will use HTTP PUT while changing a property
of a resource may be done though HTTP PATCH.

But HTTP methods only map well on idempotent operations. Sometimes
you may want to apply the same operation on a collection of resources, or
request a state change (such as system reboot) which is not idempotent and
which does not lead to immediate reflection on the resource state.

To accommodate such operations, Redfish has the concept of Actions.
With Actions you just notify Redfish what you need to do, not the
desired state of a specific resource. Examples include flipping
system power or rebooting the system.

Redfish services
================

The Redfish services is a collection of tools providing the features that
are not always directly relevant to hardware management.

When an otherwise normal operation is going to take more than a few seconds
to complete, Redfish agent may decide to run that operation asynchronously.
It then creates a task at the Task service and returns HTTP code
202 (Accepted) along with a link to that task. The client is expected to
poll that URL waiting for task to complete and eventually to receive
the response.

As a web service, Redfish supports basic user authentication as well as
sessions. Client can obtain an authentication token through the Sessions
service.

The user accounts used by clients talking to the Redfish agent are created
at the Redfish agent via the AccountService.

Some resources may need to communicate alerts or error conditions to the
clients at random times. To accommodate that need the EventService can
be used by clients to register the URL they will implement and listen at
for each Resource they are interested in.

Extending Redfish
=================

It is not realistic to come up with a standard that would fit all use-cases.
It is especially hard with hardware - it keeps developing, new features
pop up all the time.

The Redfish's approach to this is to define a minimalistic framework which
could be easily expanded whenever needed. On top of that, Redfish defines
the core functionality which is common across typical hardware (we touched
that earlier).

Vendors can define new resources and actions as well as extend the existing
ones.

As for the extensions, Redfish mandates vendor-specific stuff to be
named spaced by vendor name. That should reduce the chance of collision.

Future of Redfish
=================

Being a framework, it's no wonder that Redfish keeps building up some
more meat on top of its skeleton. The most recent and important development
efforts focus on modelling:

* directly attached and networked storage
* systems composability

Besides these core features, vendors work on moving away from their
proprietary protocols towards Redfish.

Directly attached storage
=========================

Now days we have two fundamentally different approaches to computer storage.
We can have some sort of permanent storage directly attached to a computer
system via a hardware bus. Or we can have a stand-alone storage service
that can be consumed over network by many computer systems.

Redfish has models for both directly attached and networked storage
designs.

For directly attached storage Redfish introduces the Drives, Volumes and
Storage resources. Drives represent physical media which is used to
construct Volumes. The Storage resource combines Drives, Volumes and
Storage Controllers together.

To model the physical relationship between the ComputerSystem and a
directly attached storage, a hyperlink is present from a System object
to a Storage object.

The above model is capable to represent many physical arrangements one
can encounter. Ranging from a single SATA controller + drive up to a set
of redundant storage controllers backing arrays of hard drives arranged
into RAID volumes.

Networked storage
=================

Redfish treats networked storage as a computer system offering storage
services. As such Redfish exposes such computer systems under Systems
resources. It also maintains a shortcut to those computer systems
from the StorageSystems endpoint.

The logical view of the storage being provided is available through the
StorageServices branch. Each StorageService entry reveals all the
properties and configuration of the storage.

Each such ComputerSystem references the StorageService object which
models all the details about storage structure, properties,
configuration, health, access details etc.

There is no direct association between the storage service and its clients
at the Redfish level. That's because storage consumption happens at the
OS level, that is well above hardware management protocol.

Network modelling
=================

Redfish re-uses the YANG language and modules that describe network elements
and their relationships for modelling the network. Basically, Redfish
represents YANG modules describing particular network as JSON documents
under the NetworkDevices endpoint.

Since some network elements have similarities with computer systems,
Redfish exposes devices like routers or switches under the Systems
and Chassis endpoints.

Systems composability
=====================

There is the emerging concept among the hardware vendors that hardware
resources can be pooled and then picked up and assembled into a functional
computer fully programmatically by the end user.

The motivation behind that is to improve resource utilization by provisioning
the hardware which is exactly suitable for the job. In some sense this idea
resembles cloud instances where you can allocate and spin up a virtual machine
having that among of RAM or CPUs as you want.

There is a Redfish extension that exposes pools of hardware resources and
allows the clients to compose computer systems from pieces via REST
API.

Redfish challenges
==================

To truly succeed Redfish needs to take over all those proprietary,
vendor-specific hardware management protocols. That would require
significant efforts on vendors side as well as the customers would
need to adjust their tooling.

The other concern is that having many features stuffed inside Redfish
may grow it beyond comprehension. That may also affect the performance
and stability of the Redfish agent implementation.

But we will see!
