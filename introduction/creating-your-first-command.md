---
description: We will create our very first command
icon: terminal
---

# Creating your first command

In this page, you will learn how to create commands using annotations.

* We will be using the **Bukkit** platform, as it is the most widely used platform for Lamp. However, the same concepts apply to all other platforms.

### A /greet command

{% tabs %}
{% tab title="Java" %}
```java
public class GreetCommands {

    @Command("greet")
    public void greet(BukkitCommandActor actor) {
        actor.reply("Hello, " + actor.name() + "!");
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class GreetCommands {
    
    @Command("greet")
    fun greet(actor: BukkitCommandActor) {
        actor.reply("Hello, " + actor.name() + "!")
    }
}
```
{% endtab %}
{% endtabs %}

That's it! We have our own /greet command. Let's register it:

{% tabs %}
{% tab title="Java" %}
```java
public final class TestPlugin extends JavaPlugin {

    @Override
    public void onEnable() {
        var lamp = BukkitLamp.builder(this).build();
        lamp.register(new GreetCommands());
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class TestPlugin : JavaPlugin() {

    override fun onEnable() {
        val lamp = BukkitLamp.builder(this).build()
        lamp.register(GreetCommands())
    }
}
```
{% endtab %}
{% endtabs %}

\
Let's see Lamp in action:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>/greet in Minecraft (PaperSpigot 1.21)</p></figcaption></figure>

Great! Let's buff our command with more arguments.&#x20;

We will add a `target` argument, which is a player. When executed, the target player will receive a welcome message.

{% tabs %}
{% tab title="Java" %}
```java
public class GreetCommands {

    @Command("greet")
    public void greet(BukkitCommandActor actor, Player target) {
        target.sendMessage("Welcome, " + target.getName() + "!");
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class GreetCommands {

    @Command("greet")
    fun greet(actor: BukkitCommandActor, target: Player) {
        target.sendMessage("Welcome, ${target.name}!")
    }
}
```
{% endtab %}
{% endtabs %}

Which will generate the following command:

<figure><img src="../.gitbook/assets/Screenshot 2024-08-27 235810.png" alt=""><figcaption></figcaption></figure>

Running it, we get:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>Executing /greet Revxrsal</p></figcaption></figure>

Notice that by specifying the Player type, we automatically get tab completions for free! Also, if we try to input an invalid player, we get the following error message:&#x20;

<img src="../.gitbook/assets/image (4).png" alt="" data-size="original">



This is the beauty of Lamp! You describe the command and write the code that needs to be executed. Everything else, like auto-completions, validation, and parsing is done under the hood.

> :bulb: It may seem like magic that Lamp knows how to parse the \`org.bukkit.entity.Player\` type. Not much magic is involved, though! Lamp has built-in support for the following types:
>
> * java.lang.String
> * long, int, short, byte, double, and float
> * boolean
> * java.util.UUID
> * Any `enum` type
>
> The following platform-specific types are also supported:
>
> * org.bukkit.World
> * org.bukkit.entity.Player
> * org.bukkit.OfflinePlayer&#x20;
> * `EntitySelector<SomeEntityType>`, which is a Lamp-provided type for matching `@p`, `@r`, `@e[type=cow,distance=10]`, etc.
>
> And, good news! For all the types above, you get `List<T>`, `Set<T>` and `T[]` types for free. The same applies for any parameter type you register, which we will do in the next tutorials.
