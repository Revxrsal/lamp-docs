---
icon: syringe
description: >-
  This page explains how to use dependency injection, which provides a way to
  inject dependencies (instances of objects) into command classes
---

# Dependency injection

Lamp supports a simple form of dependency injection, allowing you to manage dependencies with ease. This feature simplifies the process of injecting dependencies into your command classes.

### Adding Dependencies

Dependencies are added to the `Lamp` instance using the `builder.dependency(...)` method. You can provide dependencies as either a supplier or an object.

{% tabs %}
{% tab title="Java" %}
```java
Lamp lamp = Lamp.builder()
    .dependency(questManager) // Adding a constant dependency
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = Lamp.builder()
    .dependency(questManager)
    .build()
```
{% endtab %}
{% endtabs %}

### Injecting Dependencies

To inject dependencies into fields, use the `@Dependency` annotation. This annotation tells the framework to inject the specified dependency into the annotated field.

{% tabs %}
{% tab title="Java" %}
```java
@CommandPermission("quests.command")
public class QuestCommands {

    @Dependency
    private QuestManager questManager; // Dependency will be injected here

    @Command("quest create")
    public void createQuest(CommandSender sender, String name, String description) {
        // Use questManager to create a quest
    }
    
    ...
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@CommandPermission("quests.command")
class QuestCommands {

    @Dependency
    private lateinit var questManager: QuestManager // Dependency will be injected here

    @Command("quest create")
    fun createQuest(sender: CommandSender, name: String, description: String) {
        // Use questManager to create a quest
    }
    
    ...
}

```
{% endtab %}
{% endtabs %}
