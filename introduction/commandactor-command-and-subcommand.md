---
description: >-
  This page will introduce you to the fundamentals of creating commands with
  Lamp
icon: user
---

# CommandActor, @Command and @Subcommand

Building commands in Lamp is done **entirely** using annotations. Each command is represented as a **function** (method).



In this example, we will build a simple command, and then upgrade it gradually as we introduce more concepts.



## The CommandActor type

**CommandActor** is an interface that represents the sender of the command. It is platform-agnostic and provides generic functionality that is expected of any command sender.

**Every platform has its own subclass(es) of CommandActor:**

* Bukkit -> BukkitCommandActor
* Bungee -> BungeeCommandActor
* JDA -> SlashCommandActor
* Sponge -> SpongeCommandActor
* Velocity -> VelocityCommandActor
* ...

In these examples, we will use the common `CommandActor` interface. You can use the platform subclass to access more platform-specific functionality.

> :bulb: **For the curious**: Lamp does not enforce any specific CommandActor implementation, but provides a basic one for each platform. In further examples, we will build our own implementation of CommandActor and buff it with our own methods and functionality.

## The @Command and @Subcommand annotations

* **@Command** and **@Subcommand** are the two main annotations that we use to define commands.
  * @Command always defines the start of a new command. It is the main entry point to commands
  * @Subcommand defines a sub-command of a @Command. There cannot be a @Subcommand with no parent @Command.
* **Spaces in @Command and @Subcommand**
  * Spaces are respected in @Command and @Subcommand annotations. This means you can define complex commands easily and they would work as you expect
  * `@Command("foo bar")` -> /foo bar
  * `@Command("foo bar") @Subcommand("buzz bam")` -> /foo bar buzz bam
* **Combining @Command and @Subcommand**
  * `@Command` and `@Subcommand` can be defined on classes and inner classes to create a hierarchal tree of commands:
  * When any of @Command or @Subcommand defines multiple values, all possible combinations are taken
    * `@Command({"foo", "bar"}) @Subcommand({"joo", "kay"})` will generate:
      * /foo joo
      * /foo kay
      * /bar joo
      * /bar kay

{% code lineNumbers="true" %}
```java
@Command("game")
public class GameCommands {
    
    @Subcommand("test") // <--- /game test
    public void test(...) {}
    
    @Subcommand({"match", "arena"}) // <--- /game arena, /game match
    public static class Arena {
        
        @Subcommand("create") // <--- /game arena create, /game match create
        public void create(...) {}
        
    }
}
```
{% endcode %}

* **Argument names**
  * `@Command` and `@Subcommand` can define arguments that come in the middle of the command

```java
    @Command("bank <player> add")
    public void addCoins(Player player, int coins) {
    }

    @Command("bank <player> reset")
    public void resetCoins(Player player) {
    }
```

* Any trailing arguments that are not defined inside the annotation will automatically be added to the end of the command.
