---
description: This page explains how to create help commands in Lamp
icon: message-question
---

# Help commands

The Help API in Lamp provides powerful functionality for managing and displaying command help menus.&#x20;

Here’s a detailed explanation of how you can use this API effectively.

### Including Help Interfaces in Command Methods

1. **`Help.RelatedCommands`**
   * **Purpose**: Use this interface to include all child and sibling commands. This is ideal for providing a complete help menu that encompasses all relevant commands at the same level or below.
   * **Usage**: Add `Help.RelatedCommands` as a parameter in your command method. This will provide the combined list of both child and sibling commands.

{% tabs %}
{% tab title="Java" %}
```java
private static final int ENTRIES_PER_PAGE = 7;

@Command("myplugin help")
public void sendHelpMenu(
	   BukkitCommandActor actor,
	   @Range(min = 1) @Default("1") int page,
	   Help.RelatedCommands<BukkitCommandActor> commands
) {
   var list = commands.asPage(page, ENTRIES_PER_PAGE);
   for (var command : list) {
      actor.reply("- " + command.usage());
   }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
private const val ENTRIES_PER_PAGE = 7

@Command("myplugin help")
fun sendHelpMenu(
   actor: BukkitCommandActor,
   @Range(min = 1) @Optional page: Int = 1,
   commands: Help.RelatedCommands<BukkitCommandActor>
) {
   val list = commands.asPage(page, ENTRIES_PER_PAGE)
   for (command in list) {
	   actor.reply("- ${command.usage()}")
   }
}
```
{% endtab %}
{% endtabs %}

2. **`Help.ChildrenCommands`**
   * **Purpose**: Use this interface to include all child commands associated with a specific command. This is useful for displaying commands that fall under a parent command, creating a hierarchical help menu.
   * **Usage**: Add `Help.ChildrenCommands` as a parameter in your command method. This will automatically provide the list of child commands for the current command.

{% tabs %}
{% tab title="Java" %}
```java
private static final int ENTRIES_PER_PAGE = 7;

@Command("myplugin help")
public void sendHelpMenu(
	   BukkitCommandActor actor,
	   @Range(min = 1) @Default("1") int page,
	   Help.ChildrenCommands<BukkitCommandActor> commands
) {
   var list = commands.asPage(page, ENTRIES_PER_PAGE);
   for (var command : list) {
      actor.reply("- " + command.usage());
   }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
private const val ENTRIES_PER_PAGE = 7

@Command("myplugin help")
fun sendHelpMenu(
   actor: BukkitCommandActor,
   @Range(min = 1) @Optional page: Int = 1,
   commands: Help.ChildrenCommands<BukkitCommandActor>
) {
   val list = commands.asPage(page, ENTRIES_PER_PAGE)
   for (command in list) {
	   actor.reply("- ${command.usage()}")
   }
}
```
{% endtab %}
{% endtabs %}

3. **`Help.SiblingCommands`**
   * **Purpose**: Use this interface to include all sibling commands that are on the same level as the current command. This is useful for showing related commands that are at the same command hierarchy level.
   * **Usage**: Add `Help.SiblingCommands` as a parameter in your command method. This will automatically provide the list of sibling commands for the current command.

{% tabs %}
{% tab title="Java" %}
```java
private static final int ENTRIES_PER_PAGE = 7;

@Command("myplugin help")
public void sendHelpMenu(
	   BukkitCommandActor actor,
	   @Range(min = 1) @Default("1") int page,
	   Help.SiblingCommands<BukkitCommandActor> commands
) {
   var list = commands.asPage(page, ENTRIES_PER_PAGE);
   for (var command : list) {
      actor.reply("- " + command.usage());
   }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
private const val ENTRIES_PER_PAGE = 7

@Command("myplugin help")
fun sendHelpMenu(
   actor: BukkitCommandActor,
   @Range(min = 1) @Optional page: Int = 1,
   commands: Help.SiblingCommands<BukkitCommandActor>
) {
   val list = commands.asPage(page, ENTRIES_PER_PAGE)
   for (command in list) {
	   actor.reply("- ${command.usage()}")
   }
}
```
{% endtab %}
{% endtabs %}
