---
description: >-
  This page explains how to use visitors, which enable modular, reusable
  configurations for Lamp and Lamp.Builder through customizable registration and
  hooks.
icon: puzzle
---

# Visitors

## Modular Visitors in Lamp

The Lamp library allows for modular and reusable configurations through the use of visitors. These visitors can be used to customize the behavior of both the `Lamp.Builder` and the `Lamp` instances themselves. This document provides an overview of how to use the `LampBuilderVisitor` and `LampVisitor` interfaces to create modular visitors that can register or add specific features to your command framework.

### LampBuilderVisitor

#### Overview

`LampBuilderVisitor` is a functional interface that operates on a `Lamp.Builder` object. It allows for modular registration of additional features during the builder phase, making it easy to add reusable configurations across different parts of your application.

#### Creating a LampBuilderVisitor

Here's how you can create a `LampBuilderVisitor` to register common parameter types for Bukkit:

{% tabs %}
{% tab title="Java" %}
```java
/**
 * Registers the following parameter types:
 * <ul>
 *     <li>{@link Player}</li>
 *     <li>{@link OfflinePlayer}</li>
 *     <li>{@link World}</li>
 *     <li>{@link EntitySelector}</li>
 * </ul>
 *
 * @param <A> The actor type
 * @return The visitor
 */
public static <A extends BukkitCommandActor> @NotNull LampBuilderVisitor<A> bukkitParameterTypes() {
    return builder -> {
        builder.parameterTypes()
                .addParameterTypeLast(Player.class, new PlayerParameterType())
                .addParameterTypeLast(OfflinePlayer.class, new OfflinePlayerParameterType())
                .addParameterTypeLast(World.class, new WorldParameterType())
                .addParameterTypeFactoryLast(new EntitySelectorParameterTypeFactory());
        if (BukkitVersion.isBrigadierSupported())
            builder.parameterTypes()
                    .addParameterTypeLast(Entity.class, new EntityParameterType());
    };
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
/**
 * Registers the following parameter types:
 * <ul>
 *     <li>{@link Player}</li>
 *     <li>{@link OfflinePlayer}</li>
 *     <li>{@link World}</li>
 *     <li>{@link EntitySelector}</li>
 * </ul>
 *
 * @param <A> The actor type
 * @return The visitor
 */
fun <A : BukkitCommandActor> bukkitParameterTypes(): LampBuilderVisitor<A> {
    return LampBuilderVisitor { builder ->
        builder.parameterTypes()
            .addParameterTypeLast(Player::class.java, PlayerParameterType())
            .addParameterTypeLast(OfflinePlayer::class.java, OfflinePlayerParameterType())
            .addParameterTypeLast(World::class.java, WorldParameterType())
            .addParameterTypeFactoryLast(EntitySelectorParameterTypeFactory())
        if (BukkitVersion.isBrigadierSupported())
            builder.parameterTypes()
                .addParameterTypeLast(Entity::class.java, EntityParameterType())
    }
}
```
{% endtab %}
{% endtabs %}

#### Using a LampBuilderVisitor

To apply this visitor to your `Lamp.Builder`, simply call the `accept` method:

{% tabs %}
{% tab title="Java" %}
<pre class="language-java"><code class="lang-java"><strong>Lamp.Builder&#x3C;BukkitCommandActor> builder = Lamp.builder();
</strong>builder.accept(bukkitParameterTypes());
var lamp = builder.build();
</code></pre>
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val builder: Lamp.Builder<BukkitCommandActor> = Lamp.builder()
builder.accept(bukkitParameterTypes())
val lamp: Lamp<BukkitCommandActor> = builder.build()
```
{% endtab %}
{% endtabs %}

This approach allows you to reuse the visitor across multiple builders, ensuring consistency and reducing boilerplate code.

### LampVisitor

#### Overview

`LampVisitor` is a functional interface designed to operate on an instantiated `Lamp` object. It allows for modular modifications or additions to the `Lamp` instance after it has been built.

#### Creating a LampVisitor

You can create a `LampVisitor` to perform specific actions on the `Lamp` instance, such as registering additional commands or setting up event listeners.

{% tabs %}
{% tab title="Java" %}
```java
public static <A extends CommandActor> @NotNull LampVisitor<A> myCustomLampVisitor() {
    return lamp -> {
        // Example: Register additional commands or listeners
        lamp.register(new MyCustomCommand());
    };
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
fun <A : CommandActor> myCustomLampVisitor(): LampVisitor<A> {
    return LampVisitor { lamp ->
        // Example: Register additional commands or listeners
        lamp.commandHandler.register(MyCustomCommand())
        lamp.eventHandler.register(MyCustomEventListener())
    }
}
```
{% endtab %}
{% endtabs %}

#### Using a LampVisitor

To apply this visitor to your `Lamp` instance, use the `accept` method:

{% tabs %}
{% tab title="Java" %}
```java
var lamp = Lamp.builder().build();
lamp.accept(myCustomLampVisitor());
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = Lamp.builder().build()
lamp.accept(myCustomLampVisitor())
```
{% endtab %}
{% endtabs %}

This approach makes it easy to extend the functionality of the `Lamp` instance without directly modifying its construction logic.
