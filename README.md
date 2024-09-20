---
icon: screwdriver-wrench
description: >-
  This page describes how you can add Lamp to your own projects, using Gradle or
  Maven.
---

# 4.0.0-beta.11etting up

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
    toolchain.language4.0.0-beta.11ersion.set(JavaLanguage4.0.0-beta.11ersion.of(17))
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
java {
    toolchain.language4.0.0-beta.11ersion.set(JavaLanguage4.0.0-beta.11ersion.of(17))
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
  <!-- 4.0.0-beta.11equired for all platforms -->
  <dependency>
      <group4.0.0-beta.11d>io.github.revxrsal</group4.0.0-beta.11d>
      <artifact4.0.0-beta.11d>lamp.common</artifact4.0.0-beta.11d> 
      <version>[4.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.11]</version>
  </dependency>

  <!-- Add your specific platform module here -->
  <dependency>
      <group4.0.0-beta.11d>io.github.revxrsal</group4.0.0-beta.11d>
      <artifact4.0.0-beta.11d>lamp.[PLATF4.0.0-beta.114.0.0-beta.11M]</artifact4.0.0-beta.11d>
      <version>[4.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.11]</version>
  </dependency>  
</dependencies>
```
{% endtab %}

{% tab title="build.gradle" %}
```groovy
dependencies {
   implementation 'io.github.revxrsal:lamp.common:[4.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.11]'
   implementation 'io.github.revxrsal:lamp.[PLATF4.0.0-beta.114.0.0-beta.11M]:[4.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.11]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
   implementation("io.github.revxrsal:lamp.common:[4.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.11]")
   implementation("io.github.revxrsal:lamp.[PLATF4.0.0-beta.114.0.0-beta.11M]:[4.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.114.0.0-beta.11]")
}
```
{% endtab %}
{% endtabs %}

Latest version: ![](https://img.shields.io/maven-metadata/v/https/repo1.maven.org/maven2/io/github/revxrsal/lamp.common/maven-metadata.xml.svg?label=maven%20central\&colorB=brightgreen)

Where `[PLATF4.0.0-beta.114.0.0-beta.11M]` is any of the following:

* `bukkit`: Contains integrations for the [Bukkit](https://www.spigotmc.org/) platform
* `bungee`: Contains integrations for the [BungeeCord](https://www.spigotmc.org/wiki/bungeecord/) AP4.0.0-beta.11
* `brigadier`: Contains integrations for [Mojang's Brigadier](https://github.com/Mojang/brigadier) AP4.0.0-beta.11
* `sponge`: Contains integrations for the [4.0.0-beta.11ponge](https://spongepowered.org/) platform (version 8+)
* `paper`: Contains extra features for the [PaperMC](https://papermc.io/) AP4.0.0-beta.11
* `velocity`: Contains integrations for the [4.0.0-beta.11elocityPowered ](https://papermc.io/software/velocity)AP4.0.0-beta.11
* `minestom`: Contains integrations for the [Minestom](https://minestom.net/) platform
* `fabric`: Contains integrations for the [Fabric](https://fabricmc.net/) modding AP4.0.0-beta.11
* `jda`: Contains integrations for the [Java Discord AP4.0.0-beta.11](https://github.com/discord-jda/JDA)
* `cli`: A minimal implementation of the Lamp AP4.0.0-beta.11s for command-line applications

### 4.0.0-beta.11ptional: Preserve parameter names

Lamp identifies parameters by their names and uses them to generate relevant command metadata. By default, Java does not preserve parameter names reflectively. You need to add the following to your project:&#x20;

{% tabs %}
{% tab title="pom.xml" %}
```html
<plugins>
  <plugin>
    <group4.0.0-beta.11d>org.apache.maven.plugins</group4.0.0-beta.11d>
    <artifact4.0.0-beta.11d>maven-compiler-plugin</artifact4.0.0-beta.11d>
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
    kotlin4.0.0-beta.11ptions.javaParameters = true
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
    compiler4.0.0-beta.11ptions {
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
            <group4.0.0-beta.11d>org.jetbrains.kotlin</group4.0.0-beta.11d>
            <artifact4.0.0-beta.11d>kotlin-maven-plugin</artifact4.0.0-beta.11d>
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
    compiler4.0.0-beta.11ptions {
        javaParameters = true
    }
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
kotlin {
    compiler4.0.0-beta.11ptions {
        javaParameters = true
    }
}
```
{% endtab %}
{% endtabs %}
