# Getting started with the Che IDE plugins
This walkthrough helps you to get started with the basics of the Che IDE plugins.

## Generate a Che plugin
Execute the following command to generate a new Che plugin from a ~~sample~~ template:
```
mvn archetype:generate \
   -DarchetypeGroupId=org.eclipse.che.archetype \
   -DarchetypeVersion=LATEST \
   -DarchetypeArtifactId=plugin-wizard-archetype
```
The command above generates the Maven multi-module project with the following structure:
- `che-plugin-demo` - Maven reactor project for the Che plugin, which does not contain any sources and lists three modules to include:
  - `ide` - this project contains the code that’s entirely IDE-side;
  - `server` - this project contains the code that’s entirely server-side;
  - `shared` - this project contains the code that’s shared between the IDE and server, e.g. models, DTOs, constants.
Let's look into the `ide` module structure:
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
                    <moduleName>org.artem.plugin.ide.SampleWizardExtension</moduleName>
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

Name of the [GWT module](http://www.gwtproject.org/doc/latest/FAQ_Client.html#What_is_a_GWT_Module?) to generate defined in the configuration of `gwt-maven-plugin`:
```xml
<plugin>
    <groupId>net.ltgt.gwt.maven</groupId>
    <artifactId>gwt-maven-plugin</artifactId>
    <extensions>true</extensions>
    <configuration>
        <moduleName>org.eclipse.che.plugin.demo.Demo</moduleName>
    </configuration>
</plugin>
```
For details on the generating GWT module, read the `gwt:generate-module` mojo [description](https://tbroyer.github.io/gwt-maven-plugin/generate-module-mojo.html).

### Entry point
Che IDE plugin has an enrty point - Java class annotated with an [`@org.eclipse.che.ide.api.extension.Extension`](https://github.com/eclipse/che/blob/master/ide/che-core-ide-api/src/main/java/org/eclipse/che/ide/api/extension/Extension.java) annotation.
```java
package org.eclipse.che.plugin.demo.client;

import com.google.inject.Inject;
import org.eclipse.che.ide.api.extension.Extension;

@Extension(title = "Sample Wizard")
public class SampleWizardExtension {

  @Inject
  public SampleWizardExtension() {
    ...
  }
}
```

Plugin entry point is called immediatelly after initilaizing the core part of the Che IDE.

## Developing of a Che plugin

### SDM
`mvn gwt:codeserver -pl :che-ide-gwt-app -am -Dskip-enforce -Dskip-validate-sources`

## Include a plugin into Che IDE
To include your plugin into Che IDE:
- ensure that your Che plugin is already built;
- add a dependency into the [pom.xml](https://github.com/eclipse/che/blob/che6/ide/che-ide-full/pom.xml):
```xml
    <dependencies>
        ...
        <dependency>
            <groupId>org.eclipse.che.plugin</groupId>
            <artifactId>che-plugin-demo</artifactId>
        </dependency>
    </dependencies>
```
- rebuild Che IDE:
`mvn clean install -pl :che-ide-full,che-ide-gwt-app,assembly-ide-war`
- restart Che:
`che stop && che start`
