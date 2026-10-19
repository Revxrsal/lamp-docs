---
icon: flag
description: >-
  Flags are named parameters that can be given in any order. Switches are
  boolean flags that are turned on by being present.
---

# Flags and switches

Besides regular parameters, which are given in a fixed order, Lamp supports two kinds of named parameters:

* **Switches**, with `@Switch`: booleans that are `true` when present, like `--silent`
* **Flags**, with `@Flag`: named parameters that take a value, like `--reason griefing`

## Switches

```java
@Command("ban")
public void ban(BukkitCommandActor actor, Player target, @Switch("silent") boolean silent) {
    // ...
}
```

* `/ban Notch` → `silent` is `false`
* `/ban Notch --silent` → `silent` is `true`
* `/ban Notch -s` → `silent` is `true`

A switch parameter must be a `boolean`.

## Flags

```java
@Command("ban")
public void ban(
        BukkitCommandActor actor,
        Player target,
        @Flag("reason") String reason,
        @Optional @Flag("days") Integer days,
        @Default("1") @Flag("level") int level
) {
    // ...
}
```

* Flags are required by default: `/ban Notch` fails, as `--reason` is missing
* Flags are compatible with `@Optional` and `@Default`: `/ban Notch --reason griefing` gives `days = null` and `level = 1`
* Values with spaces can be quoted: `/ban Notch --reason "being mean" --days 3`

## Names and shorthands

Every flag and switch has a long form, prefixed with `--`, and a short form, prefixed with `-`:

* The long form is the annotation's value. If left empty, the parameter's name is used. This requires compiling with `-parameters`, or using `@Named`.
* The short form is the first letter of the long form. For example, `@Switch("silent")` is `-s`. You can choose a different one with `shorthand`:

```java
@Flag(value = "reason", shorthand = 'm') String reason // --reason or -m
```

Short switches can be combined, Unix-style: `-sp` is the same as `-s -p`.

## Where they go

Flags and switches can come anywhere after the command name. `/ban --reason griefing Notch` is the same as `/ban Notch --reason griefing`.

Inputs that look like negative numbers, such as `-1` or `-.5`, are read as values rather than short flags.

{% hint style="warning" %}
Declare required flags **before** switches and optional flags. Lamp requires optional parameters to come last, and switches count as optional. Otherwise, registering the command fails with:

`Found a non-optional parameter (...) sitting between optional parameters`
{% endhint %}

## Auto-completion

Flags and switches are suggested automatically. Typing `--` suggests the long forms that haven't been used yet, and `-` suggests the short forms.

## Brigadier and Minestom

On platforms where commands are checked against a client-side command tree (Bukkit 1.13+ unless Brigadier is disabled, Fabric, Velocity and Minestom), players must type flags and switches **after** the regular arguments:

* With up to 4 flags and switches, they can come in any order
* With more than 4, they must come in the order they are declared in, though optional ones can be skipped
