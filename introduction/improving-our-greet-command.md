---
description: In this page, we will make our greet more customizable and robust.
icon: handshake
---

# Improving our greet command

Right now, our /greet command does not do much. Also, it has the following problems:

1. The command has no description that appears in /help
2. The command is executable by everyone
3. The welcome message is not customized

Also, we would like to make the command work with no arguments. If a target was not specified, we want to default to the command sender.

#### **Setting a description**

This can be easily set using the `@Description` annotation

{% tabs %}
{% tab title="Java" %}
```diff
public class GreetCommands {

    @Command("greet")
+    @Description("Greets the specified player")
    public void greet(BukkitCommandActor actor, Player target) {
        target.sendMessage("Welcome, " + target.getName() + "!");
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```diff
class GreetCommands {

    @Command("greet")
+    @Description("Greets the specified player")
    fun greet(actor: BukkitCommandActor, target: Player) {
        target.sendMessage("Welcome, " + target.name + "!")
    }
}
```
{% endtab %}
{% endtabs %}

That's it! We can see the description appear in /help:

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>The command now has a description</p></figcaption></figure>

#### Setting a permission

Setting a permission is equivalently simple. It can be set using the `@CommandPermission` annotation, which is functionally equal to Bukkit's `Permission` class.&#x20;

It takes two parameters: `value`, which is the permission node, and `defaultAccess`, which defines who can access the command by default. This is set to `PermissionDefault.OP`, and can be changed explicitly.

{% tabs %}
{% tab title="Java" %}
```diff
public class GreetCommands {

    @Command("greet")
    @Description("Greets the specified player")
+    @CommandPermission("test.plugin.greet")
    public void greet(BukkitCommandActor actor, Player target) {
        target.sendMessage("Welcome, " + target.getName() + "!");
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
<pre class="language-diff"><code class="lang-diff">class GreetCommands {
    
    @Command("greet")
    @Description("Greets the specified player")
<strong>+    @CommandPermission("test.plugin.greet")
</strong>    fun greet(actor: BukkitCommandActor, target: Player) {
        target.sendMessage("Welcome, ${target.name}!")
    }
}
</code></pre>
{% endtab %}
{% endtabs %}

That's it! Now, only people with the `test.plugin.greet` permission can use the command.

#### Setting default values

We would like to make our `target` argument optional. If it is not specified, it should default to the sender.

This can be done using the `@Default` annotation, which specifies a default value for the parameter.&#x20;

Because this is very common in plugins (default `Player`s to the sender), Lamp accepts special values for the `Player` type: `me`, `@s` and `self`. If these are supplied to a Player parameter, it will receive the sender as a value.&#x20;

{% tabs %}
{% tab title="Java" %}
```java
public class GreetCommands {

    @Command("greet")
    @Description("Greets the specified player")
    @CommandPermission("test.plugin.greet")
    public void greet(BukkitCommandActor actor, @Default("me") Player target) {
        target.sendMessage("Welcome, " + target.getName() + "!");
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class GreetCommands {

    @Command("greet")
    @Description("Greets the specified player")
    @CommandPermission("test.plugin.greet")
    fun greet(actor: BukkitCommandActor, @Default("me") target: Player) {
        target.sendMessage("Welcome, " + target.name + "!")
    }
}
```
{% endtab %}
{% endtabs %}

> :tada:  **Good news, Kotlin users!**&#x20;
>
> Lamp supports Kotlin's default arguments. You can use Kotlin's natural syntax for specifying the default parameters. The only caveat is that you need to explicitly mark these parameters with the `@Optional` annotation:
>
> ```kotlin
> class GreetCommands {
>
>     @Command("greet")
>     @Description("Greets the specified player")
>     @CommandPermission("test.plugin.greet")
>     fun greet(
>         actor: BukkitCommandActor,
>         @Optional target: Player = actor.requirePlayer()
>     ) {
>         target.sendMessage("Welcome, " + target.name + "!")
>     }
> }
> ```

Seeing it in action:

<figure><img src="../.gitbook/assets/image (6).png" alt="" width="299"><figcaption></figcaption></figure>
