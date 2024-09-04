---
description: >-
  This page explains how we can create static, dynamic and context-aware
  auto-completions
icon: wand-sparkles
---

# Suggestions and auto-completion

Auto-completions are a vital part of commands. They help users input valid values and save them a lot of typing.

Lamp provides a lot of helpful APIs for creating auto-completions. We will go through them in this page.

## The SuggestionProvider interface

This is the foundational building block that is responsible for handling suggestions and auto-completion. It is a basic functional interface that has access to the command actor and provided parameters (parsed) and returns a list of suggestions.

You will see `SuggestionProvider` in the following places:

* `SuggestionProviders`: An immutable registry that contains all registrations for `SuggestionProvider`s. It is maintained through `Lamp#suggestionProviders()` and can be constructed inside `Lamp.Builder#suggestionProviders()`.
* `SuggestionProvider.Factory`: An interface that can generate `SuggestionProvider`s dynamically for parameters. This factory can access the parameter type (and generics), its annotations, and other SuggestionProviders. It is a powerful interface as it can generate custom suggestions based on the type generics, a common interface, a specific data type (such as enums), or suggestions bound to a particular annotation.
* `ParameterType#defaultSuggestions()`: This is a method that can be overridden in subclasses of ParameterType. It allows parameter types to define a default `SuggestionProvider` that is used when no other suggestion provider is available.

Let's move on to the creation of custom suggestions

## Static completions

This is the simplest form of auto-completion: we have a static set of values that we would like to recommend to the user.

By static completions, we mean hard-coded, compile-time-defined constant completions. For that, we use the `@Suggest` annotation.

{% tabs %}
{% tab title="Java" %}
```java
@Command("coins give")
public void give(
    BukkitCommandActor actor,
    Player target,
    @Suggest({"100", "200", "300", "500", "1000"}) int coins
) {
    /* ... */
}

```
{% endtab %}

{% tab title="Kotlin" %}
<pre class="language-kotlin"><code class="lang-kotlin">@Command("coins give")
fun give(
<strong>    actor: BukkitCommandActor,
</strong>    target: Player,
    @Suggest("100", "200", "300", "500", "1000") coins: Int
) {
    /* ... */
}
</code></pre>
{% endtab %}
{% endtabs %}

This will automatically supply the users with all the suggestions in `@Suggest`. A nicety of the `@Suggest` annotation is that suggestions are allowed to contain spaces. We can define suggestions that contain spaces easily and Lamp will handle the rest.

{% tabs %}
{% tab title="Java" %}
```java
@Command("teleport")
public void tp(
    BukkitCommandActor actor,
    Player target,
    @Suggest("~ ~ ~") Location location
) {
    /* ... */
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("teleport")
fun tp(
    actor: BukkitCommandActor,
    target: Player,
    @Suggest("~ ~ ~") location: Location
) {
    /* ... */
}
```
{% endtab %}
{% endtabs %}

As you can see, there isn't much you can do with `@Suggest`. This is why we will need to introduce more complicated ways for creating suggestions.

## Type-specific completions

It's common to have a certain set of completions bound to a particular Java class. Let's imagine we have a `World` parameter. We can create a `SuggestionProvider` that will retrieve the names of available worlds and give them back to us.

We can register this in our `Lamp.Builder`:

{% tabs %}
{% tab title="Java" %}
```java
var lamp = Lamp.builder()
    .suggestionProviders(providers -> {
        providers.addProvider(World.class, (input, context) -> {
            return Bukkit.getWorlds().stream()
                    .map(World::getName)
                    .toList();
        });
    })
    .build();

```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = Lamp.builder<CommandActor>()
    .suggestionProviders { providers ->
        providers.addProvider(World::class.java) { _, _ ->
            Bukkit.getWorlds().map { it.name }
        }
    }
    .build()

