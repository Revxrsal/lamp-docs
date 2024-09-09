---
description: >-
  This page describes how to integrate Lamp with Bukkit (Spigot) and Paper, as
  well as what to expect out of it.
---

# Bukkit / Spigot / Paper

## Notes on Brigadier

Lamp allows for native integration with Brigadier, which is Minecraft's new command system introduced in 1.13 that comes with colorful suggestions, client-side validation, and space-aware auto-completion.

However, because Bukkit does not provide supported APIs for that, Lamp has to rely on hacks for injecting commands.

You may need to depend on the `brigadier` module if you would like to customize the Brigadier integration.

Lamp's Brigadier integration is enabled by default for 1.13+ but with some nuances. Here is a support table:

| Version         | Brigadier Supported                                               |
| --------------- | ----------------------------------------------------------------- |
| 1.13 - 1.19     | :white\_check\_mark:                                              |
| 1.19.1 - 1.20.5 | :heavy\_check\_mark: Paper only                                   |
| 1.20.6+         | :heavy\_check\_mark: Paper only + **requires the `paper` module** |

## Setting up

### Prerequisites

* Java 17 or newer
* `paper`: Java 21 or newer

### Adding Lamp dependency

To add Lamp to your project, add the following (depending on your project structure):

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

  <!-- Bukkit module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.bukkit</artifactId>
      <version>[VERSION]</version>
  </dependency>  
  
  <!-- Optional: Brigadier module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.brigadier</artifactId>
      <version>[VERSION]</version>
  </dependency>
  
  <!-- Optional: Paper module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.paper</artifactId>
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
   
   // Bukkit module
   implementation 'io.github.revxrsal:lamp.bukkit:[VERSION]'
   
   // Optional: Brigadier module
   implementation 'io.github.revxrsal:lamp.brigadier:[VERSION]'
   
   // Optional: Paper module (requires Java 21+)
   implementation 'io.github.revxrsal:lamp.paper:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")
    
    // Bukkit module
    implementation("io.github.revxrsal:lamp.bukkit:[VERSION]")
    
    // Optional: Brigadier module
    implementation("io.github.revxrsal:lamp.brigadier:[VERSION]")
    
    // Optional: Paper module (requires Java 21+)
    implementation("io.github.revxrsal:lamp.paper:[VERSION]")
}
```
{% endtab %}
{% endtabs %}

Latest version: ![](https://img.shields.io/maven-metadata/v/https/repo1.maven.org/maven2/io/github/revxrsal/lamp.common/maven-metadata.xml.svg?label=maven%20central\&colorB=brightgreen)

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

## Bukkit-specific annotations

### `@CommandPermission`

Adds a command permission for the given command

### `@FallbackPrefix`

Customizes the fallback prefix of a command

Bukkit uses this fallback prefix when two commands conflict with their main label, in which Bukkit would register it as `/<fallback prefix>:command` instead.

## Supported Bukkit types

* `org.bukkit.command.CommandSender` and its subclasses in place of CommandActor
* `org.bukkit.entity.Player`
* `org.bukkit.OfflinePlayer`
* `org.bukkit.World`
* `org.bukkit.entity.Entity` on 1.13+ (requires Brigadier)

## Example

{% tabs %}
{% tab title="Java" %}
```java
public final class MyPlugin extends JavaPlugin {

    @Override
    public void onEnable() {
        Lamp<BukkitCommandActor> lamp = BukkitLamp.builder(this).build();
        lamp.register(new MyCommand());
    }

    public class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        public void hello(BukkitCommandActor actor) {
            // Command logic here
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class MyPlugin : JavaPlugin() {

    override fun onEnable() {
      val lamp: Lamp<BukkitCommandActor> = BukkitLamp.builder(this)
          .build()
      lamp.register(MyCommand())
    }

    class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        fun hello(actor: BukkitCommandActor) {
            // Command logic here
        }
    }
}
```
{% endtab %}
{% endtabs %}

