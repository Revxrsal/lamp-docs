---
description: >-
  This page describes how to integrate Lamp with BungeeCord, as well as what to
  expect out of it.
icon: chart-network
---

# BungeeCord

## Setting up

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

  <!-- BungeeCord module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.bungee</artifactId>
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
   
   // Bungee module
   implementation 'io.github.revxrsal:lamp.bungee:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")
    
    // Bungee module
    implementation("io.github.revxrsal:lamp.bungee:[VERSION]")
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

## Bungee-specific annotations

### `@CommandPermission`

Adds a command permission for the given command

## Supported Bungee types

* `net.md_5.bungee.api.CommandSender` and its subclasses in place of CommandActor
* `net.md_5.bungee.api.connection.ProxiedPlayer`

## Example

{% tabs %}
{% tab title="Java" %}
```java
public class MyPlugin extends Plugin {

    @Override
    public void onEnable() {
        Lamp<BungeeCommandActor> lamp = BungeeLamp.builder(this).build();
        lamp.register(new MyCommand());
    }

    public class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        public void hello(BungeeCommandActor actor) {
            // Command logic here
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
import net.md_5.bungee.api.plugin.Plugin
import revxrsal.commands.BungeeLamp
import revxrsal.commands.Lamp
import revxrsal.commands.command.Command
import revxrsal.commands.command.CommandActor
import revxrsal.commands.command.CommandPermission

class MyPlugin : Plugin() {

    override fun onEnable() {
        val lamp: Lamp<BungeeCommandActor> = BungeeLamp.builder(this).build()
        lamp.register(MyCommand())
    }

    inner class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        fun hello(actor: BungeeCommandActor) {
            // Command logic here
        }
    }
}

```
{% endtab %}
{% endtabs %}