```
{% endtab %}
{% endtabs %}

Then, whenever we have a `World` parameter, we will automatically receive all World suggestions that are retrieved on demand.

{% tabs %}
{% tab title="Java" %}
```java
@Command("world")
public void gotoWorld(Player sender, World world) {
    /* ... */
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("world")
fun gotoWorld(sender: Player, world: World) {
    /* ... */
}
```
{% endtab %}
{% endtabs %}

## Annotation-specific completions

Lamp also provides a way to create auto-completions that are bound to certain annotations. This is a very flexible approach as it combines the benefits of dynamic parameters as well as annotations that allow you to tweak suggestions as needed.

Let's create an annotation named `@WithPermission("some.permission.node")`. This annotation will automatically give us all players that have a certain annotation node.

Let's define our annotation:

{% tabs %}
{% tab title="Java" %}
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME) // <--- it's important!
public @interface WithPermission {
    
    String value();
    
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Target(AnnotationTarget.VALUE_PARAMETER)
annotation class WithPermission(val value: String)
```
{% endtab %}
{% endtabs %}

> :warning:  It's important to ensure that your annotation has **runtime retention**. This means that it should have `@Retention(RetentionPolicy.RUNTIME)` on it. Kotlin annotations have it by default.

Let's create our suggestion provider. We can register one with `SuggestionProviders.Builder#addProviderForAnnotation`, which constructs a `SuggestionProvider.Factory` under the hood.

{% tabs %}
{% tab title="Java" %}
```java
var lamp = Lamp.builder()
    .suggestionProviders(providers -> {
        providers.addProviderForAnnotation(WithPermission.class, withPermission -> {
            String permission = withPermission.value(); // <-- the value inside @WithPermission
            return (input, context) -> { // <-- here we return a SuggestionProvider
                return Bukkit.getOnlinePlayers()
                        .stream()
                        .filter(player -> player.hasPermission(permission))
                        .map(Player::getName)
                        .toList();
            };
        });
    })
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = Lamp.builder<CommandActor?>()
    .suggestionProviders { providers ->
        providers.addProviderForAnnotation(WithPermission::class.java) { withPermission: WithPermission ->
            val permission = withPermission.value
            SuggestionProvider { _, _ ->
                Bukkit.getOnlinePlayers().asSequence()
                    .filter { it.hasPermission(permission) }
                    .map { it.name }
                    .toList()
            }
        }
    }
    .build()
```
{% endtab %}
{% endtabs %}

Now, we can use our `@WithPermission` annotation as needed.

{% tabs %}
{% tab title="Java" %}
```java
@Command("helpop")
public void helpOp(
    Player sender,
    @WithPermission("helpop.admin") Player op,
    String message
) {
    /* ... */
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("helpop")
fun helpOp(
    sender: Player,
    @WithPermission("helpop.admin") op: Player,
    message: String
) {
    /* ... */
}
```
{% endtab %}
{% endtabs %}

> :warning:  **Note**: While Lamp allows you to do everything in lambdas, it is actually discouraged as it can lead to terribly-looking code! For small lambdas, it's okay. But when things get big and messy, it's better to put them in a separate class.
>
> Compare the above with the following, and decide for yourself:
>
> ```java
> public enum WithPermissionSuggestionFactory implements SuggestionProvider.Factory<CommandActor> {
>
>     INSTANCE;
>
>     @Override
>     public @Nullable SuggestionProvider<CommandActor> create(
>             @NotNull Type type,
>             @NotNull AnnotationList annotations,
>             @NotNull Lamp<CommandActor> lamp
>     ) {
>         WithPermission withPermission = annotations.get(WithPermission.class);
>         if (withPermission == null)
>             return null;
>         String permission = withPermission.value();
>         return (input, context) -> {
>             return Bukkit.getOnlinePlayers().stream()
>                     .filter(player -> player.hasPermission(permission))
>                     .map(Player::getName)
>                     .toList();
>         };
>     }
> }
> ```
>
> ```java
> var lamp = BukkitLamp.builder(this)
>     .suggestionProviders(suggestions -> {
>         suggestions.addProviderFactory(WithPermissionSuggestionFactory.INSTANCE);
>     })
>     .build();
> ```
>
> A few extra lines, sure. But this will protect us from messy lambdas, and keep everything in its own separate place. Better organization and maintenance.&#x20;

## Suggestion provider factories

So far, our suggestion providers have been very specific, either to a particular class or to a particular annotation. But, what if we want to generate `SuggestionProvider`s that work even beyond such scopes? This is where `SuggestionProvider.Factory` comes in. Here are some use-cases where it would come useful:

* Create annotation-bound SuggestionProviders that act differently depending on the parameter type or the annotations present on it
* Create SuggestionProviders for a certain class or its subclasses
* Create SuggestionProviders that respect generics (e.g. `List<String>` vs `List<Integer>`)
* Create SuggestionProviders made of other SuggestionProviders (e.g. `T[]` which uses the completions of `T`)
* Create SuggestionProviders that act on a certain type of classes, e.g. automatically generate suggestions for all `enum` types without having to explicitly register them.
* ...

Let's create a basic factory that will generate suggestions for all enum types. This will save us from creating individual SuggestionProviders.

{% tabs %}
{% tab title="Java" %}
```java
public enum EnumSuggestionProviderFactory implements SuggestionProvider.Factory<CommandActor> {
    
    INSTANCE;

    @Override
    public @Nullable SuggestionProvider<CommandActor> create(@NotNull Type type, @NotNull AnnotationList annotations, @NotNull Lamp<CommandActor> lamp) {
        return null;
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
object EnumSuggestionProviderFactory : SuggestionProvider.Factory<CommandActor> {

    override fun create(
        type: Type,
        annotations: AnnotationList,
        lamp: Lamp<CommandActor>
    ): SuggestionProvider<CommandActor>? {
        return null
    }
}
```
{% endtab %}
{% endtabs %}

> :bulb:  A SuggestionProvider.Factory should almost always work with well-defined constraints (in our case, enum types only). Because a SuggestionProvider.Factory has the potential to create suggestion providers for _anything_, we should be careful with what we return. **When a factory receives types/annotations that it does not care about (in our case, a non-enum type), it should return null!**

Let's check if our type is an enum, and if it is not, tell Lamp that we cannot deal with it:

{% tabs %}
{% tab title="Java" %}
```java
@Override public @Nullable SuggestionProvider<CommandActor> create(
    @NotNull Type type, 
    @NotNull AnnotationList annotations, 
    @NotNull Lamp<CommandActor> lamp
) {
    Class<?> rawType = Classes.getRawType(type);
    if (!rawType.isEnum())
        return null;
    return /* TODO() */
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
override fun create(
    type: Type,
    annotations: AnnotationList,
    lamp: Lamp<CommandActor>
): SuggestionProvider<CommandActor>? {
    val rawType = Classes.getRawType(type)
    if (!rawType.isEnum) 
        return null
    return TODO()
}
```
{% endtab %}
{% endtabs %}

We made sure our factory only works on enums. Let's proceed to creating the suggestion provider.

1. We will cache the names of enums
2. We will create a static suggestion provider that returns them **in lowercase**.

{% tabs %}
{% tab title="Java" %}
```java
@Override public @Nullable SuggestionProvider<CommandActor> create(
    @NotNull Type type, 
    @NotNull AnnotationList annotations, 
    @NotNull Lamp<CommandActor> lamp
) {
    Class<?> rawType = Classes.getRawType(type);
    if (!rawType.isEnum())
        return null;
    Enum<?>[] enumValues = rawType.asSubclass(Enum.class).getEnumConstants();
    
    /* Create the list of suggestions */
    List<String> suggestions = new ArrayList<>();
    for (Enum<?> enumValue : enumValues) {
        /* Lower-case and add */
        suggestions.add(enumValue.name().toLowerCase());
    }
    
    /* Return a suggestion provider that always returns suggestions */
    return (input, context) -> suggestions;
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
override fun create(
    type: Type,
    annotations: AnnotationList,
    lamp: Lamp<CommandActor>
): SuggestionProvider<CommandActor>? {
    val rawType = Classes.getRawType(type)
    if (!rawType.isEnum) return null

    /* Map enums */
    val suggestions = rawType.enumConstants
        .map { (it as Enum<*>).name.lowercase() }

    /* Return a suggestion provider that always returns suggestions */
    return SuggestionProvider { _, _ -> suggestions }
}
```
{% endtab %}
{% endtabs %}

That's it! We can simply register this to our `SuggestionProviders.Builder`, and automatically get suggestions for all enums generated. Effortless, efficient and easy!

{% tabs %}
{% tab title="Java" %}
```java
var lamp = Lamp.builder()
    .suggestionProviders(suggestions -> {
        suggestions.addProviderFactory(EnumSuggestionProviderFactory.INSTANCE);
    })
    .build()
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = Lamp.builder()
    .suggestionProviders { suggestions ->
        suggestions.addProviderFactory(EnumSuggestionProviderFactory)
    }
    .build()
```
{% endtab %}
{% endtabs %}

## Parameter-bound completions

Finally, Lamp provides a utility annotation: `@SuggestWith` that allows you to specify either a SuggestionProvider or a SuggestionProvider.Factory for _individual_ parameters. This is useful if we want a quick, dirty way of modifying suggestions for a particular parameter without creating individual annotations or wrapper types.

Let's imagine that we have this suggestion provider that automatically suggests all worlds that start with `world`.

{% tabs %}
{% tab title="Java" %}
```java
public final class DefaultWorlds implements SuggestionProvider<BukkitCommandActor> {
    
    @Override public @NotNull List<String> getSuggestions(@NotNull StringStream input, @NotNull ExecutionContext<BukkitCommandActor> context) {
        return Bukkit.getWorlds().stream()
                .map(World::getName)
                .filter(name -> name.startsWith("world"))
                .toList();
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class DefaultWorlds : SuggestionProvider<BukkitCommandActor> {
    
    override fun getSuggestions(input: StringStream, context: ExecutionContext<BukkitCommandActor>): List<String> {
        return Bukkit.getWorlds().stream()
            .map { obj: World -> obj.name }
            .filter { name: String -> name.startsWith("world") }
            .toList()
    }
}
```
{% endtab %}
{% endtabs %}

Then, we can supply this class using `@SuggestWith`:

{% tabs %}
{% tab title="Java" %}
```java
@Command("world")
public void gotoWorld(
    Player sender,
    @SuggestWith(DefaultWorlds.class) World world
) {
    /* ... */
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("world")
fun gotoWorld(
    sender: Player, 
    @SuggestWith(DefaultWorlds::class) world: World
) {
    /* ... */
}
```
{% endtab %}
{% endtabs %}

> :warning:  Suggestion providers (or factories) supplied with `@SuggestWith` must provide a no-arg constructor, as Lamp will have to construct them reflectively.&#x20;
>
> In the future, Lamp will try different things (`instance`, `getInstance`, `INSTANCE`, singleton methods, etc.) for providing instances, but in the meantime, this is what you have to do.
