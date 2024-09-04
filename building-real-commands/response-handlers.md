---
icon: table-tennis-paddle-ball
description: >-
  This page explains how to use response handlers, which process the return
  values of command methods, enabling custom handling of command outputs.
---

# Response handlers

## Using the `ResponseHandler` Interface

The `ResponseHandler` interface is designed for post-processing command responses within a commands framework. It allows you to define custom logic for handling results returned from command methods. This can be particularly useful for processing or formatting responses before they are sent to the command actor.

### Example: Handling Adventure's `Component` Type

In this example, we will create a `ResponseHandler` that processes responses of type `Component` from Adventure. The handler will send the formatted text to the command actor's sender.

#### Custom Response Handler

{% tabs %}
{% tab title="Java" %}
```java
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.TextColor;

public class ComponentResponseHandler implements ResponseHandler<BukkitCommandActor, Component> {

    @Override
    public void handleResponse(Component response, ExecutionContext<BukkitCommandActor> context) {
        // Get the sender from the command context
        var sender = context.actor().sender();

        // Send the Component response to the sender
        sender.sendMessage(response);
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
import net.kyori.adventure.text.Component
import net.kyori.adventure.text.format.TextColor

class ComponentResponseHandler : ResponseHandler<CommandActor, Component> {

    override fun handleResponse(response: Component?, context: ExecutionContext<CommandActor>) {
        // Get the sender from the command context
        val sender = context.actor().sender()

        // Send the Component response to the sender
        sender.sendMessage(response)
    }
}
```
{% endtab %}
{% endtabs %}

#### Registering the Handler

To use the `ComponentResponseHandler`, you need to register it with your command framework:

{% tabs %}
{% tab title="Java" %}
```java
var lamp = BukkitLamp.builder(this)
    .responseHandler(Component.class, new ComponentResponseHandler())
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = BukkitLamp.builder(this)
    .responseHandler(Component::class.java, ComponentResponseHandler())
    .build()
```
{% endtab %}
{% endtabs %}

#### Example Command Method

Here’s an example command method that returns an Adventure `Component`:

{% tabs %}
{% tab title="Java" %}
```java
@Command("hello")
public Component sendMessage() {
    return Component.text("Hello, World!")
        .color(TextColor.color(0, 255, 0)) // Green color
        .append(Component.text(" Click here").color(TextColor.color(255, 0, 0))) // Red color
        .clickEvent(ClickEvent.openUrl("https://example.com"));
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("hello")
fun sendMessage(): Component {
    return Component.text("Hello, World!")
        .color(TextColor.color(0, 255, 0)) // Green color
        .append(Component.text(" Click here").color(TextColor.color(255, 0, 0))) // Red color
        .clickEvent(ClickEvent.openUrl("https://example.com"))
}
```
{% endtab %}
{% endtabs %}

In this example:

* The `sendMessage` method returns a `Component` with rich text formatting.
* The `ComponentResponseHandler` handles the response by sending it to the command actor’s sender.

Here’s a markdown section to include in the documentation, explaining the `ResponseHandler.Factory` interface and providing the example for handling `Optional<T>`:

***

## Dynamic Response Handlers with ResponseHandler.Factory

The `ResponseHandler.Factory` interface allows for the dynamic creation of `ResponseHandler` instances based on the response type and function annotations. This feature enables more flexible and complex response handling scenarios by composing handlers for different types, such as `Optional<T>`, `Supplier<T>`, and others.

#### Interface Method

*   **`@Nullable <T> ResponseHandler<A, T> create(@NotNull Type type, @NotNull AnnotationList annotations, @NotNull Lamp<A> lamp)`**

    Creates a new `ResponseHandler` for the given type and list of annotations. If the factory does not handle the input, it may return `null`.

#### Example: Handling `Optional<T>`

Here’s an example of a `ResponseHandler.Factory` implementation that automatically constructs `ResponseHandler` instances for `Optional<T>` where `T` is any type that has a registered response handler:

{% tabs %}
{% tab title="Java" %}
```java
/**
 * A {@link ResponseHandler.Factory} that creates {@link ResponseHandler}s for
 * {@link Optional {@code Optional<T>}} where T is any type that has
 * a registered response handler.
 */
public enum OptionalResponseHandler implements ResponseHandler.Factory<CommandActor> {

    INSTANCE;

    @Override
    public @Nullable <T> ResponseHandler<CommandActor, T> create(@NotNull Type type, @NotNull AnnotationList annotations, @NotNull Lamp<CommandActor> lamp) {
        Class<?> rawType = getRawType(type);
        if (rawType != Optional.class)
            return null;
        Type suppliedType = getFirstGeneric(type, Object.class);
        ResponseHandler<CommandActor, Object> delegate = lamp.responseHandler(suppliedType, AnnotationList.empty());
        return (response, context) -> {
            //noinspection unchecked
            Optional<Object> optional = (Optional<Object>) response;
            optional.ifPresent(value -> delegate.handleResponse(value, context));
        };
    }
}
```


{% endtab %}

{% tab title="Kotlin" %}
```kotlin
/**
 * A [ResponseHandler.Factory] that creates [ResponseHandler]s for
 * [Optional] where T is any type that has
 * a registered response handler.
 */
object OptionalResponseHandler : ResponseHandler.Factory<CommandActor> {

    @Suppress("UNCHECKED_CAST")
    override fun <T> create(type: Type, annotations: AnnotationList, lamp: Lamp<CommandActor>): ResponseHandler<CommandActor, T>? {
        val rawType = getRawType(type)
        if (rawType != Optional::class.java) {
            return null
        }
        val suppliedType = getFirstGeneric(type, Any::class.java)
        val delegate: ResponseHandler<CommandActor, Any> = lamp.responseHandler(suppliedType, AnnotationList.empty())
        return ResponseHandler<CommandActor, Any> { response, context ->
            val optional = response as? Optional<Any>
            optional?.ifPresent { value -> delegate.handleResponse(value, context) }
        } as ResponseHandler<CommandActor, T>
    }
}

```
{% endtab %}
{% endtabs %}

#### Explanation

* **`getRawType(type)`**: Retrieves the raw type from the generic `Type` (stripped from generics). Checks if it's `Optional`.
* **`getFirstGeneric(type, Object.class)`**: Extracts the first generic type argument from `Optional<T>`.
* **`lamp.responseHandler(suppliedType, AnnotationList.empty())`**: Obtains the appropriate `ResponseHandler` for the type `T`.
* **`optional.ifPresent(value -> delegate.handleResponse(value, context))`**: Passes the contained value to the delegate handler if present.

This factory implementation ensures that any `Optional<T>` response types are handled appropriately by delegating to an existing `ResponseHandler` for `T`, allowing for flexible and dynamic response processing.

**Registration**

{% tabs %}
{% tab title="Java" %}
```java
// Create and configure your Lamp instance with a custom ResponseHandler.Factory
var lamp = Lamp.builder()
    .responseHandler(OptionalResponseHandler.INSTANCE) // Add the factory here
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
// Create and configure your Lamp instance with a custom ResponseHandler.Factory
val lamp = Lamp.builder<CommandActor>()
    .responseHandler(OptionalResponseHandler.INSTANCE) // Add the factory here
    .build()
```
{% endtab %}
{% endtabs %}
