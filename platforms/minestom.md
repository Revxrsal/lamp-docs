---
description: >-
  This page describes how to integrate Lamp with Minestom, as well as what to
  expect out of it.
icon: cloud
---

# Minestom

## Setting up

### Prerequisites

* Java 21 or newer

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

  <!-- Sponge module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.minestom</artifactId>
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
   
   // Sponge module
   implementation 'io.github.revxrsal:lamp.minestom:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")
    
    // Velocity module
    implementation("io.github.revxrsal:lamp.minestom:[VERSION]")
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

## Minestom-specific annotations

### `@CommandPermission`

Adds a command permission for the given command

## Supported Minestom types

* `net.minestom.server.command.CommandSender` and its subclasses in place of CommandActor
* `Player`
* `Instance`
* `EntityFinder`
* `ItemStack`
* `Component`
* `BinaryTag`
* `CompoundBinaryTag`

## Example

{% tabs %}
{% tab title="Java" %}
```java
public class MinestomServerTest {

    public static void main(String[] args) {
        createServer();
        var lamp = MinestomLamp.builder()
                .build();
        lamp.register(new TestCommands());
    }

    private static void createServer() {
        MinecraftServer minecraftServer = MinecraftServer.init();
        /*...*/
        minecraftServer.start("0.0.0.0", 25565);
    }

    public static class TestCommands {

        @Command("nbt")
        public void sendNBT(MinestomCommandActor actor, BinaryTag nbt) {
            actor.reply("NBT: " + nbt);
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
fun main(args: Array<String>) {
    createServer()
    val lamp = MinestomLamp.builder().build()
    lamp.register(TestCommands())
}

private fun createServer() {
    val minecraftServer = MinecraftServer.init()
    /*...*/
    minecraftServer.start("0.0.0.0", 25565)
}

class TestCommands {

    @Command("nbt")
    fun sendNBT(actor: MinestomCommandActor, nbt: BinaryTag) {
        actor.reply("NBT: $nbt")
    }
}
```
{% endtab %}
{% endtabs %}
