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
The "method" argument must be "api" and the "thing" argument must be a supplier of a thread-safe, preferrably synchronized (callback) function that takes in a String "key" and returns a function "value" for that key. Six keys must always return a function in order for a mod to be UIMCP-compliant:
  1.  The supplied function must return a function of type <String, Void> for the key "version". The function associated with the key "version" must return the API       version when invoked with no arguments.
  2.  The supplied function must return a function of type <String, Void> for the key "apiName". The function associated with the key "apiName" must return the           API's name when invoked with no arguments.
  3.  The supplied function must return a function of type <Function<Function<?,?>, String> for the key "apiFunctions". The function associated with the key             "apiFunctions" must return a function which returns API functions when provided with a valid API function name. This is the primary way API functions should       be accessed using the UIMC protocol. If no function exists for that API function name, the returned function shall return null.
  4.  The supplied function must return a function of type <Boolean, String> for the key "hasFunction". The function associated with the key                             "hasFunction" must return either true or false, depending on if an API function associated with the input string exists.
  5.  The supplied function must return a function of type <Boolean, String> for the key "hasCapability". The function associated with the key "hasCapability"           returns wether this API has the capability passed to the function. A list of capabilities and their requirements can be found below.

### 2. Registering
In the InterModProcessEvent of the mod that wishes to use the API, the mod that wishes to use the API iterates over all incoming IMCMessages by invoking InterModComms.getMessages(consumerModID). The mod that wishes to use the API then must check if the API name, API version and API capabilities are appropriate and may continue if they are. If they are not, it is up to the mod that wishes to use the API on how to deal with this. Preferrably, an error is written to stdout to inform the user, and the starting process continues on like normal. This is recommended, but not neccessary. 
After the sanity check has been established, the mod that wishes to use the API should then invoke any special methods which are provided by special capabilities in their defined order (again, see below.) 
After that, the mod that wishes to use the API may then go ahead and check if the API provides the required functions by invoking the function returned by "hasFunction" with the function name to check for. If any one those return false, it is up to the mod that wishes to use the API on how to handle this error.
Finally, the mod that wishes to use the API can get the individual API functions by invoking the function returned by "apiFunctions" with the function name it wishes to receive.

## Capabilities
A full definition and example usage of each capability can be found in it's appropriate file in the "Capabilities" folder of this repository. Here, only rough summaries are provided.

## Example usage
An example implementation would look like the following:

API providing mod:
```java
private static class API {
  public static String getApiName() {
    return "anExampleApi";
  }
  
  public static String getApiVersion() {
    return "1.2.3_04"; //let's see who get's that reference
  }
  
  public static Function<?, ?> getApiFunction(String functionName) {
  
  }
  
  public static boolean foo(String input) {
    return input.equals("a string");
  }
  
  public static void bar() {
    System.out.println("bar"); 
  ]
}

@SubscribeEvent
public void enqueueIMC(InterModQueueEvent event) {
  
}
```

API using mod:
```java
@SubscribeEvent 
public void processIMC(InterModProcessEvent event) {

}
```
