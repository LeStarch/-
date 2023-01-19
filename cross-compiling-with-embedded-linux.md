# Cross-Compiling with Embedded Linux

This is a collection of notes gathered while trying to do simple cross-compiling with embedded linux. Some deductions
may be blatantly incorrect and based on false speculation, however; these deductions seem to "fit the data" gathered.

**Note:** the data is (most definitely) a lie but has led to enough confidence int the false speculations to justify
logging them herein.

Within this 講話 the goal is to cross-compile application code on a host computer with the intent of running the
application on an embedded Linux system. We are specifically not cross-compiling the Linux Kernel itself.

## Vocabulary and Terminology

Some useful domain specific terms:

1. Application: program to be run
2. Host: computer used to build but not run the desired application. (e.g. my Linux Laptop)
3. Target: computer that will run but not build the desired application. (e.g A Raspberry PI)

## What Is Needed To Cross-Compile

There are fundamentally two things that are needed to cross-compile to another target. These items include:

1. **Libraries and Headers**: these represent the target's Linux environment such that an application can run within the
      existing Linux installation.

2. **Toolchain**: the toolchain is the compiler and its associated friends (C++ compiler, assembler,linker, and
                  binary manipulation tooling).

In order to cross-compile, one needs to compile the application code with the toolchain and against the headers and then
link against the libraries. If done correctly the produced binary should run on the target hardware.

This is easier said than done. Notably the libraries and headers (seem to) come from multiple sources: the toolchain
provides some, sysroot can be used to add others, and even the host computer tries to inject others. We'll cover these
concepts in more detail in the following sections

## Libraries and Headers: Where to Get Them

The libraries and headers represent the interface to the target's Linux installation that will run an application. This
means it is essential to ensure that the libraries installed on the target system are fully compatible with the built
application.

The easiest way to do this is to use static linking when building an application. This will ensure that code from the
libraries is copied into the applications' binary and loaded from there. Compatibility is no longer an issue because the
application internalizes a copy of the library code and does not ask the target installation for help. **Where possible
use static linking**.

The alternative to static-linking is to use dynamic (at runtime) linking. The application asks the target operating
system for shared library code while running. Here compatibly is a major issue because the OS must supply the same
version of libraries as was used at build time. 

### SupplyingTarget Libraries to the Compiler

For most user-provided libraries, there are several ways to provide a library to the compiler:

1. Using a `SYSROOT`: this involves supplying a directory that represents the installed target filesystem including
   headers and libraries. The compiler can be told to search there for libraries. **Warning:** there is a major caveat
   see [The LIBC Problem](#the-libc-problem).
2. Building the library from source: one can also build a given library from source and add it via include flags and
   ship the library with the application. This is typically done to support static linking with third party libraries.

### The LIBC Problem

For various reasons the standard LIBC provided but GNU Linux environments should not be statically linked. I am not
entirely clear why this should not be done, but the denizens of the internet seem to agree on this point. This becomes
a problem as LIBC is typically supplied (along with some helpful startup code) from the compiler not from a supplied
path (i.e. SYSROOT) and thus the LIBC version is determined from the compiler/toolchain/host and **not** the target. 
The issue arises when one runs the application on the target and only the target's libc exists.  SYSROOT will not
save you from this terrible fate.

To complicate this, there is also library code supplied by the compiler that is used to start up the application and
this cannot be found anywhere in the target distribution (i.e. SYSROOT). 

**Possible solutions:**
1. **(Not Tried)**: Use a statically linked LIBC implementation (MUSL LIBC)
2. **(Works)**: Choose a toolchain supplying a compatible version of LIBC (i.e. a vendor, or older platform toolchain)
3. **(Difficult)**: Turn off standard supplied libraries and directly link against desired LIBC. **Note:** this doesn't
   seem to work on a practical level as determining the "start up code" is non-trivial.
4. **(Not Tried)**: Ship the compiler's LIBC implementation as a user-installed library.
5. **(Not Tried)**: Pin LIBC symbol versions with inlined-assembly

**Recommendation:** the only sure way to produce a compatible binary that can use a dynamically linked LIBC is to build
with a toolchain that is compatible. Since LIBC is backwards compatible this typically means an older compiler. We will
elaborate on toolchains below.


## Toolchains: Where to Get Them

There are many sources for toolchains for a given embedded Linux platform. Each has its advantages and disadvantages
discussed in this section. First let's enumerate some sources for toolchains:

1. Embedded Linux Vendor
2. Generic Toolchain Packages
3. Community Toolchain Packages
4. Host OS Packages (i.e. `apt install <toolchain>`)
5. Rolling your own

### Vendor Toolchains: Pros and Cons

Vendor toolchains should "just work" as they are supplied by the vendor of the embedded Linux platform. These are not
always available but when they are available (and reasonably up-to-date) this is the best choice for easy-cross-compiles
as the results should be tailored to the embedded system.

**Pros:**
- Easy
- Preconfigured for platform
- Preconfigured for vendor provided libraries
- Tested

**Cons:**
- Not provided by every vendor
- Licensing
- Old compiler versions
- Probably not Open Source
- Expects specific Linux Toolchain

### Generic Toolchains: Pros and Cons

Sometimes organizations promoting an architecture will publish toolchains for that architecture. This usually targets
a specific architecture (e.g. ARM) but not a specific platform (e.g. Raspberry PI). These work well 

**Pros:**
- Easy
- Preconfigured for platform
- Tested

**Cons:**
- Licensing
- Old compiler versions
- Probably not Open Source
- Expects specific Linux Toolchain
