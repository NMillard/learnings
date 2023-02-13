# Gradle Build tool

Gradle comes with a domain specific language (DSL), and is written in groovy or kotlin.

You'll need to create a build script named `build.gradle.kts` in the root of your java project.

## Plugins
By using plugins you can extend gradle's functionality (e.g. add more tasks).

```kotlin
plugins {
    java
}
```