---
icon: file-circle-exclamation
description: >-
  This page explains how to use exception handlers, which allow you to catch and
  handle exceptions that occur during command execution, and modify the default
  behavior for each exception.
---

# Exception handling

## CommandExceptionHandler Interface

The `CommandExceptionHandler` interface is a key component in Lamp that allows handling exceptions thrown during command execution. Each platform (e.g., Bukkit, Sponge, Velocity, JDA, etc.) has its own set of exceptions and a corresponding `DefaultExceptionHandler` extension tailored to handle those exceptions.

### Overview

#### What is `CommandExceptionHandler`?

`CommandExceptionHandler` is a functional interface designed to handle any exceptions that may occur during command invocation. These exceptions can be standard Java exceptions or specific ones thrown by different components of the framework.

#### Default Exception Handlers

For each platform, there is a `DefaultExceptionHandler` extension that handles common exceptions relevant to that platform. Users should extend the respective platform's `DefaultExceptionHandler` to customize their exception handling.

For example:

* **Bukkit**: `BukkitExceptionHandler`
* **Sponge**: `SpongeExceptionHandler`
* **Velocity**: `VelocityExceptionHandler`
* **JDA**: `SlashJDAExceptionHandler`
* and so on

#### How to Use

To create custom exception handling logic for your platform, follow these steps:

1. **Extend the DefaultExceptionHandler**: Extend the default exception handler class provided for your platform.
2. **Add Custom Exception Handling**: Use the `@HandleException` annotation to define methods that handle specific exceptions.
3. **Set the Exception Handler**: Register your custom exception handler with the Lamp builder.

### Example: Bukkit Platform

Here’s an example of how to create a custom exception handler for the Bukkit platform.

#### Step 1: Extend `BukkitExceptionHandler`

{% tabs %}
{% tab title="Java" %}
```java
public class MyBukkitExceptionHandler extends BukkitExceptionHandler {

    @Override
    public void onInvalidPlayer(InvalidPlayerException e, BukkitCommandActor actor) {
        actor.error(legacyColorize("&cInvalid player: &e" + e.input() + "&c."));
    }

    @HandleException
    public void onSomeCustomException(SomeCustomException e, BukkitCommandActor actor) {
        actor.error(legacyColorize("..."));
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class MyBukkitExceptionHandler : BukkitExceptionHandler() {

    override fun onInvalidPlayer(e: InvalidPlayerException, actor: BukkitCommandActor) {
        actor.error(legacyColorize("&cInvalid player: &e${e.input()}&c."))
    }

    @HandleException
    fun onSomeCustomException(e: SomeCustomException, actor: BukkitCommandActor) {
        actor.error(legacyColorize("..."))
    }
}
```
{% endtab %}
{% endtabs %}

In this example, `MyBukkitExceptionHandler` extends `BukkitExceptionHandler` and handles two exceptions: `InvalidPlayerException` and `NoPermissionException`.

#### Step 2: Register Your Custom Handler

{% tabs %}
{% tab title="Java" %}
```java
var lamp = BukkitLamp.builder(this)
    .exceptionHandler(new MyBukkitExceptionHandler())
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = BukkitLamp.builder(this)
    .exceptionHandler(MyBukkitExceptionHandler())
    .build()
```
{% endtab %}
{% endtabs %}

The `exceptionHandler` method in the Lamp builder registers your custom exception handler with the framework.
