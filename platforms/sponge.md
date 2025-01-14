---
description: >-
  This page describes how to integrate Lamp with Sponge 8+, as well as what to
  expect out of it.
icon: soap
---

# Sponge

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

  <!-- Sponge module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.sponge</artifactId>
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
   implementation 'io.github.revxrsal:lamp.sponge:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")
    
    // Velocity module
    implementation("io.github.revxrsal:lamp.sponge:[VERSION]")
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

## Sponge-specific annotations

### `@CommandPermission`

Adds a command permission for the given command

## Supported Sponge types

* `org.spongepowered.api.command.CommandCause` and its subclasses in place of CommandActor
* `net.kyori.adventure.audience.Audience` and its subclasses in place of CommandActor
* `org.spongepowered.api.service.permission.Subject` and its subclasses in place of CommandActor
* `org.spongepowered.api.world.server.ServerWorld`
* `org.spongepowered.api.entity.living.player.server.ServerPlayer`
* `org.spongepowered.api.command.selector.Selector`

## Example

{% tabs %}
{% tab title="Java" %}
```java
@Plugin(id = "myplugin", name = "MyPlugin", version = "1.0")
public class MyPlugin {

    @Listener
    private void onConstructPlugin(ConstructPluginEvent event) {
        Lamp<SpongeCommandActor> lamp = SpongeLamp.builder(this).build();
        lamp.register(new MyCommand());
    }

    public class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        public void hello(SpongeCommandActor actor) {
            // Command logic here
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Plugin(id = "myplugin", name = "MyPlugin", version = "1.0")
class MyPlugin {

    @Listener
    private fun onConstructPlugin(event: ConstructPluginEvent) {
        val lamp: Lamp<SpongeCommandActor> = SpongeLamp.builder(this).build()
        lamp.register(MyCommand())
    }

    class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        fun hello(actor: SpongeCommandActor) {
            // Command logic here
        }
    }
}
```
{% endtab %}
{% endtabs %}
