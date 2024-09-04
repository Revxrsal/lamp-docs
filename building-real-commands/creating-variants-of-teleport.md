---
icon: transporter-1
description: >-
  This page will explain how we can use Lamp to create multiple variants of the
  /teleport command
---

# Creating variants of /teleport

The /greet command we built in the last section was relatively simple, and enough to showcase basic command creation in Lamp.&#x20;

In this section, however, we will build more complicated commands that simulate real-life cases. Tons of fun awaits us!



## Creating variants of /teleport

The /teleport command in Minecraft is a good example of a multi-functional command. In these examples, we will build the following:

* `/teleport <x> <y> <z>`
* `/teleport <target> <x> <y> <z>`
* `/teleport <target> here`
* `/teleport <to target>`

We will also define them with the `/tp` as a shorter form.

We will start with implementing the core functionality. To keep things simple for now, we will not add fancy features like `~` and `^` (relative coordinates and angles). We will introduce them in later sections, however.



Let's start by creating a separate class for containing these commands:

{% tabs %}
{% tab title="Java" %}
```java
public class TeleportCommands {

}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class TeleportCommands {

}
```
{% endtab %}
{% endtabs %}

And register it to our `Lamp` instance:

{% tabs %}
{% tab title="Java" %}
```java
public final class TestPlugin extends JavaPlugin {

    @Override public void onEnable() {
        var lamp = BukkitLamp.builder(this).build();
        lamp.register(new TeleportCommands());
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class TestPlugin : JavaPlugin() {

    override fun onEnable() {
        val lamp = BukkitLamp.builder(this).build()
        lamp.register(TeleportCommands())
    }
}
```
{% endtab %}
{% endtabs %}

### `/teleport <x> <y> <z>`

This command should be easy to create. Let's define our function:

{% tabs %}
{% tab title="Java" %}
```java
public class TeleportCommands {

    @Command({"teleport", "tp"})
    public void teleport(Player sender, double x, double y, double z) {
        Location location = new Location(sender.getWorld(), x, y, z);
        sender.teleport(location);
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class TeleportCommands {
    
    @Command("teleport", "tp")
    fun teleport(sender: Player, x: Double, y: Double, z: Double) {
        val location = Location(sender.world, x, y, z)
        sender.teleport(location)
    }
}
```
{% endtab %}
{% endtabs %}

Let's break this down:

* `@Command({"teleport", "tp"})`: This command defines our command as /teleport and /tp.
  * Any aliases defined&#x20;
* `Player sender`: This is the argument that represents the player executing the command.
  * Because this is the first parameter in the method, Lamp will implicitly infer it as the command sender
  * Because it is a `Player`, any non-player entity attempting to execute this command will receive a `You must be a player to use this command!`-like error. This makes it easy to restrict certain commands to player senders only.
* `double x, double y, double z`: These are the arguments that our command will receive. Lamp will automatically parse the user input and parse it into `double`s, or emit errors if the user inputs an invalid value.

Let's try our command:

<figure><img src="https://i.gyazo.com/de2e8ea2e3249df6dab97ee3d1415d33.gif" alt=""><figcaption><p>Executing /teleport</p></figcaption></figure>

That's one variant down. Let's create another.

### &#x20;`/teleport <target> <x> <y> <z>`

{% tabs %}
{% tab title="Java" %}
```java
@Command({"teleport", "tp"})
public void teleport(Player sender, EntitySelector<LivingEntity> target, double x, double y, double z) {
    Location location = new Location(sender.getWorld(), x, y, z);
    for (LivingEntity entity : target)
        entity.teleport(location);
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("teleport", "tp")
fun teleport(sender: Player, target: EntitySelector<LivingEntity>, x: Double, y: Double, z: Double) {
    val location = Location(sender.world, x, y, z)
    for (entity in target) 
        entity.teleport(location)
}
```
{% endtab %}
{% endtabs %}

Note that our commands with similar signatures can co-exist peacefully with no problems.

![](<../.gitbook/assets/image (7).png>)

We can have as many variants of /teleport as we want, as long as Lamp can actually _differentiate_ between them.&#x20;

When there are multiple candidates for commands, Lamp will try to find the best one. This method is not foolproof and may go wrong in rare cases of real confusion. &#x20;

At the end of the page, we will go through the criteria Lamp uses to decide the best execution candidate.

### `/teleport <target> here`

Now, we will implement `/teleport <target> here`. This command is slightly different from the ones above as it involves an argument in the middle of the command.&#x20;

We noticed that, in previous commands, arguments would always come at the end of the command, in the same order they are defined. However, we can declare the order in the command annotations as needed. And, as expected, if a parameter is not defined in the command path, it will be put at the end of the command.

> :bulb: To define an argument in the middle of the command, simply declare it in the command annotation, enclosed with `<>`. For example

{% tabs %}
{% tab title="Java" %}
```java
@Command("teleport <target> here")
public void teleportHere(Player sender, EntitySelector<LivingEntity> target) {
    for (LivingEntity entity : target)
        entity.teleport(sender);
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("teleport <target> here")
fun teleportHere(sender: Player, target: EntitySelector<LivingEntity>) {
    for (entity in target)
        entity.teleport(sender)
}
```
{% endtab %}
{% endtabs %}

When Lamp encounters a name enclosed by `<>`, it will automatically infer it as a parameter name and look for a parameter with that name.

> :warning:  **Important note**: You need to [enable parameter names ](../#optional-preserve-parameter-names)for this, or define an `@Named` annotation on the method's parameters. Lamp will throw an exception if it cannot find the parameter.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>Using <code>/teleport &#x3C;target> here</code> command</p></figcaption></figure>

Let's create the simple `/teleport <to>`&#x20;

### /teleport \<to>

This one is relatively simple too:

{% tabs %}
{% tab title="Java" %}
```java
@Command({"teleport", "tp"})
public void teleport(Player sender, Entity target) {
    sender.teleport(target);
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("teleport", "tp")
fun teleport(sender: Player, target: Entity) {
    sender.teleport(target)
}
```
{% endtab %}
{% endtabs %}

That's it! We can see how Lamp can work with a single command that has multiple variants, and how we can define arguments that come in different places of the command.



In the next
