---
icon: user-slash
description: >-
  Command permissions allow you to restrict access to commands based on user
  permissions
---

# Command permissions

The `CommandPermission` interface is used to define permissions required to execute a command. It is a functional interface that evaluates whether a command can be executed by a given `CommandActor`. This implementation may vary depending on the target platform.

#### Interface Methods

*   **`boolean isExecutableBy(@NotNull A actor)`**

    Determines whether the specified `actor` has permission to execute the command.

#### Factory Interface

The `CommandPermission.Factory` interface allows you to create custom `CommandPermission` implementations based on annotations.

*   **`@Nullable CommandPermission<A> create(@NotNull AnnotationList annotations, @NotNull Lamp<A> lamp)`**

    Creates a new `CommandPermission` based on the provided list of annotations and the `Lamp` instance. If the factory does not handle the given input, it may return `null`.

## Platform-Specific Command Permissions

In addition to the generic `CommandPermission` interface, various platforms have their own specialized `@CommandPermission` annotations for handling permissions. This allows you to define and manage permissions specific to the platform you are working with.

## Custom permission annotations

### Example: Custom Command Permission in Bukkit

#### 1. Create a Custom Annotation

First, define a custom annotation to specify the required group:

{% tabs %}
{% tab title="Java" %}
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RequiresGroup {
    String value();
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FUNCTION)
annotation class RequiresGroup(val value: String)
```
{% endtab %}
{% endtabs %}

#### 2. Implement a Custom CommandPermission.Factory

Next, create a `CommandPermission.Factory` that checks if the `CommandActor` has the required group:

{% tabs %}
{% tab title="Java" %}
```java
public class GroupPermissionFactory implements CommandPermission.Factory<BukkitCommandActor> {

    @Override
    @Nullable
    public CommandPermission<BukkitCommandActor> create(@NotNull AnnotationList annotations, @NotNull Lamp<BukkitCommandActor> lamp) {
        RequiresGroup requiresGroup = annotations.get(RequiresGroup.class);
        if (requiresGroup == null) return null;

        String requiredGroup = requiresGroup.value();

        return actor -> RankManager.getRank(actor.sender()).equals(requiredGroup);
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class GroupPermissionFactory : CommandPermission.Factory<BukkitCommandActor> {

    @Nullable
    override fun create(annotations: AnnotationList, lamp: Lamp<BukkitCommandActor>): CommandPermission<BukkitCommandActor>? {
        val requiresGroup = annotations.get(RequiresGroup::class.java) ?: return null
        val requiredGroup = requiresGroup.value

        return CommandPermission { actor ->
            RankManager.getRank(actor.sender) == requiredGroup
        }
    }
}

```
{% endtab %}
{% endtabs %}

#### 3. Register the Factory with Lamp

Register your custom `CommandPermission.Factory` with the `Lamp` instance:

{% tabs %}
{% tab title="Java" %}
```java
var lamp = BukkitLamp.builder(this)
    .permissionFactory(new GroupPermissionFactory())
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = BukkitLamp.builder(this)
    .permissionFactory(GroupPermissionFactory())
    .build()
```
{% endtab %}
{% endtabs %}

#### 4. Use the Annotation in a Command

Finally, use the `@RequiresGroup` annotation in your command method:

{% tabs %}
{% tab title="Java" %}
```java
public class MyCommands {

    @RequiresGroup("admin")
    @Command("admin command")
    public void adminCommand(BukkitCommandActor actor) {
        actor.reply("You are authorized to use this command!")
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class MyCommands {

    @RequiresGroup("admin")
    @Command("admin command")
    fun adminCommand(actor: BukkitCommandActor) {
        actor.reply("You are authorized to use this command!")
    }
}
```
{% endtab %}
{% endtabs %}

In this setup:

* The `GroupPermissionFactory` checks if the sender’s rank matches the required group specified in the `@RequiresGroup` annotation.
* If the sender is authorized, they can execute the command; otherwise, they will be denied access.
