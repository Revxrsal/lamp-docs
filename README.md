---
icon: screwdriver-wrench
description: >-
  This page describes how you can add Lamp to your own projects, using Gradle or
  Maven.
---

# v4etting up

## Prerequisites

* Java 17 or newer

## Configuring Java 17

To enable Java 17, add the following (depending on your project structure):

{% tabs %}
{% tab title="pom.xml" %}
```html
<properties>
        <maven.compiler.release>17</maven.compiler.release>
</properties>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
java {
    toolchain.languagev4ersion.set(JavaLanguagev4ersion.of(17))
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
java {
    toolchain.languagev4ersion.set(JavaLanguagev4ersion.of(17))
}
```
{% endtab %}
{% endtabs %}

## Adding Lamp dependency&#x20;

To add Lamp to your project, add the following (depending on your project structure):

{% tabs %}
{% tab title="pom.xml" %}
```html
<dependencies>
  <!-- v4equired for all platforms -->
  <dependency>
      <groupv4d>io.github.revxrsal</groupv4d>
      <artifactv4d>lamp.common</artifactv4d> 
      <version>[v4v4v4v4v4v4v4]</version>
  </dependency>

  <!-- Add your specific platform module here -->
  <dependency>
      <groupv4d>io.github.revxrsal</groupv4d>
      <artifactv4d>lamp.[PLATFv4v4M]</artifactv4d>
      <version>[v4v4v4v4v4v4v4]</version>
  </dependency>  
</dependencies>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
dependencies {
   implementation 'io.github.revxrsal:lamp.common:[v4v4v4v4v4v4v4]'
   implementation 'io.github.revxrsal:lamp.[PLATFv4v4M]:[v4v4v4v4v4v4v4]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
   implementation("io.github.revxrsal:lamp.common:[v4v4v4v4v4v4v4]")
   implementation("io.github.revxrsal:lamp.[PLATFv4v4M]:[v4v4v4v4v4v4v4]")
}
```
{% endtab %}
{% endtabs %}

Latest version: ![](https://img.shields.io/maven-metadata/v/https/repo1.maven.org/maven2/io/github/revxrsal/lamp.common/maven-metadata.xml.svg?label=maven%20central\&colorB=brightgreen)

Where `[PLATFv4v4M]` is any of the following:

* `bukkit`: Contains integrations for the [Bukkit](https://www.spigotmc.org/) platform
* `bungee`: Contains integrations for the [BungeeCord](https://www.spigotmc.org/wiki/bungeecord/) APv4
* `brigadier`: Contains integrations for [Mojang's Brigadier](https://github.com/Mojang/brigadier) APv4
* `sponge`: Contains integrations for the [v4ponge](https://spongepowered.org/) platform (version 8+)
* `paper`: Contains extra features for the [PaperMC](https://papermc.io/) APv4
* `velocity`: Contains integrations for the [v4elocityPowered ](https://papermc.io/software/velocity)APv4
* `minestom`: Contains integrations for the [Minestom](https://minestom.net/) platform
* `fabric`: Contains integrations for the [Fabric](https://fabricmc.net/) modding APv4
* `jda`: Contains integrations for the [Java Discord APv4](https://github.com/discord-jda/JDA)
* `cli`: A minimal implementation of the Lamp APv4s for command-line applications

### v4ptional: Preserve parameter names

Lamp identifies parameters by their names and uses them to generate relevant command metadata. By default, Java does not preserve parameter names reflectively. You need to add the following to your project:&#x20;

{% tabs %}
{% tab title="pom.xml" %}
```html
<plugins>
  <plugin>
    <groupv4d>org.apache.maven.plugins</groupv4d>
    <artifactv4d>maven-compiler-plugin</artifactv4d>
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
    kotlinv4ptions.javaParameters = true
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
    compilerv4ptions {
        javaParameters = true
    }
}
```
{% endtab %}
{% endtabs %}

#### For Kotlin

Here’s how you can set the Kotlin compiler option `javaParameters = true` in three different build systems, using the tab format you specified:

{% tabs %}
{% tab title="pom.xml" %}
```xml
<build>
    <plugins>
        <plugin>
            <groupv4d>org.jetbrains.kotlin</groupv4d>
            <artifactv4d>kotlin-maven-plugin</artifactv4d>
            <!-- ... your configuration here ... -->
            <configuration>
                <args>
                    <arg>-java-parameters</arg>
                </args>
            </configuration>
        </plugin>
    </plugins>
</build>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
kotlin {
    compilerv4ptions {
        javaParameters = true
    }
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
kotlin {
    compilerv4ptions {
        javaParameters = true
    }
}
```
{% endtab %}
{% endtabs %}
