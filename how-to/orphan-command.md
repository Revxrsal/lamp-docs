---
icon: tree
description: >-
  This page explains how to use orphan commands, which are commands defined
  without the @Command annotation and whose path will be determined dynamically
  at runtime.
---

# Orphan command

Due to the nature of annotations, all commands and subcommands in a typical command framework must have known paths at compile-time. This requirement makes it impossible to create commands whose paths are dynamically determined, such as those sourced from configuration files or user input. The `OrphanCommand` API solves this limitation by allowing commands to have paths that are resolved at runtime.

### Overview

#### What is an `OrphanCommand`?

An `OrphanCommand` is a command whose parent or path is not known at compile-time. Implementing the `OrphanCommand` interface signals to the framework (Lamp) that the command's path will be dynamically determined and should be resolved at runtime.

#### How to Register an `OrphanCommand`

To register an `OrphanCommand`, the command must be wrapped using the `Orphans` utility. This utility allows specifying the command path dynamically and is similar to using the `@Command` annotation but without the compile-time constraints.

#### Example Usage

1. **Implement the `OrphanCommand` Interface**: Create a class that implements `OrphanCommand` and define your command methods within it.
2. **Register the Orphan Command**: Use the `Orphans.path()` method to set the path for the orphan command dynamically.

### Example

#### Step 1: Implementing an `OrphanCommand`

{% tabs %}
{% tab title="Java" %}
<pre class="language-java"><code class="lang-java">public class Foo implements OrphanCommand {
<strong>
</strong>    @CommandPlaceholder 
    // ^^^
    // will get replaced with @Command("the path here")
    // at runtime
    public void onCommand(CommandActor actor) {
        // ...
    }

    @Subcommand("bar")
    public void bar(CommandActor actor) {
        actor.reply("Hello!");
    }
}
</code></pre>
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class Foo : OrphanCommand {
    
    @CommandPlaceholder 
    // ^^^
    // will get replaced with @Command("the path here")
    // at runtime
    fun onCommand(actor: CommandActor) {
        // ...
    }
    
    @Subcommand("bar")
    fun bar(actor: CommandActor) {
        actor.reply("Hello!")
    }
}
```
{% endtab %}
{% endtabs %}

In this example, `Foo` is an orphan command class with a subcommand `bar`.

#### Step 2: Registering the Orphan Command

{% tabs %}
{% tab title="Java" %}
```java
var lamp = ...; // Initialize your Lamp instance

// Dynamically set the command path using Orphans.path()
lamp.register(Orphans.path(args[0]).handler(new Foo()));
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = ... // Initialize your Lamp instance

// Dynamically set the command path using Orphans.path()
lamp.register(Orphans.path(args[0]).handler(Foo()))
```
{% endtab %}
{% endtabs %}

Assuming `args[0]` is `"buzz"`, the above code will register a command with the path `"buzz"`.

**Example Command Execution**

```sh
> buzz bar
Hello!
```

#### `Orphans.path()` Behavior

* `Orphans.path("foo", "bar")` is equivalent to `@Command("foo", "bar")`
* `Orphans.path("foo bar", "buzz boom")` is equivalent to `@Command("foo bar", "buzz boom")`

However, `Orphans.path()` can also accept strings that are not known at compile-time, such as values loaded from configuration files or provided by user input.

### Summary

The `OrphanCommand` interface and `Orphans` utility enable dynamic command path registration, allowing for greater flexibility and configurability in command definitions. This feature is especially useful when command paths need to be determined at runtime, bypassing the compile-time limitations of traditional annotations.
