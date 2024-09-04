---
icon: vial
description: This page will explain how you can create and resolve custom parameter types
---

# Custom parameter types

One of the core features of Lamp is the ability to use custom parameter types for commands. This allows us to use types with specific meanings, restrict values to certain options, or provide customized tab completions.

We will illustrate this with a simple _**Quests**_ plugin.

### Creating a Quest type

Let's create a `Quest` class that contains all relevant data for a single `Quest`. Because this is slightly irrelevant to our end goal, we will use a relatively simple implementation.

{% tabs %}
{% tab title="Java" %}
```java
public record Quest(
        String id,
        String description
) {}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
data class Quest(
    val id: String,
    val description: String
)
```
{% endtab %}
{% endtabs %}

We will create a class that handles, stores, and retrieves all Quest objects. Let's call it `QuestManager`. It will contain basic functionality for creating, updating, querying and deleting quests

### Creating a QuestManager

{% tabs %}
{% tab title="Java" %}
```java
public final class QuestManager {

    private final Map<String, Quest> quests = new HashMap<>();

    public boolean questExists(@NotNull String name) {
        return quests.containsKey(name);
    }

    public void add(@NotNull Quest quest) {
        if (questExists(quest.name()))
            throw new IllegalArgumentException("Quest with name '" + quest.name() + "' already exists!");
        quests.put(quest.name(), quest);
    }

    public Quest remove(@NotNull Quest quest) {
        return quests.remove(quest.name());
    }

    public void clearAllQuests() {
        quests.clear();
    }
    
    public Quest quest(@NotNull String name) {
        return quests.get(name);
    }
    
    public Map<String, Quest> quests() {
        return quests;
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class QuestManager {
    
    val quests = mutableMapOf<String, Quest>()

    fun questExists(name: String): Boolean {
        return quests.containsKey(name)
    }

    fun add(quest: Quest) {
        require(!questExists(quest.name)) { 
            "Quest with name '" + quest.name + "' already exists!" 
        }
        quests[quest.name] = quest
    }

    fun remove(quest: Quest): Quest? {
        return quests.remove(quest.name)
    }

    fun clearAllQuests() {
        quests.clear()
    }

    fun quest(name: String): Quest? {
        return quests[name]
    }
}
```
{% endtab %}
{% endtabs %}

We will create a single instance of this QuestManager in our main class.

{% tabs %}
{% tab title="Java" %}
```diff
public final class QuestsPlugin extends JavaPlugin {

+    private final QuestManager questManager = new QuestManager();

}
```
{% endtab %}

{% tab title="Kotlin" %}
<pre class="language-diff"><code class="lang-diff">class QuestsPlugin : JavaPlugin() {
    
<strong>+    private val questManager = QuestManager()
</strong>
}
</code></pre>
{% endtab %}
{% endtabs %}

Now, let's tell Lamp how to resolve a `Quest` parameter.&#x20;

To create custom parameter types, we must implement the `ParameterType` interface. This interface describes how parameters are resolved and what suggestions they receive by default.

### Creating a QuestParameterType

{% tabs %}
{% tab title="Java" %}
```java
public final class QuestParameterType implements ParameterType<BukkitCommandActor, Quest> {

    @Override
    public Quest parse(@NotNull MutableStringStream input, @NotNull ExecutionContext<BukkitCommandActor> context) {
        /* Resolve a Quest here */
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class QuestParameterType : ParameterType<BukkitCommandActor, Quest> {
    
    override fun parse(input: MutableStringStream, context: ExecutionContext<BukkitCommandActor>): Quest {
        /* Resolve a Quest here */
    }
}
```
{% endtab %}
{% endtabs %}

You may have noticed that `ParameterType` contains generics. In fact, it requires that you define two generics when you use it:

* `A`: A subclass of `CommandActor` that the ParameterType can work with. If we are creating a general ParameterType that works with _any_ platform, we can have this as the `CommandActor` interface. If we, however, need a ParameterType that only works with Bukkit, for example, this would be `BukkitCommandActor` or any of its subclasses. \
  \
  _In other words, what is the most general CommandActor implementation we can work with?_
* `T`: The type of object we need to resolve. In this case, it is a `Quest` type. This is used by `#parse(...)` to dictate what types we are expected to return.

We would like to resolve our objects from our QuestManager. For this, let's create a constructor that receives a QuestManager:

{% tabs %}
{% tab title="Java" %}
```java
private final QuestManager questManager;

public QuestParameterType(QuestManager questManager) {
    this.questManager = questManager;
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class QuestParameterType(private val questManager: QuestManager) /* ... */
```
{% endtab %}
{% endtabs %}

Let's create a simple implementation of our `parse` function:

