---
description: >-
  This page explains how to use the AnnotationReplacer interface to dynamically
  replace custom annotations.
icon: highlighter-line
---

# Annotation replacers

The `AnnotationReplacer` interface allows you to create dynamic annotations that can be replaced by other annotations at runtime. This feature is powerful for creating flexible and configurable annotations that are not restricted by static, compile-time constraints.

### Overview

#### Interface Definition

```java
@FunctionalInterface
public interface AnnotationReplacer<T extends Annotation> {

    /**
     * Returns a collection of annotations that will substitute the given annotation,
     * and be accessible in {@link AnnotationList#get(Class)}.
     *
     * @param element    The element (method, parameter, class, etc.)
     * @param annotation The annotation to replace.
     * @return The list of replacing annotations. The collection may be null or empty.
     */
    @Nullable Collection<Annotation> replaceAnnotation(@NotNull AnnotatedElement element, @NotNull T annotation);

}
```

#### How It Works

1. **Define Your Custom Annotation**: Create an annotation that will be dynamically replaced.
2. **Implement the `AnnotationReplacer` Interface**: Define how your custom annotation will be replaced with others.
3. **Register the Replacer**: Use the builder to register your `AnnotationReplacer` with the framework.
4. **Apply the Custom Annotation**: Use the custom annotation in your code, and it will be replaced dynamically by the specified annotations.

### Example: Creating and Using `@PluginCommand`

In this example, we will create a custom `@PluginCommand` annotation that contains `path`, `description`, and `usage` attributes. We will then use an `AnnotationReplacer` to replace `@PluginCommand` with `@Command`, `@Description`, and `@Usage` annotations respectively.

#### Step 1: Define the `@PluginCommand` Annotation

{% tabs %}
{% tab title="Java" %}
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface PluginCommand {
    String path();
    String description();
    String usage();
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FUNCTION)
annotation class PluginCommand(
    val path: String,
    val description: String,
    val usage: String
)
```
{% endtab %}
{% endtabs %}

#### Step 2: Implement the `AnnotationReplacer` for `@PluginCommand`

{% tabs %}
{% tab title="Java" %}
```java
public class PluginCommandReplacer implements AnnotationReplacer<PluginCommand> {

    @Override
    public Collection<Annotation> replaceAnnotation(AnnotatedElement element, PluginCommand annotation) {
        // Create the replacing annotations dynamically
        Annotation commandAnnotation = Annotations.create(Command.class, "value", annotation.path());
        Annotation descriptionAnnotation = Annotations.create(Description.class, "value", annotation.description());
        Annotation usageAnnotation = Annotations.create(Usage.class, "value", annotation.usage());

        return Arrays.asList(commandAnnotation, descriptionAnnotation, usageAnnotation);
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class PluginCommandReplacer : AnnotationReplacer<PluginCommand> {

    override fun replaceAnnotation(element: AnnotatedElement, annotation: PluginCommand): Collection<Annotation> {
        // Create the replacing annotations dynamically
        val commandAnnotation = Annotations.create(Command::class.java, "value", annotation.path)
        val descriptionAnnotation = Annotations.create(Description::class.java, "value", annotation.description)
        val usageAnnotation = Annotations.create(Usage::class.java, "value", annotation.usage)

        return listOf(commandAnnotation, descriptionAnnotation, usageAnnotation)
    }
}
```
{% endtab %}
{% endtabs %}

#### Step 3: Register the Replacer

{% tabs %}
{% tab title="Java" %}
```java
BukkitLamp.builder(this)
    .annotationReplacer(PluginCommand.class, new PluginCommandReplacer())
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = BukkitLamp.builder(this)
    .annotationReplacer(PluginCommand::class.java, PluginCommandReplacer())
    .build()
```
{% endtab %}
{% endtabs %}

#### Step 4: Using the `@PluginCommand` Annotation

{% tabs %}
{% tab title="Java" %}
```java
@PluginCommand(
    path = "example", 
    description = "An example command", 
    usage = "/example <arg>"
)
public void exampleCommand() {
    // Command implementation
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@PluginCommand(
    path = "example", 
    description = "An example command", 
    usage = "/example <arg>"
)
fun exampleCommand() {
    // Command implementation
}
```
{% endtab %}
{% endtabs %}

In this example:

* `@PluginCommand` is defined with `path`, `description`, and `usage` attributes.
* The `PluginCommandReplacer` dynamically replaces `@PluginCommand` with `@Command`, `@Description`, and `@Usage` annotations.
* The command method `exampleCommand` is annotated with `@PluginCommand`, which will be replaced at runtime.

### Summary

The `AnnotationReplacer` interface provides a flexible way to dynamically replace annotations, allowing for powerful and configurable annotation setups. By creating and registering custom annotation replacers, you can enhance your command framework's capabilities and adapt annotations to your needs.
