---
description: >-
  This page describes how to integrate Lamp with JDA's slash commands, as well
  as what to expect out of it.
---

# JDA

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

  <!-- JDA module -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.jda</artifactId>
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
   
   // JDA module
   implementation 'io.github.revxrsal:lamp.jda:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
   // Required for all platforms
   implementation("io.github.revxrsal:lamp.common:[VERSION]")
    
   // JDA module
   implementation("io.github.revxrsal:lamp.jda:[VERSION]")
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

## JDA-specific annotations

### `@Choices`

Defines up to 25 predefined choices for the parameter. The user can only provide one of the choices and cannot specify any other value.

### `@CommandPermission`

Annotation for specifying the required [Permission](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/Permission.html)s for JDA slash commands

### `@GuildOnly`

Marks a command as executable in guilds only.

### `@NSFW`

Marks a command as NSFW

## Supported JDA types

* `SlashCommandInteractionEvent` and its subclasses in place of CommandActor
* `User` and its subclasses in place of CommandActor
* `Member` and its subclasses in place of CommandActor
* `MessageChannel` and its subclasses in place of CommandActor
* `Role`
* `User`
* `Member`
* `TextChannel`
* `VoiceChannel`
* `StageChannel`
* `NewsChannel`
* `ThreadChannel`
* `ScheduledEvent`
* `Category`
* `Emoji`

## Example

{% tabs %}
{% tab title="Java" %}
```java
public static void main(String[] args) {
    JDA jda = JDABuilder.createDefault(args[0]).build();
    var lamp = JDALamp.builder().build();

    // register all our commands here
    lamp.register(new ModerationCommands());
    lamp.register(new LevelCommands());
    lamp.register(new RoleCommands());
    lamp.register(new SupportCommands());

    // after we register all commands, we *must* call this to
    // send our commands as slash commands
    lamp.accept(slashCommands(jda));
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
fun main(args: Array<String>) {
    val jda = JDABuilder.createDefault(args[0]).build()
    val lamp = JDALamp.builder().build()

    // Register all our commands here
    lamp.register(ModerationCommands())
    lamp.register(LevelCommands())
    lamp.register(RoleCommands())
    lamp.register(SupportCommands())

    // After we register all commands, we *must* call this to
    // send our commands as slash commands
    lamp.accept(slashCommands(jda))
}
```
{% endtab %}
{% endtabs %}
