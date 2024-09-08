---
description: >-
  This page describes how to integrate Lamp with Fabric, as well as what to
  expect out of it.
---

# Fabric

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

  <!-- Fabric module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.fabric</artifactId>
      <version>[VERSION]</version>
  </dependency>  
  
  <!-- Optional: Brigadier module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.brigadier</artifactId>
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
   
   // Velocity module
   implementation 'io.github.revxrsal:lamp.fabric:[VERSION]'
   
   // Optional: Brigadier module
   implementation 'io.github.revxrsal:lamp.brigadier:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")
    
    // Velocity module
    implementation("io.github.revxrsal:lamp.fabric:[VERSION]")
    
    // Optional: Brigadier module
    implementation("io.github.revxrsal:lamp.brigadier:[VERSION]")
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

## Fabric-specific annotations

### `@CommandPermission`

Adds a command permission level for the given command

## Supported Fabric types

* `net.minecraft.server.command.ServerCommandSource` and its subclasses in place of CommandActor
* `net.minecraft.server.network.ServerPlayerEntity`
* `net.minecraft.world.World`

## Example

{% tabs %}
{% tab title="Java" %}
```java
public class MyMod implements ModInitializer {

    @Override
    public void onInitialize() {
        var lamp = FabricLamp.builder(this).build();
        lamp.register(new MyCommand());
    }

    public class MyCommand {

        @Command("hello")
        @Description("Sends a hello message")
        @CommandPermission(4)
        public void hello(FabricCommandActor actor) {
            actor.reply("Hello from Fabric mod!");
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class MyMod : ModInitializer {

    override fun onInitialize() {
        val lamp = FabricLamp.builder(this).build()
        lamp.register(MyCommand())
    }

    inner class MyCommand {

        @Command("hello")
        @Description("Sends a hello message")
        @CommandPermission(4)
        fun hello(actor: FabricCommandActor) {
            actor.reply("Hello from Fabric mod!")
        }
    }
}
```
{% endtab %}
{% endtabs %}
