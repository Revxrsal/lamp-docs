---
description: >-
  This page describes how to integrate Lamp with Geyser extensions, as well as
  what to expect out of it.
icon: cube
---

# Geyser

## Setting up

### Adding Lamp dependency

To add Lamp to your extension, add the following (depending on your project structure):

{% tabs %}
{% tab title="pom.xml" %}
```markup
<dependencies>
  <!-- Required for all platforms -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.common</artifactId>
      <version>[VERSION]</version>
  </dependency>

  <!-- Geyser module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.geyser</artifactId>
      <version>[VERSION]</version>
  </dependency>
</dependencies>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
dependencies {
   // Required for all platforms
   implementation 'io.github.revxrsal:lamp.common:[VERSION]'

   // Geyser module
   implementation 'io.github.revxrsal:lamp.geyser:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")

    // Geyser module
    implementation("io.github.revxrsal:lamp.geyser:[VERSION]")
}
```
{% endtab %}
{% endtabs %}

Latest version: ![](https://img.shields.io/maven-metadata/v/https/repo1.maven.org/maven2/io/github/revxrsal/lamp.common/maven-metadata.xml.svg?label=maven%20central\&colorB=brightgreen)

The Geyser module requires Java 21.

#### Optional: Preserve parameter names

Lamp identifies parameters by their names and uses them to generate relevant command metadata. By default, Java does not preserve parameter names reflectively. You need to add the following to your project:

{% tabs %}
{% tab title="pom.xml" %}
```markup
<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <configuration>
      <compilerArgs>
        <!-- Preserves parameter names -->
        <arg>-parameters</arg>
      </compilerArgs>
    </configuration>
  </plugin>
</plugins>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
compileJava { 
    // Preserve parameter names in the bytecode
    options.compilerArgs += ["-parameters"]
}

// optional: if you're using Kotlin
compileKotlin {
    kotlinOptions.javaParameters = true
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
tasks.withType<JavaCompile> {
    // Preserve parameter names in the bytecode
    options.compilerArgs.add("-parameters")
}

// optional: if you're using Kotlin
tasks.withType<KotlinJvmCompile> {
    compilerOptions {
        javaParameters = true
    }
}
```
{% endtab %}
{% endtabs %}

## Registering commands

Geyser only accepts commands while it fires `GeyserDefineCommandsEvent`. Therefore, you must create Lamp and register **all** your commands before that, in the `GeyserPreInitializeEvent`:

```java
@Subscribe
public void onPreInitialize(GeyserPreInitializeEvent event) {
    Lamp<GeyserCommandActor> lamp = GeyserLamp.builder(this).build();
    lamp.register(new MyCommands());
}
```

Registering a command after Geyser has defined its commands throws an exception.

## Geyser-specific annotations

### `@CommandPermission`

Adds a command permission for the given command.

### `@BedrockOnly`

Marks the command as only executable by Bedrock players. This is implied when the first parameter of the command is a `GeyserConnection` or a `GeyserPlayerEntity`.

### `@PlayerOnly`

Marks the command as only executable by players, either Java or Bedrock.

## How commands appear in Geyser

Geyser knows commands by their first word (e.g. `/game` for `/game arena create`), and allows a single permission per command. Lamp maps your commands as follows:

* If all commands under the same name share one `@CommandPermission`, it is given to Geyser. Otherwise, Lamp checks each command's permission when it is executed.
* If all commands under the same name are `@BedrockOnly` (or `@PlayerOnly`), Geyser rejects the others before the command reaches Lamp.
* The description is the first `@Description` among them.

## Supported Geyser types

### Sender types

When they come first in a command, these are resolved from the command sender:

* `org.geysermc.geyser.api.command.CommandSource`, in place of `GeyserCommandActor`
* `org.geysermc.geyser.api.connection.GeyserConnection`, for Bedrock players only
* `org.geysermc.geyser.api.entity.type.player.GeyserPlayerEntity`, for Bedrock players only

### Parameter types

* `GeyserConnection` and `GeyserPlayerEntity`, by the name of an online Bedrock player. `me`, `self` and `@s` refer to the sender.

### Context parameters

* Your `Extension`, using its interface or its class. It is also available as a dependency through `@Dependency`.

## Messages

`actor.reply(...)` and `actor.error(...)` accept ampersand color codes, such as `&a`.

## Example

{% tabs %}
{% tab title="Java" %}
```java
public class MyExtension implements Extension {

    @Subscribe
    public void onPreInitialize(GeyserPreInitializeEvent event) {
        Lamp<GeyserCommandActor> lamp = GeyserLamp.builder(this).build();
        lamp.register(new MyCommands());
        // register all your commands here
        lamp.register(...);
    }

    public static class MyCommands {

        @Command("hello")
        @CommandPermission("myextension.hello")
        public void hello(GeyserCommandActor actor) {
            actor.reply("&aHello!");
        }

        @Command("whoami")
        public void whoami(GeyserConnection sender) {
            // Bedrock players only, as the sender is a GeyserConnection
            ...
        }

        @Command("wave")
        public void wave(GeyserCommandActor actor, GeyserConnection target) {
            actor.reply("You waved at " + target.name());
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class MyExtension : Extension {

    @Subscribe
    fun onPreInitialize(event: GeyserPreInitializeEvent) {
        val lamp = GeyserLamp.builder(this).build()
        lamp.register(MyCommands())
        // register all your commands here
        lamp.register(...)
    }

    class MyCommands {

        @Command("hello")
        @CommandPermission("myextension.hello")
        fun hello(actor: GeyserCommandActor) {
            actor.reply("&aHello!")
        }

        @Command("whoami")
        fun whoami(sender: GeyserConnection) {
            // Bedrock players only, as the sender is a GeyserConnection
            ...
        }

        @Command("wave")
        fun wave(actor: GeyserCommandActor, target: GeyserConnection) {
            actor.reply("You waved at ${target.name()}")
        }
    }
}
```
{% endtab %}
{% endtabs %}
