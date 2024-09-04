---
icon: fishing-rod
description: >-
  This page explains how to use hooks, which allow for injecting custom logic
  before or after command registration, unregistration, or execution.
---

# Hooks

In Lamp, hooks provide a powerful way to interact with the command lifecycle, allowing you to execute custom logic at various points before and after command operations. You can use hooks to perform actions or modify behavior related to command registration, unregistration, and execution.

#### Available Hooks

There are three main types of hooks you can use:

1. **Command Registered Hook**: Invoked when a command is about to be registered.
   * **Interface**: `CommandRegisteredHook<A extends CommandActor>`
   * **Method**: `void onRegistered(@NotNull ExecutableCommand<A> command, @NotNull CancelHandle cancelHandle)`
2. **Command Unregistered Hook**: Invoked when a command is about to be unregistered.
   * **Interface**: `CommandUnregisteredHook<A extends CommandActor>`
   * **Method**: `void onUnregistered(@NotNull ExecutableCommand<A> command, @NotNull CancelHandle cancelHandle)`
3. **Command Executed Hook**: Invoked when a command is about to be executed.
   * **Interface**: `CommandExecutedHook<A extends CommandActor>`
   * **Method**: `void onExecuted(@NotNull ExecutableCommand<A> command, @NotNull ExecutionContext<A> context, @NotNull CancelHandle cancelHandle)`

#### Cancel Handle

Each hook method may receive a `CancelHandle` parameter which can be used to cancel the command operation if needed:

* **Interface**: `CancelHandle`
* **Methods**:
  * `boolean wasCancelled()`
  * `void cancel()`

#### Configuring Hooks

Hooks are managed through an immutable registry. You can configure hooks using `Hooks.Builder`, which is available in `Lamp.Builder`. For example:

```java
var lamp = Lamp.builder()
    .hooks(hooks -> {
        hooks.onCommandExecuted(new MyCommandExecutedHook());
        // Add other hooks as needed
    })
    .build();
```

#### Example: Command Executed Hook

Here's an example of a `CommandExecutedHook` that prints a message to the console when a command is executed:

{% tabs %}
{% tab title="Java" %}
```java
public class MyCommandExecutedHook implements CommandExecutedHook<CommandActor> {

    @Override
    public void onExecuted(@NotNull ExecutableCommand<CommandActor> command, @NotNull ExecutionContext<CommandActor> context, @NotNull CancelHandle cancelHandle) {
        System.out.println("Command executed: " + command.path());
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class MyCommandExecutedHook : CommandExecutedHook<CommandActor> {

    override fun onExecuted(command: ExecutableCommand<CommandActor>, context: ExecutionContext<CommandActor>, cancelHandle: CancelHandle) {
        println("Command executed: ${command.path()}")
    }
}
```
{% endtab %}
{% endtabs %}

This hook will output a message to the console every time a command is executed, allowing you to track or log command executions easily.
