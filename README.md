# The Forge simple InterModComms Standard
A simple standard for making IMCs useful for exposing an API without actually needing to compile against any mods that provide an API.

## Motivation
MinecraftForge provides a way to send and receive messages from one mod to another mod at mod loading time. The idea behind this is that mods may use APIs provided by other mods in order to achieve inter-mod compatibility (or even synergetic features). Unfortiounately, this mechanism does not come with a standard on how these messages should be formatted. As a result of this, currently inter-mod compatibility is achieved primarily by adding compile-time dependencies to other mods, which may provide a richer and more straight-forward API, but also increases workspace setup complexity and may set the entry bar for newbie mod developers higher than it ought to be, as well as adding unneccessary bloat to mods when the mod for which compatibility is desired is not present. *(Note: this standard is **not** applicable for dependencies, only for exposing an API other mods so mods can make use of each other's features.)* To solve this issue, this repository provides the definition and a sample implementation of what I call the UnifiedIMC Protocol (UIMCP for short).

## Prerequisites
This standard is only applicable to Minecraft Versions 1.16 and beyond as well as to Java 8 and onwards as it relies heavily on java.util.function.* .

## The baseline
At it's core, MinecraftForge provides InterModComms (IMC) by adding two Events: the InterModEnqueueEvent as well as the InterModProcessEvent. First, the InterModEnqueueEvent is called on **all** mods, then the InterModProcessEvent is called on each mod which has a message sent to it. That mod can then process it's IMCs in whatever way it perceives as appropriate.

## The UIMCP pipeline
The UIMC Protocol works by completing steps in a pre-defined order, which are the following:
### 1. Broadcasting
