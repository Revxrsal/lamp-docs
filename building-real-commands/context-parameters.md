---
icon: nfc-magnifying-glass
---

# Context parameters

### Overview

`ContextParameter` is a functional interface that allows you to resolve parameters based on the context of the command execution. This can be useful for extracting complex types or values from the command context.

### ContextParameter Interface

The `ContextParameter` interface is used to define how a parameter should be resolved from the command context.

```java
@FunctionalInterface
public interface ContextParameter<A extends CommandActor, T> {

    /**
     * Reads input from the given {@link MutableStringStream}, parses the object, or throws
     * exceptions if needed.
     *
     * @param parameter The parameter
     * @param context   The command execution context, as well as arguments that have been resolved
     * @return The parsed object. This should never be null.
     */
    T resolve(@NotNull CommandParameter parameter, @NotNull ExecutionContext<A> context);
}
```

### ContextParameter.Factory Interface

The `ContextParameter.Factory` interface is used to create `ContextParameter` instances dynamically. It is helpful for creating custom parameter types.

```java
@FunctionalInterface
public interface ContextParameter.Factory<A extends CommandActor> extends ParameterFactory {

    /**
     * Dynamically creates a {@link ContextParameter}
     *
     * @param <T>           The parameter type
     * @param parameterType The command parameter to create for
     * @param annotations   The parameter annotations
     * @param lamp          The Lamp instance (for referencing other parameter types)
     * @return The newly created {@link ContextParameter}, or {@code null} if this factory
     * cannot deal with it.
     */
    @Nullable
    <T> ContextParameter<A, T> create(@NotNull Type parameterType, @NotNull AnnotationList annotations, @NotNull Lamp<A> lamp);
}
```

### Example: PlayerWorldContextParameterFactory

Here is an example of a `ContextParameter.Factory` that resolves a `World` from the `Player` context:

{% tabs %}
{% tab title="Java" %}
```java
public class PlayerWorldContextParameterFactory implements ContextParameter.Factory<BukkitCommandActor> {

    @Nullable
    @Override
    public <T> ContextParameter<BukkitCommandActor, T> create(@NotNull Type parameterType, @NotNull AnnotationList annotations, @NotNull Lamp<BukkitCommandActor> lamp) {
        if (parameterType != World.class) {
            return null;
        }
        PlayerWorld playerWorldAnnotation = annotations.get(PlayerWorld.class);
        if (playerWorldAnnotation == null) {
            return null;
        }
        return (parameter, context) -> context.actor().sender().requirePlayer().getWorld();
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
object PlayerWorldContextParameterFactory : ContextParameter.Factory<BukkitCommandActor> {

    @Suppress("UNCHECKED_CAST")
    override fun <T> create(parameterType: Type, annotations: AnnotationList, lamp: Lamp<BukkitCommandActor>): ContextParameter<BukkitCommandActor, T>? {
        if (parameterType != World::class.java) {
            return null
        }
        val playerWorldAnnotation = annotations.get(PlayerWorld::class.java)
        if (playerWorldAnnotation == null) {
            return null
        }
        return ContextParameter { _, context -> context.actor().sender().requirePlayer().world } as ContextParameter<BukkitCommandActor, T>
    }
}
```
{% endtab %}
{% endtabs %}

### Registering ContextParameter.Factory

You can register a `ContextParameter.Factory` with the `Lamp` builder to handle custom parameter types.

{% tabs %}
{% tab title="Java" %}
```java
var lamp = BukkitLamp.builder(this)
    .parameterTypes(parameters -> {
        parameters.addContextParameterFactory(new PlayerWorldContextParameterFactory());
    })
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = BukkitLamp.builder(this)
    .parameterTypes { parameters ->
        parameters.addContextParameterFactory(PlayerWorldContextParameterFactory())
    }
    .build()
```
{% endtab %}
{% endtabs %}
