---
description: This page explains how to create cooldowns for commands
icon: stopwatch
---

# Cooldowns

Cooldowns are important in many sensitive operations that cannot afford to be done many times in a short period of time. Therefore, Lamp provides a neat way to integrate cooldowns



### Simple, fixed cooldowns

The most straightforward way to declare cooldowns is using the `@Cooldown`annotation:

{% tabs %}
{% tab title="Java" %}
```java
@Command("foo")
@Cooldown(value = 3, unit = TimeUnit.SECONDS)
public void foo(CommandActor actor) {
    // ...
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("foo")
@Cooldown(value = 3, unit = TimeUnit.SECONDS)
fun foo(actor: CommandActor) {
    // ...
}
```
{% endtab %}
{% endtabs %}

This will put the user on cooldown if the command successfully executes without any error (if the command body throws any exception, the user will not be put on cooldown).



This method is sufficient for simple use cases. However, more sophisticated use cases may need something a bit more granular.

### Cooldown handles

Lamp provides a convenient way to control cooldowns, the `CooldownHandle`interface. This interface is passed as a parameter to the command method, and it provides finer control over the cooldown.

A cooldown handle provides the following functions:

* `isOnCooldown()`: Check if the user is currently on cooldown.
* `elapsedMillis()`: Get the elapsed time since the cooldown started in milliseconds.
* `cooldown()`: Puts the user on cooldown
* `requireNotOnCooldown()`: Checks if the user is on cooldown, and if they are, throws a `CooldownException`(which is handled by Lamp automatically)
* `removeCooldown()` : Removes the cooldown
* `remainingTime(TimeUnit)`: Gets the time remaining and converts it to the given unit

**Note**: Some of the above functions provide two overloads: `f(...)`and `f(..., long, TimeUnit)`. The first overload can **only** be used if you have `@Cooldown`on the handle parameter (or received a handle from `#withCooldown()`, otherwise you must provide the cooldown value in the second overload. Attempting to use the first variant will throw an exception.



#### Usage

It can be used in two ways:

1. Combined with `@Cooldown`
2. Without `@Cooldown`

#### With `@Cooldown`:

This is done by declaring the `CooldownHandle` parameter with `@Cooldown`on it:

{% tabs %}
{% tab title="Java" %}
```java
@Command("foo")
public void foo(
    CommandActor actor,
    @Cooldown(value = 3, unit = TimeUnit.SECONDS) CooldownHandle handle
) {
    // ...
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("foo")
fun foo(
    actor: CommandActor,
    @Cooldown(value = 3, unit = TimeUnit.SECONDS) handle: CooldownHandle
) {
    // ...
}
```
{% endtab %}
{% endtabs %}

Note that this will **not affect the behavior of the command by itself**! To actually put the user on cooldown, you have to use the handle:

{% tabs %}
{% tab title="Java" %}
```java
@Command("foo")
public void foo(
    CommandActor actor,
    @Cooldown(value = 3, unit = TimeUnit.SECONDS) CooldownHandle handle
) {
    if (handle.isOnCooldown()) {
        // user is on cooldown
    }
    
    // ...
    
    // put the user on cooldown:
    handle.cooldown();
}

```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("foo")
fun foo(
    actor: CommandActor,
    @Cooldown(value = 3, unit = TimeUnit.SECONDS) handle: CooldownHandle
) {
    if (handle.isOnCooldown()) {
        // user is on cooldown
    }

    // ...

    // put the user on cooldown:
    handle.cooldown()
}
```
{% endtab %}
{% endtabs %}

A clear drawback of this method is that it requires the cooldown value to be known at compile-time. To counter this, it is possible to use `CooldownHandle`s without `@Cooldown`:

{% tabs %}
{% tab title="Java" %}
```java
@Command("foo")
public void foo(
    CommandActor actor,
    CooldownHandle handle
) {
    if (handle.isOnCooldown()) {
        // user is on cooldown
    }

    // ...

    // put the user on cooldown:
    handle.cooldown(10, TimeUnit.SECONDS);
}

```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("foo")
fun foo(
    actor: CommandActor,
    handle: CooldownHandle
) {
    if (handle.isOnCooldown()) {
        // user is on cooldown
    }

    // ...

    // put the user on cooldown:
    handle.cooldown(10, TimeUnit.SECONDS)
}

```
{% endtab %}
{% endtabs %}

Attempting to invoke `#cooldown()`(without parameters) will throw an exception.

To save you from repeating yourself a lot, you can also do `handle.withCooldown()`so that you can use `#cooldown()`and other methods without having to repeat the cooldown value:

{% tabs %}
{% tab title="Java" %}
```java
@Command("foo")
public void foo(
    CommandActor actor,
    CooldownHandle handle
) {
    handle = handle.withCooldown(10, TimeUnit.SECONDS);
    if (handle.isOnCooldown()) {
        // user is on cooldown
    }

    // ...

    // put the user on cooldown:
    handle.cooldown();
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("foo")
fun foo(
    actor: CommandActor,
    handle: CooldownHandle
) {
    handle = handle.withCooldown(10, TimeUnit.SECONDS)
    if (handle.isOnCooldown()) {
        // user is on cooldown
    }

    // ...

    // put the user on cooldown:
    handle.cooldown()
}
```
{% endtab %}
{% endtabs %}

This last method combines the benefits of all the above methods, albeit requires manual management. Pick the one that fits your use case!
