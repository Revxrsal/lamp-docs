---
description: >-
  This page explains how to create ParameterValidators that perform checks
  against parameters after they are supplied by the user.
icon: binary-circle-check
---

# Parameter validators

The `ParameterValidator` interface is a part of the commands framework that allows you to perform custom validation on command parameters. This can be particularly useful when you want to enforce specific rules or constraints on command inputs, such as validating ranges, formats, or other criteria.

### Implementing a Custom `ParameterValidator`

Let's create an example of a custom `ParameterValidator` that checks if a number is within a specified range. This validator will make use of a custom annotation called `@Range` to specify the minimum and maximum allowed values.

#### Step 1: Define the `@Range` Annotation

First, we'll create a custom annotation `@Range` to specify the minimum and maximum values for our parameter:

{% tabs %}
{% tab title="Java" %}
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
public @interface Range {
    double min();
    double max();
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.VALUE_PARAMETER)
annotation class Range(val min: Double, val max: Double)
```
{% endtab %}
{% endtabs %}

#### Step 2: Implement the `RangeValidator`

Next, we implement the `RangeValidator` using the `ParameterValidator` interface. This validator checks the parameter value against the `@Range` annotation's constraints:

{% tabs %}
{% tab title="Java" %}
```java
public enum RangeValidator implements ParameterValidator<CommandActor, Number> {
    INSTANCE;

    @Override
    public void validate(@NotNull CommandActor actor, Number value, @NotNull ParameterNode<CommandActor, Number> parameter, @NotNull Lamp<CommandActor> lamp) {
        Range range = parameter.getAnnotation(Range.class);
        if (range == null) return;
        double d = value.doubleValue();
        if (d < range.min())
            throw new CommandErrorException("Specified value (" + d + ") is less than minimum " + range.min());
        if (d > range.max())
            throw new CommandErrorException("Specified value (" + d + ") is greater than maximum " + range.max());
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
object RangeValidator : ParameterValidator<CommandActor, Number> {
    
    override fun validate(
        actor: CommandActor,
        value: Number,
        parameter: ParameterNode<CommandActor, Number>,
        lamp: Lamp<CommandActor>
    ) {
        val range = parameter.getAnnotation(Range::class.java) ?: return
        val d = value.toDouble()
        if (d < range.min) 
            throw CommandErrorException("Specified value ($d) is less than minimum ${range.min}")
        if (d > range.max)
             throw CommandErrorException("Specified value ($d) is greater than maximum ${range.max}")
    }
}
```
{% endtab %}
{% endtabs %}

#### Step 3: Register the Validator

To use the `RangeValidator`, you need to register it with your `Lamp` instance. This is done through the `Lamp.Builder`:

{% tabs %}
{% tab title="Java" %}
```java
var lamp = Lamp.builder()
    .parameterValidator(Number.class, RangeValidator.INSTANCE)
    .build();
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
val lamp = Lamp.builder<CommandActor>()
    .parameterValidator(Number::class.java, RangeValidator)
    .build()
```
{% endtab %}
{% endtabs %}

#### Step 4: Using the `@Range` Annotation in Commands

Now, you can use the `@Range` annotation in your command methods to specify the valid range for numeric parameters:

{% tabs %}
{% tab title="Java" %}
```java
@Command("age")
public void setAge(CommandActor actor, @Range(min = 13, max = 99) int age) {
    /* ... */
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
@Command("age")
fun setAge(actor: CommandActor, @Range(min = 13.0, max = 99.0) age: Int) {
    /* ... */
}
```
{% endtab %}
{% endtabs %}

In this example, the `setAge` command requires the `age` parameter to be between 18 and 99. If the user provides a value outside this range, the `RangeValidator` will throw an exception, and Lamp will handle it appropriately.
