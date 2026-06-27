---
title: "Running Thorntail apps in the IDE with Thorntail Runner"
description: "A cleaner way to run Thorntail applications from your IDE — using the JBoss Modules classpath instead of a flat one, so in-IDE runs match java -jar."
date: 2019-01-15
# This is a genuine older post from my Red Hat / Thorntail days, kept as an
# example. Delete it (or set `draft: true`) if you want a single-topic launch.
---

Since 2.2.1.Final, there's a new way of running Thorntail apps in the IDE.

Previously, the only way was running the `main` method of the `Swarm` class. It caused problems with the classpath: the flat classpath used in that case didn't mimic the JBoss Modules classpath used in the `java -jar ...` scenario well enough.

Thorntail Runner uses the JBoss Modules approach. It was tested against all the issues users reported with the old way of running apps in the IDE, and it solved all of them.

## How to use Runner

### Make it visible to your IDE

To use Thorntail Runner, you add `io.thorntail:thorntail-runner` to your application. If you use Maven, add the following to your dependencies. Neither the runner itself nor its dependencies leak into the fat jar that the Thorntail Maven plugin generates for your project.

```xml
<dependency>
  <groupId>io.thorntail</groupId>
  <artifactId>thorntail-runner</artifactId>
  <version>${version.thorntail}</version>
</dependency>
```

### Run it

In IntelliJ IDEA, use <kbd>Ctrl/Cmd</kbd>+<kbd>N</kbd> to find the `Runner` class. In Eclipse you can use <kbd>Ctrl/Cmd</kbd>+<kbd>Shift</kbd>+<kbd>T</kbd>. Once you've found it, just run its `main` method.

You can customise Runner's behaviour with system properties. The full list lives in the javadoc of the class (or in the [source](https://github.com/thorntail/thorntail/blob/master/thorntail-runner/src/main/java/org/wildfly/swarm/runner/Runner.java)).

## Feedback

If you hit any issues with Thorntail Runner — or any other part of the project — let us know. Bug reports can be filed in the project's issue tracker.
