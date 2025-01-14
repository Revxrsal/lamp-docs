---
description: >-
  This page describes how to integrate Lamp with the command line, as well as
  what to expect out of it.
icon: rectangle-terminal
---

# Command line

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

  <!-- CLI module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.cli</artifactId>
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
   
   // CLI module
   implementation 'io.github.revxrsal:lamp.cli:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")
    
    // CLI module
    implementation("io.github.revxrsal:lamp.cli:[VERSION]")
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

## Supported CLI types

* `java.util.Scanner` in place of CommandActor
* `java.io.PrintStream` in place of CommandActor

## Example

### Execute and exit

{% tabs %}
{% tab title="Java" %}
```java
public final class CLIApp {

    public static void main(String[] args) {
        var lamp = CLILamp.builder()
                .build();
        lamp.register(new PingCommand());
        /* register all other commands here */

        ConsoleActor actor = ActorFactory.defaultFactory().createForStdIo(lamp);
        lamp.dispatch(actor, String.join(" ", args));
    }

    static class PingCommand {

        @Command("ping")
        public void ping(ConsoleActor actor) {
            actor.reply("Pong!");
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
fun main(args: Array<String>) {
    val lamp = CLILamp.builder().build()
    lamp.register(PingCommand())
    /* register all other commands here */

    val actor = ActorFactory.defaultFactory().createForStdIo(lamp)
    lamp.dispatch(actor, args.joinToString(" "))
}

class PingCommand {

    @Command("ping")
    fun ping(actor: ConsoleActor) {
        actor.reply("Pong!")
    }
}
```
{% endtab %}
{% endtabs %}

### Poll console indefinitely

{% tabs %}
{% tab title="Java" %}
```java
public final class CLIApp {

    public static void main(String[] args) {
        var lamp = CLILamp.builder()
                .build();
        lamp.register(new PingCommand());
        /* register all other commands here */

        // Call this to continuously read input from the console
        //
        // pollStdin() is CLIVisitors.pollStdin().
        // we use static imports here for brevity.
        lamp.accept(pollStdin());
    }

    static class PingCommand {

        @Command("ping")
        public void ping(ConsoleActor actor) {
            actor.reply("Pong!");
        }
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
fun main(args: Array<String>) {
    val lamp = CLILamp.builder().build()
    lamp.register(PingCommand())
    /* register all other commands here */

    // Call this to continuously read input from the console
    //
    // pollStdin() is CLIVisitors.pollStdin().
    // we use static imports here for brevity.
    lamp.accept(pollStdin())
}

class PingCommand {

    @Command("ping")
    fun ping(actor: ConsoleActor) {
        actor.reply("Pong!")
    }
}
```
{% endtab %}
{% endtabs %}
