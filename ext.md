# Che IDE plugins getting started guide
This walkthrough helps you to get started with the basics of the Che IDE plugins.
- [Generate a Che plugin](#generate-a-che-plugin)
  * [pom.xml](#pomxml)
  * [Entry point](#entry-point)
  * [Consuming the shared libraries](#consuming-the-shared-libraries)
- [Developing of a Che plugin](#developing-of-a-che-plugin)
  * [Running](#running)
  * [Debugging](#debugging)

## Generate a Che IDE plugin
Execute the following command to generate a new Che IDE plugin from a sample:
```
mvn archetype:generate \
   -DarchetypeGroupId=org.eclipse.che.archetype \
   -DarchetypeVersion=LATEST \
   -DarchetypeArtifactId=plugin-menu-archetype
```
During the code genration you will be asked to define several properies:
- groupId - type `org.eclipse.che.plugin`
- artifactId - type `che-plugin-menu`
- version - just hit the <kbd>Enter</kbd> to use `1.0-SNAPSHOT`
- package - type `org.eclipse.che.plugin.menu`

The command above generates an IDE plugin that adds a menu item (action) that pops up a notification.

Project with Che IDE plugin represents a Maven multi-module project with the following structure:
```
che-plugin-menu
├─ assembly
│  ├─ assembly-ide-war
│  └─ assembly-main
└─ plugins
   └─ che-plugin-menu
      └─ che-plugin-menu-ide // this project contains the code that’s entirely IDE-side
```

Let's look into the `che-plugin-menu-ide` module structure:
```
ide
├─ src                                              // sources folder
│  ├─ main
│  │  ├─ java
│  │  │  └─ org.eclipse.che.ide.ext.demo.client
│  │  │     └─ DemoExtension.java                   // entry point
│  │  ├─ resources
│  │  │  └─ org.eclipse.che.ide.ext.demo.client
│  │  └─ module.gwt.xml                             // template for generating GWT module
│  └─ test                                          // tests folder
├─ target                                           // build output
│  └─ classes
│     ├─ META-INF
│     │  └─ gwt
│     │     └─ mainModule                           // see https://tbroyer.github.io/gwt-maven-plugin/generate-module-metadata-mojo.html
│     └─ org.eclipse.che.ide.ext.demo
│        ├─ client
│        └─ Demo.gwt.xml                            // generated GWT module (for more details, see https://tbroyer.github.io/gwt-maven-plugin/generate-module-mojo.html)
└─ pom.xml
```

### pom.xml
There are several important parts in the ide/pom.xml:
```xml
<project>
    ...
    <!-- packaging triggers gwt-lib Maven life-cycle -->
    <packaging>gwt-lib</packaging>
    <dependencies>
        ...
        <!-- Che IDE API dependency -->
        <dependency>
            <groupId>org.eclipse.che.core</groupId>
            <artifactId>che-core-ide-api</artifactId>
        </dependency>
        ...
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>net.ltgt.gwt.maven</groupId>
                <artifactId>gwt-maven-plugin</artifactId>
                <extensions>true</extensions>
                <configuration>
                    <!-- GWT module name -->
                    <moduleName>org.eclipse.che.plugin.menu.SampleMenuExtension</moduleName>
                </configuration>
            </plugin>
            ...
        </plugins>
    </build>
</project>
```

Packaging of the Maven project must be `gwt-lib` which triggers a [`gwt-lib`](https://tbroyer.github.io/gwt-maven-plugin/lifecycles.html#GWT_Library:_gwt-lib) Maven lifecycle that will build a [GWT library](https://tbroyer.github.io/gwt-maven-plugin/artifact-handlers.html#GWT_Library:_gwt-lib).

Dependency on the library that provides a [set of APIs](https://docs.google.com/spreadsheets/d/1ijapDnl1G7svy7sIKgTntyTuVsnd9nFcH0-357C0MxE/edit#gid=0) for extending Che IDE:
```xml
<dependency>
    <groupId>org.eclipse.che.core</groupId>
    <artifactId>che-core-ide-api</artifactId>
</dependency>
```

Name of the GWT module to generate defined in the configuration of `gwt-maven-plugin`:
```xml
<plugin>
    <groupId>net.ltgt.gwt.maven</groupId>
    <artifactId>gwt-maven-plugin</artifactId>
    <extensions>true</extensions>
    <configuration>
        <moduleName>org.eclipse.che.plugin.menu.SampleMenuExtension</moduleName>
    </configuration>
</plugin>
```
For details on the generating GWT module, read the `gwt:generate-module` mojo [description](https://tbroyer.github.io/gwt-maven-plugin/generate-module-mojo.html).

### Entry point
Che IDE plugin has an enrty point - Java class annotated with an [`@org.eclipse.che.ide.api.extension.Extension`](https://github.com/eclipse/che/blob/master/ide/che-core-ide-api/src/main/java/org/eclipse/che/ide/api/extension/Extension.java) annotation.
```java
package org.eclipse.che.plugin.menu.ide;

import org.eclipse.che.ide.api.extension.Extension;

@Extension(title = "Sample Menu")
public class SampleMenuExtension {
  ...
}
```
Plugin entry point is called immediatelly after initilaizing the core part of the Che IDE.

## Developing of a Che plugin
### Running
To try your plugin with Che IDE:
1. Build Che plugin with `mvn clean install`.
2. Start Che with your local binaries:
```
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    -v <local-path>:/data \
                    -v <local-assembly>:/assembly \
                    eclipse/che-cli:che6 start
```
Now open launched Che IDE and see that plugin is included:
- plugin name is in the plugins list `Profile->Preferences`;
- menu `Sample` with action `Say Hello` is present in the menu bar.

### Test changes
Rebuilding Che IDE with your changes in the plugin's code isn't so fast. But you can use GWT Super DevMode to quick test your changes. To load Che IDE in Super DevMode you need to launch GWT Code Server. Go to the root folder of your plugin (`che-plugin-menu`) and execute the following command `mvn gwt:codeserver -pl :assembly-ide-war -am`.
Once CodeServer is run there's a message in a terminal:

`[INFO] The code server is ready at http://127.0.0.1:9876/`

Now you can recompile Che IDE with your chages applyed by `Assistant -> GWT Super DevMode: recompile` action.

### Debugging
You can debug your plugin using Chrome DevTools when Che IDE launched in a Super DevMode.
