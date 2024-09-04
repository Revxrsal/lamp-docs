---
description: >-
  This page explains how to use the CommandCondition interface to restrict
  command execution based on custom conditions
icon: hexagon-check
---

# Command conditions

The `CommandCondition` interface allows you to define conditions that must be met for a command invocation to proceed. This is useful for performing checks based on specific annotations or other criteria to control command execution flow.

### Implementing a Custom `CommandCondition`

In this example, we will create a custom condition that checks if the command actor is an operator (op). We will use a custom annotation `@IsOpped` to mark commands that should only be executable by ops. If the condition is not met, the condition will throw an exception, preventing the command from executing.

#### Step 1: Define the `@IsOpped` Annotation

The `@IsOpped` annotation will be used to mark commands that require the actor to be an operator:

{% tabs %}
{% tab title="Java" %}
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface IsOpped {}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FUNCTION)
annotation class IsOpped
```
{% endtab %}
{% endtabs %}

#### Step 2: Implement the `IsOppedCondition`

Next, we create a `CommandCondition` implementation that checks if the actor is an operator by inspecting the `@IsOpped` annotation:

{% tabs %}
{% tab title="Java" %}
```java
public enum IsOppedCondition implements CommandCondition<BukkitCommandActor> {
    INSTANCE;

    @Override
    public void test(@NotNull ExecutionContext<BukkitCommandActor> context, @NotNull StringStream input) {
        boolean requiresOp = context.command().annotations().contains(IsOpped.class);
        if (requiresOp && !context.actor().sender().isOp()) {
            throw new CommandErrorException("You must be an operator to execute this command.");
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
object IsOppedCondition : CommandCondition<BukkitCommandActor> {

    override fun test(context: ExecutionContext<BukkitCommandActor>, input: StringStream) {
        val requiresOp = context.command().annotations().contains(IsOpped::class.java)
        if (requiresOp && !context.actor().sender().isOp()) {
            throw CommandErrorException("You must be an operator to execute this command.")
        }
    }
}

```
{% endtab %}
{% endtabs %}

#### Step 3: Register the Condition

To use the `IsOppedCondition`, you need to register it with your `Lamp` instance, specifying that it should be used whenever the `@IsOpped` annotation is present:

{% tabs %}
{% tab title="Java" %}
```java
var lamp = BukkitLamp.builder(this)
    .condition(IsOppedCondition.INSTANCE)
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = BukkitLamp.builder(this)
    .condition(IsOppedCondition)
    .build()
```
{% endtab %}
{% endtabs %}

#### Step 4: Applying the `@IsOpped` Annotation to Commands

Now, you can apply the `@IsOpped` annotation to any command method that should only be accessible by operators:

{% tabs %}
{% tab title="Java" %}
```java
@IsOpped
@Command("some admin command")
public void opOnlyCommand(Player player) {
    /* ... */
}
```
{% endtab %}

{% tab title="Kotlin" %}
<pre class="language-kotlin"><code class="lang-kotlin"><strong>@IsOpped
</strong><strong>@Command("some admin command")
</strong>fun opOnlyCommand(Player player) {
    /* ... */
}
</code></pre>
{% endtab %}
{% endtabs %}

In this example, the `opOnlyCommand` method is marked with the `@IsOpped` annotation, indicating that it requires the actor to be an operator. If a non-operator attempts to use this command, the `IsOppedCondition` will throw a `CommandErrorException`, and the command framework will handle the exception appropriately.
