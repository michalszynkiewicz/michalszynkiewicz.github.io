# Thorntail Runner

Since 2.2.1.Final, there is a new way of running Thorntail apps in the IDE.

Previously, the only way was running the `main` method of the `Swarm` class. It caused problems with classpath. 
Flat classpath that was used in this case didn't mimic the jboss modules classpath that is used in the `java -jar ...` scenario well enough.

Thorntail Runner uses jboss modules approach. It was tested with all the issues users reported in the old way of running apps in the IDE and it solved all of them.

## How to use Runner

### Make it visible for your browser
In order to use Thorntail Runner, you have add `io.thorntail:thorntail-runner` to your application. If you use Maven, you can simply make your project depend on it.
If you use maven, add the following to your dependencies. Don't worry, neither the runner itself nor its dependencies won't leak to the fat jar that thorntail maven plugin generates for your project.

```XML
<dependency>
  <groupId>io.thorntail</groupId>
  <artifactId>thorntail-runner</artifactId>
  <version>${version.thorntail}</version>
</dependency>

```

### Run it
If you use IntelliJ IDEA, use Ctrl/Cmd+N to find the `Runner` class. In Eclipse you can use Ctrl/Cmd+Shift+T.
When you found it, it's just a matter of running it's main method.

You can customize Runner's behavior by using specific system properties.
List of all of them can be found in the javadoc of the class (or [here](https://github.com/thorntail/thorntail/blob/master/thorntail-runner/src/main/java/org/wildfly/swarm/runner/Runner.java)

## Please give us feedback
If you find any issues with the Thorntail Runner, or any other part of the project, please let us know.

You can contact us on the [Google Group](https://groups.google.com/forum/#!forum/thorntail), on IRC, in the #thorntail channel on Freenode.
Bug reports can be filed in the [issues.jboss.org](https://issues.jboss.org) JIRA.