{% tabs %}
{% tab title="Java" %}
<pre class="language-java"><code class="lang-java">@Override
public Quest parse(@NotNull MutableStringStream input, @NotNull ExecutionContext&#x3C;BukkitCommandActor> context) {
<strong>    String name = input.readString();
</strong>    Quest quest = questManager.quest(name);
    if (quest == null)
        throw new CommandErrorException("No such quest: " + name);
    return quest;
}
</code></pre>
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
override fun parse(input: MutableStringStream, context: ExecutionContext<BukkitCommandActor>): Quest {
    val name = input.readString()
    val quest = questManager.quest(name) 
        ?: throw CommandErrorException("No such quest: $name")
    return quest
}
```
{% endtab %}
{% endtabs %}

Let's break down what we are doing here:

* `input.readString()`: This consumes a string from a MutableStringStream. You can think of MutableStringStream as a String that is tracked by a cursor that moves along that string. When we read something from it, we move the cursor forward and receive the value that the cursor passed over. `readString()` will consume an entire string token. This means that if the user gives a value enclosed by double-quotations, for example, `"Hello world"`, this will return `Hello world`. Otherwise, this will consume the next string until it finds a space.\
  \
  A `ParameterType` is free to consume as much of a MutableStringStream as it needs. The only requirement is that if it consumes a token, it must consume it _entirely_. It cannot consume part of a string, for example.
* `throw new CommandErrorException("No such quest: " + name)`: This will signal that the command execution failed, and tell the user that they supplied an invalid quest.

### Adding tab completion to our Quest type

A nicety in ParameterType is that it allows us to define custom suggestions for our quest type. These can be supplied using the `defaultSuggestions` method:

{% tabs %}
{% tab title="Java" %}
```java
@Override public @NotNull SuggestionProvider<BukkitCommandActor> defaultSuggestions() {
    return (input, context) -> List.copyOf(questManager.quests().keySet());
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
override fun defaultSuggestions(): SuggestionProvider<BukkitCommandActor> {
    return SuggestionProvider { _, _ -> questManager.quests.keys.toList() }
}
```
{% endtab %}
{% endtabs %}

### Registering our QuestParameterType to Lamp

Let's create our Lamp instance:

{% tabs %}
{% tab title="Java" %}
```java
@Override public void onEnable() {
    var lamp = BukkitLamp.builder(this)
        .parameterTypes(builder -> {
            builder.addParameterType(Quest.class, new QuestParameterType(questManager));
        })
        .build();
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
override fun onEnable() {
    val lamp = BukkitLamp.builder(this)
        .parameterTypes {
            it.addParameterType(Quest::class.java, QuestParameterType(questManager))
        }
        .build()
}
```
{% endtab %}
{% endtabs %}

That's it! We can now use `Quest` in our commands to our heart's delight.

> :bulb: You may have noticed that the `builder` provides `addXXX` and `addXXXLast`. Why the two variants?
>
> When you use the `addXXXLast` variant, you are essentially giving your ParameterType less priority over others. When two ParameterTypes, one registered with `addXXX` while the other with `addXXXLast` conflict, the `addXXX` one will be used.
>
> Using `addXXXLast` is very useful if you want to leave area for later overriding. Lamp uses it under the hood to provide all default ParameterTypes, which means you can override any of the default ones easily.

### Creating our QuestCommands class

Let's create a simple QuestCommands class:

{% tabs %}
{% tab title="Java" %}
```java
public class QuestCommands {}
```

And register it:

```java
lamp.register(new QuestCommands());
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class QuestCommands
```

And register it:

```kotlin
lamp.register(QuestCommands());
```
{% endtab %}
{% endtabs %}

&#x20;We need to access this QuestManager from our command class. How are we going to do this?

We have multiple solutions:

* **Pass it to the constructor**: It is the traditional Java way of doing things. It involves no magic and no overhead. It, however, creates a tightly-coupled class that&#x20;
* **Create a Lamp dependency**: This is the way Lamp encourages. It is a simple form of [dependency injection](https://stackoverflow.com/questions/130794/what-is-dependency-injection) that allows us to create loosely coupled code. In simpler terms, it says "_I want a QuestManager. I don't care where this QuestManager comes from. I just want it_"

We will go with the second way. It will keep our code clean and easy for future refactoring. Dependency injection also comes with many benefits, and this is not the place to discuss them. You can read up on the topic for more details.

To create a dependency, we must register it in our `Lamp` instance as follows:

{% tabs %}
{% tab title="Java" %}
```java
var lamp = BukkitLamp.builder(this)
    // ...
    .dependency(QuestManager.class, questManager)
    // ...
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = BukkitLamp.builder(this)
    // ...
    .dependency(QuestManager::class.java, questManager)
    // ...
    .build()
```
{% endtab %}
{% endtabs %}

Now, to use it in our `QuestsCommand` class:

{% tabs %}
{% tab title="Java" %}
```java
public class QuestCommands {

    @Dependency
    private QuestManager questManager;

}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class QuestCommands {
 
    @Dependency
    private lateinit var questManager: QuestManager

}
```
{% endtab %}
{% endtabs %}

That's it! We can now create our Quest commands easily. And we have a `QuestManager` at our disposal for all Quest-related operations.

{% tabs %}
{% tab title="Java" %}
```java
@Command("quest")
@CommandPermission("quests.command")
public class QuestCommands {

    @Dependency 
    private QuestManager questManager;

    @Subcommand("create")
    public void createQuest(BukkitCommandActor actor, String name, String description) {
        /*...*/
    }

    @Subcommand("delete")
    public void deleteQuest(BukkitCommandActor actor, Quest quest) {
        /*...*/
    }
    
    @Subcommand("start")
    public void startQuest(Player actor, Quest quest) {
        /*...*/
    }

    @Subcommand("clear")
    public void clearQuests(BukkitCommandActor actor) {
        /*...*/
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("quest")
@CommandPermission("quests.command")
class QuestCommands {

    @Dependency
    private lateinit var questManager: QuestManager

    @Subcommand("create")
    fun createQuest(actor: BukkitCommandActor, name: String, description: String) {
        /*...*/
    }

    @Subcommand("delete")
    fun deleteQuest(actor: BukkitCommandActor, quest: Quest) {
        /*...*/
    }

    @Subcommand("start")
    fun startQuest(actor: Player, quest: Quest) {
        /*...*/
    }

    @Subcommand("clear")
    fun clearQuests(actor: BukkitCommandActor) {
        /*...*/
    }
}
```
{% endtab %}
{% endtabs %}

### Using a ParameterType Factory

`ParameterType.Factory` allows you to dynamically create `ParameterType` instances based on the type of parameter and annotations. This is particularly useful for complex parameter parsing scenarios. Below is an example demonstrating how to create a custom factory for handling enum types.

#### Example: Enum Parameter Type Factory

This example shows how to implement a `ParameterType.Factory` that handles enum types. The factory converts a string input into an enum constant and provides suggestions based on enum names.

{% tabs %}
{% tab title="Java" %}
```java
public enum EnumParameterTypeFactory implements ParameterType.Factory<CommandActor> {
    INSTANCE;

    @Override
    @SuppressWarnings({"rawtypes", "unchecked"})
    public <T> ParameterType<CommandActor, T> create(@NotNull Type parameterType, @NotNull AnnotationList annotations, @NotNull Lamp<CommandActor> lamp) {
        Class<?> rawType = getRawType(parameterType);
        if (!rawType.isEnum())
            return null;
        Enum<?>[] enumConstants = (Enum<?>[]) rawType.getEnumConstants();
        Map<String, Enum<?>> byKeys = new HashMap<>();
        List<String> suggestions = new ArrayList<>();
        for (Enum<?> enumConstant : enumConstants) {
            String name = enumConstant.name().toLowerCase();
            byKeys.put(name, enumConstant);
            suggestions.add(name);
        }
        return new EnumParameterType(byKeys, suggestions);
    }

    private record EnumParameterType<E extends Enum<E>>(
            Map<String, E> byKeys,
            List<String> suggestions
    ) implements ParameterType<CommandActor, E> {

        @Override
        public E parse(@NotNull MutableStringStream input, @NotNull ExecutionContext<CommandActor> context) {
            String key = input.readUnquotedString();
            E value = byKeys.get(key.toLowerCase());
            if (value != null)
                return value;
            throw new EnumNotFoundException(key);
        }

        @Override
        public @NotNull SuggestionProvider<CommandActor> defaultSuggestions() {
            return SuggestionProvider.of(suggestions);
        }

        @Override
        public @NotNull PrioritySpec parsePriority() {
            return PrioritySpec.highest();
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
object EnumParameterTypeFactory : ParameterType.Factory<CommandActor> {
    
    @Suppress("UNCHECKED_CAST")
    override fun <T> create(parameterType: Type, annotations: AnnotationList, lamp: Lamp<CommandActor>): ParameterType<CommandActor, T>? {
        val rawType = getRawType(parameterType)
        if (!rawType.isEnum) return null
        val enumConstants = rawType.enumConstants as Array<Enum<*>>
        val byKeys = mutableMapOf<String, Enum<*>>()
        val suggestions = mutableListOf<String>()
        for (enumConstant in enumConstants) {
            val name = enumConstant.name.lowercase()
            byKeys[name] = enumConstant
            suggestions.add(name)
        }
        return EnumParameterType(byKeys, suggestions) as ParameterType<CommandActor, T>
    }

    private data class EnumParameterType<E : Enum<E>>(
        val byKeys: Map<String, E>,
        val suggestions: List<String>
    ) : ParameterType<CommandActor, E> {

        override fun parse(input: MutableStringStream, context: ExecutionContext<CommandActor>): E {
            val key = input.readUnquotedString()
            return byKeys[key.lowercase()] ?: throw EnumNotFoundException(key)
        }

        override fun defaultSuggestions(): SuggestionProvider<CommandActor> {
            return SuggestionProvider.of(suggestions)
        }

        override fun parsePriority(): PrioritySpec {
            return PrioritySpec.highest()
        }
    }
}
```
{% endtab %}
{% endtabs %}

This example demonstrates how to create a `ParameterType.Factory` that can parse enums and provide suggestions based on the enum values.

Well done! In this tutorial, we have learned the following:

* How to create a custom parameter type
* How to provide default suggestions for a parameter type
* How to create and use dependencies

In the next tutorial, we will go through suggestions and auto-completion
