---
icon: joystick
---

# Customizing the dispatcher and failure behavior

Lamp's strategy for finding the best command is not fail-proof, and, because it relies on user input, it may not always be able to find the error cause spot on (simply put, it may not exist, or there may be many!)

Therefore, Lamp allows you to tweak the resolution strategy, and change the failure behavior. This is done using the `DispatcherSettings` class, which you can customize using `Lamp.Builder#dispatcherSettings()`.

## DispatcherSettings

The `DispatcherSettings` class allows you to customize the following:

1. The maximum number of commands it should try
2. The failure behavior that receives all the failed attempts (`Potential`s) and the user input. This is done using the `FailureHandler` interface.

### Example

{% tabs %}
{% tab title="Java" %}
```java
var builder = ...;
builder.dispatcherSettings(settings -> {
    settings.maximumFailedAttempts(10).failureHandler((actor, failedAttempts, input) -> {
        ...
    });
});
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val builder = ...
builder.dispatcherSettings { settings ->
    settings.maximumFailedAttempts(10).failureHandler { actor, failedAttempts, input ->
    
    }
}
```
{% endtab %}
{% endtabs %}
