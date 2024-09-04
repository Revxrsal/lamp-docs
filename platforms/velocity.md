---
description: >-
  This page describes how to integrate Lamp with Velocity 3+, as well as what to
  expect out of it.
---

# Velocity

## Setting up

### Prerequisites

* Java 17 or newer

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

  <!-- Velocity module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.velocity</artifactId>
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
   implementation 'io.github.revxrsal:lamp.velocity:[VERSION]'
   
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
    implementation("io.github.revxrsal:lamp.velocity:[VERSION]")
    
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

## Velocity-specific annotations

### `@CommandPermission`

Adds a command permission for the given command

## Supported Velocity types

* `com.velocitypowered.api.command.CommandSource` and its subclasses in place of CommandActor
* `com.velocitypowered.api.proxy.Player`

## Example

{% tabs %}
{% tab title="Java" %}
```java
@Plugin(
    id = "myplugin",
    name = "MyPlugin",
    version = "1.0",
    description = "A simple Velocity plugin",
    authors = {"YourName"}
)
public class MyPlugin {

    private final ProxyServer server;
    private final Lamp<VelocityCommandActor> lamp;

    @Inject
    public MyPlugin(ProxyServer server, @DataDirectory Path dataDirectory) {
        this.server = server;
        this.lamp = VelocityLamp.builder(this, server).build();
        lamp.register(new MyCommand());
    }

    public static class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        public void hello(VelocityCommandActor actor) {
            ...
        }
    }
}

```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Plugin(
    id = "myplugin",
    name = "MyPlugin",
    version = "1.0",
    description = "A simple Velocity plugin",
    authors = ["YourName"]
)
class MyPlugin @Inject constructor(
    private val server: ProxyServer,
    @DataDirectory private val dataDirectory: Path
) {

    private val lamp = VelocityLamp.builder(this, server).build()

    init {
        lamp.register(MyCommand())
    }

    class MyCommand {

        @Command("hello")
        @CommandPermission("myplugin.hello")
        fun hello(actor: VelocityCommandActor) {
            ...
        }
    }
}

```
{% endtab %}
{% endtabs %}
