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
In the InterModEnqueueEvent of the mod that provides the API, an IMC message is sent to every other loaded mod:
The "method" argument must be "api" and the "thing" argument must be a supplier of a (callback) function that takes in a String "key" and returns a function "value" for that key. Three keys must always return a function in order for a mod to be UIMCP-compliant:
  1.  The supplied function must return a function of type <String, Void> for the key "version". The function associated with the key "version" must return the API       version when invoked with no arguments.
  2.  The supplied function must return a function of type <String, Void> for the key "apiName". The function associated with the key "apiName" must return the           API's name when invoked with no arguments.
  3.  The supplied function must return a function of type <Function <<Function<Function<?,?>, String>, String> for the key "customAPIFunctions". The function           associated with the key "customAPIFunctions" must return a function which takes in the callee's modId as an argument and returns a function which returns           functions for a specific keyword. This is used for providing special functions only for a specific calling mod. An example can be seen below. If no special         functions are available for the calling mod, the function returned by "customAPIFunctions" should return null when called with the modId of the calling mod.
  4.  The supplied function must return a function of type <Function<Function<?,?>, String>

## Example usage
An example implementation would look like the following:
