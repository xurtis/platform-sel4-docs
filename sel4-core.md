% The seL4^\textregistered^\ Core
% Benno
% Draft of \today
<!--
	Use the above to set title, author and date.
	First use of seL4 must have registered trademark sign, as above.
	Date is optional; if no date is given, author(s) is optional.
-->

<!--
	Copyright 2020, Ben Leslie, Gernot Heiser
	SPDX-License-Identifier: CC-BY-SA-4.0
-->
	
\doCopyright[2020]
<!--
	Keep the above command at the top to produce the copyright note,
	the argument is the copyright year; if omitted (incl brackets),
	it defaults to the year of build.
-->

seL4 core is the essential set of seL4 components that are needed to build any system.
In the spirit of the general microkernel approach, these components are the set that imposes minimal policy on system builders.
The policy free spirit applies to both run-time and build-time.

A attribute of seL4 Core is that it *must* be high quality.


# Quality

What does quality mean?

Here are some attributes:

1: It must be clearly documented which boards and configurations are supported.

2: All the supported boards must support all configurations.

3: The release must include a way test test the functionality and performance of the components.

4: All the individual tests and benchmarks must be clearly documented as to their purpose.

5: All the tests must pass.

6: All the benchmarks must run and provide reasonable results; any marketing material as to kernel performance *must* be based these benchmarks for a release version only.

7: The release must include evidence of tests and benchmarks (so a human readable report with clear traceability must be included as part of the release).

8: The release must include a cryptographic hash of the contents that is clearly published on the website.

9: The build must be reproducible; a user must be able to take the source release and generate a bit-for-bit accurate copy of the release.

10: Any configuration options must be documented, including explaining why/when a user would want to set it (ideally fewer options the better).

11: All APIs must be accurately documented.

12: Source code of the core must be high-quality and an exemplar for users.

13: Any jira tickets associated with the configurations / boards have been resolved. Or, if not, are very clearly spelled out in the release notes.


# Components

Exactly what components should go in the core is a challenge.

seL4 Core is for users writing their own rootserver and systems.
It should avoid policy where possible.
However, it should also be sufficient enough to get started writing a rootserver out of the box.
There is obviously a tension here!

I would suggest that it must include:

1/ The kernel itself (obviously!)
2/ libseL4 (obviously!)
2.1/ libsel4vm (maybe?)
3/ sel4runtime
4/ minimal libc (printf yes, malloc no, file system ops no)
5/ board UART driver sufficient to support printf [not interrupt driven, output only].
6/ elfloader [where applicable]
7/ script for generating a boot image for the platform


# Configurations

We there should minimal set of configurations.
Supporting many configuration sucks.
Current configurations (apart from debug/release which I consider a separate thing) worth supporting:

 SMP    yes/no =>
 MCS    yes/no =>
 HYP    yes/no =>
 32/64  yes/no =>

Four boolean flags is a total of 16 different configurations (per board) to support.
Double it for release/non-release on of each.

I think there should be concerted effort to remove non-MCS configuration.

Is there a motivating case for Aarch32 kernel on 64-bit capable CPUs? Probably not?


# Example projects

In addition, to the above components there should be sample projects for:

* very simple hello world - enough to demonstrate some very common basic usage, directly using libseL4, without layers and layers of abstraction.
   - print hello world
   - create a thread
   - create an address space
   - handle faults
   - send ping-pong IPC

* sel4bench

* sel4test

Each of these above project should be done in such a way that their build system is not intrinsically intertwined with the SDK.
Even is this means some duplication!


# SDK layout

There should be a simple layout per board/configuration. An example layout would be something like:

   sdk/<board name>/<release/debug>/<config name>

A user, in their build system, would point to the appropriate SDK directory.

Under this top level directory would be something like:

   include/...
   include/seL4/...
   lib/*.a
   image/kernel
   image/elfloader
   bin/build_image_script

The user just need to make sure their compiler is passed `-I$SDK/include -L$SDK/lib -lsel4` for the most part when building their project.
The user is then relatively free to use the tools that are most appropriate to them, and at the level of complexity warranted for their project.


# How does this change the end user experience

- They never have to deal with repo (unless they want to)
- They don't have to deal with cmake (unless they want to)
- They can start an build up easily. For example, the path can be:
  - Download the binary SDK
  - Open a text editor with "rootserver.c"
  - Type in the classical hello world.
  - Compile with gcc -I$SDK/include -L$SDK/lib -lsel4 rootserver.c -o rootserver
  - Create an image  $SDK/bin/build_image_script rootserver -o bootable_image
  - Run the image on their board
  - Compare this to now... learn repo, learn cmake, take seL4Test, start adapting, read code that
    has 11 layers of indirection to find out how things work. This is a horrible experience. I
    should know I've just done it!

# How can this help a kernel developer / engineer porting to a new platform?

 - The code to generate seL4 Core should just go in a single repo -- no more repo.
 - We could then only have to write a UART driver once not 3 times.
 - Testing seL4Bench/seL4Test/helloworld against a new SDK would be much easier than manifest hell.
 - It would be clear which configurations are working or not when starting a new port.
 - Clearer set of things to support / target.

# Things to do

## Standard kernel configurations

Build system to define 2 orthogonal configuration options
1. SMP enabled
2. hypervisor mode enabled.
All standard configurations are based on the MCS kernel.

A platform can only be considered *supported* if all four
configurations work (unless the processor lacks the relevant feature).

Different standard configs may have different versions of libsel4,
although this should be deprecated. Should be tared up as an SDK.

There should be a binary release for each configuration.

## sel4corelib

Minimal C library with no dependencies.

* No `putc()`

## SDK

* kernel binary
* `sel4corelib`
* minimal linker
* no Makefile needed
* SHA hashes of source used to build

## sel4bench

* sel4bench should run on the "standard config" production kernel
  (release binary)
* To keep the null-syscall number, this should be run on a *separate*
kernel that is only there for that purpose
* MCS kernel benchmarking must use passive servers for the headline
IPC latencies
* We need performance regression alerts!
