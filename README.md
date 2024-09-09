---
icon: screwdriver-wrench
description: >-
  This page describes how you can add Lamp to your own projects, using Gradle or
  Maven.
---

# Setting up

## Prerequisites

* Java 17 or newer

## Configuring Java 17

To enable Java 17, add the following (depending on your project structure):

{% tabs %}
{% tab title="pom.xml" %}
```markup
<properties>
        <maven.compiler.release>17</maven.compiler.release>
</properties>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
java {
    toolchain.languageVersion.set(JavaLanguageVersion.of(17))
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
java {
    toolchain.languageVersion.set(JavaLanguageVersion.of(17))
}
```
{% endtab %}
{% endtabs %}

## Adding Lamp dependency&#x20;

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

  <!-- Add your specific platform module here -->
  <dependency>
      <groupId>io.github.revxrsal</groupId>
      <artifactId>lamp.[PLATFORM]</artifactId>
      <version>[VERSION]</version>
  </dependency>  
</dependencies>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
dependencies {
   implementation 'io.github.revxrsal:lamp.common:[VERSION]'
   implementation 'io.github.revxrsal:lamp.[PLATFORM]:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
   implementation("io.github.revxrsal:lamp.common:[VERSION]")
   implementation("io.github.revxrsal:lamp.[PLATFORM]:[VERSION]")
}
```
{% endtab %}
{% endtabs %}

Latest version: ![](https://img.shields.io/maven-metadata/v/https/repo1.maven.org/maven2/io/github/revxrsal/lamp.common/maven-metadata.xml.svg?label=maven%20central\&colorB=brightgreen)

Where `[PLATFORM]` is any of the following:

* `bukkit`: Contains integrations for the [Bukkit](https://www.spigotmc.org/) platform
* `bungee`: Contains integrations for the [BungeeCord](https://www.spigotmc.org/wiki/bungeecord/) API
* `brigadier`: Contains integrations for [Mojang's Brigadier](https://github.com/Mojang/brigadier) API
* `sponge`: Contains integrations for the [Sponge](https://spongepowered.org/) platform (version 8+)
* `paper`: Contains extra features for the [PaperMC](https://papermc.io/) API
* `velocity`: Contains integrations for the [VelocityPowered ](https://papermc.io/software/velocity)API
* `minestom`: Contains integrations for the [Minestom](https://minestom.net/) platform
* `fabric`: Contains integrations for the [Fabric](https://fabricmc.net/) modding API
* `jda`: Contains integrations for the [Java Discord API](https://github.com/discord-jda/JDA)
* `cli`: A minimal implementation of the Lamp APIs for command-line applications

### Optional: Preserve parameter names

Lamp identifies parameters by their names and uses them to generate relevant command metadata. By default, Java does not preserve parameter names reflectively. You need to add the following to your project:&#x20;

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
