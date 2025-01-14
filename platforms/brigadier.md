---
description: >-
  This page describes how to integrate Lamp with Brigadier, as well as what to
  expect out of it.
icon: sword
---

# Brigadier

Because Lamp and Brigadier share a lot in common in structure and organization, Lamp provides a `brigadier` module that allows for creating 1-to-1 mappings between the two frameworks easily.&#x20;

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

  <!-- Brigadier module -->
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
   
    // Brigadier module
    implementation 'io.github.revxrsal:lamp.brigadier:[VERSION]'
}
```
{% endtab %}

{% tab title="build.gradle.kts" %}
```kotlin
dependencies {
    // Required for all platforms
    implementation("io.github.revxrsal:lamp.common:[VERSION]")

    // Brigadier module
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

## Brigadier-specific annotations

### `@BType`

Specifies what `ArgumentType` a parameter should be mapped to

## Lamp to Brigadier conversion

All Lamp to Brigadier conversions are done using two main classes:

* **`BrigadierConverter`**: An interface that specifies how certain Brigadier or Lamp types should be converted.
  * `createActor`: Specifies how to convert Brigadier's sender S to Lamp's CommandActor A
  * `getArgumentType`: Specifies how to convert Lamp's ParameterType to Brigadier's ArgumentType. **It's recommended to use the registry Lamp provides, using the `ArgumentTypes` class**.
* **`BrigadierParser`**: Contains functions that accept Lamp objects and a **`BrigadierConverter`** to convert into respective Brigadier type&#x73;**.**
  * `createNode(ExecutableCommand)`: Creates a `LiteralCommandNode` from a Lamp `ExecutableCommand`
  * `createNode(CommandNode node)`: Creates a Brigadier `CommandNode` from a Lamp `CommandNode`.
  * `createRequirement`: Creates a `Predicate` for Brigadier from Lamp's CommandPermission
  * `createAction`: Creates a `Command<S>` from Lamp's `CommandAction`
  * `createSuggestionProvider`: Creates a Brigadier `SuggestionProvider` from Lamp's `SuggestionProvider`.
  * `toParameterType`: Converts a Brigadier `ArgumentType` to Lamp's `ParameterNode`
