# Java 9 Modules

## Just a recap - what existed before
Before version 9 every java program and the the java runtime were organized according to the model:
* A java class was defined in a separated file. 
* Java classes could be defined in a package. (usualy materialized as a directory) 
* classes could be public (every class on any package could access this class) , protected (every class on any package could extend this class) , only accessable to other classes from the same package (even if in diferent jars). 
* The classes and packages are distributed in zip files called jar files (with the same extension)
* The java runtime find the classes that makes a java program by loading all classes specified in the **classPath**. This classpath can specify a set of standalone class files or a set of jar files.        

## How to list the existing java runtime modules

Starting with version 9 the Java runtime will be organized internaly as a set of modules. To see the existing java runtime modules from a JDK is only necessary to run the **--list-modules** parameter from the java program.

```
./java --list-modules
module java.activation@9-ea
module java.base@9-ea
module java.compiler@9-ea
module java.corba@9-ea
module java.datatransfer@9-ea
module java.desktop@9-ea
module java.instrument@9-ea
module java.jnlp@9-ea
module java.logging@9-ea
module java.management@9-ea
module java.management.rmi@9-ea
module java.naming@9-ea
...
```

It is possible to go in detail about what is declared in a specific module, by using the **--list-modules** parameter:

```
bin$ ./java --list-modules java.base
module java.base@9-ea
  exports java.io
  exports java.lang
  exports java.lang.annotation
  exports java.lang.invoke
  exports java.lang.module
  exports java.lang.ref
  exports java.lang.reflect
  exports java.math
  exports java.net
  exports java.net.spi
 ...
``` 

## Java module annatomy

A java module 
* As a *module-info.java* file on the module root directory where is specified:
* the module name: has the same rules as a class name
* the exports packages 
* the requires other modules (with their exported packages ) 

Every java modules depends from **java.base** even if is not stated on the module-info file.

If a module needs to use classes/packages defined in another module, then that module must be declared as **required** otherwise it will not have access to the other module public types. That means a public type of one module can't be accessable by another type of a second modules if it is not required. To be public is not enought. 
If a module exports a class that also indirectly exposes a class exported from another module then the module must be declared as **requires transitive**. That means that module A that requires Module B (with a require transitive of Module C) will automatically (provided by the java module system) require module C.
A module exports all **public types** present in packages. Only the packages exported explicitly on the module-info file are accessable by other public types from other modules.

### Readability graph

When one module requires another module, which also requires a third module and so on, what happens is in fact , if we follow the required relationships we can build a graph, called readability graph. 
Java 9, when compiling and starting the JVM will create this gragh and check for two constraints. Both must be valid for the compilation to be done or for the JVM to startup. The constratins are:
* the graph must **not have any cicle**:
  * For example, module A requires module B and module B requires module A.
* *no split packages*, that is no two or more exported packages from its required modules can have the same name. 
  * For exemple, module A requires module B and C and both B and C exports package X.  


### module-info.java

```
module lmen.mod1 {
    exports lmen.package.one; // export all public types definied in package lmen.package.one
    exports lmme.package.two; // export all public types defined in package lmen.package.two
    requires java.sql;  // lmen.mod1 reads java.sql
    requires transitive java.logger; // every module that reads lmen.mod1 will also read the module java.logger
}
``` 

### --module-path
To execute or compile a module is is necessaty to state where the modules are. This is done using the new parameter **--module-path**
So the java or javac programs from the JDK 9 read this parameter to findout which directories have the modules binaries.

`java --module-path dir1:dir2:dir3`

### To diagnost which module are loaded by a java program 

The java program has the **-Xdiag:resolver** that outputs to the console how the java virtual machine does the module resolver to execute a java program.  

```
$ ~/jdk9/jdk-9/bin/java -Xdiag:resolver --module-path mod1/target:mod2/target -m lmen.mod2/lmen.pt.two.Class2
[Resolver] Root module lmen.mod2 located
[Resolver]   (file:///home/l/java/modules-tut/mod2/target/)
[Resolver] Module lmen.mod1 located, required by lmen.mod2
[Resolver]   (file:///home/l/java/modules-tut/mod1/target/)
[Resolver] Module java.base located, required by lmen.mod2
....
```

### How to create a java module

#### Module source code

##### Module 1
* Create  java classes for the module as normal java classes
* define a module-info.java file in the root directory
```
src/
src/lmen
src/lmen/pt
src/lmen/pt/internal
src/lmen/pt/internal/Class1.java
src/lmen/pt/one
src/lmen/pt/one/Class1.java
src/module-info.java 
```

where 

```
mod1$ cat src/module-info.java 
module lmen.mod1 {
   exports lmen.pt.one; 
}

mod1$ cat src/lmen/pt/internal/Class1.java 
package lmen.pt.internal;

public class Class1 {

	public void doStuff() {
            System.out.println("this is internal class1");
	}	
}

mod1$ cat src/lmen/pt/one/Class1.java 
package lmen.pt.one;

public class Class1 {
	public void justDoIt() {
		System.out.println("This is package one class1 calling ");
		new lmen.pt.internal.Class1().doStuff();	
	}
}
```

##### Module 2 that uses the other module

Module lmen.mod2 will read Class1 from modul lmen.mod1

```
target/
target/lmen
target/lmen/pt
target/lmen/pt/two
target/lmen/pt/two/Class2.class
target/module-info.class

$ cat  mod2/src/module-info.java
module lmen.mod2 {
	requires lmen.mod1;
} 

$ cat mod2/src/lmen/pt/two/Class2.java 
package lmen.pt.two;

import lmen.pt.one.Class1;

public class Class2 {

   public static void main(String[] args) {
      System.out.println("This is module two main method, calling class from module 1");
      new Class1().justDoIt(); 
   }
}
```

#### Module source code compilation

##### Module 1
Using the java compile program by identifying the module-info and every class from the module.
```
bin/javac -d target src/module-info.java src/lmen/pt/one/Class1.java src/lmen/pt/internal/Class1.java 
```

The java compiled files are placed on the target directory as normal java class files. See that module-info.java was also compiled into module-info.class

```
target
target/lmen
target/lmen/pt
target/lmen/pt/internal
target/lmen/pt/internal/Class1.class
target/lmen/pt/one
target/lmen/pt/one/Class1.class
target/module-info.class
```

##### Module 2

Because module 2 reads module 1 then the java compiler needs to know where the module 1 compiled classes are. This is done using the argument  **--module-path** 

```
$ ~/jdk9/jdk-9/bin/javac --module-path ../mod1/target -d target src/lmen/pt/two/Class2.java src/module-info.java 
src/module-info.java:1: warning: [module] module name component mod2 should avoid terminal digits
module lmen.mod2 {
           ^
1 warning
```

#### Java execution with modules

To execute the module2 that has a main method, it is necessary to specify to the java program where it can find the modules binaries 
and the class with the main method. For this, the java program has two new arguments:
* **--module-path** or **-p** in short version to specify the module location;
* **--module [moduleName]/[MainClas]** or **-m [moduleName]/[MainClas]** in short version to tell where the main class 

```
$ ~/jdk9/jdk-9/bin/java --module-path mod1/target:mod2/target -m lmen.mod2/lmen.pt.two.Class2
This is module two main method, calling class from module 1
This is package one class1 calling 
this is internal class1
```

The java modules (like java.base, java.xml) are automatically added to the **--module-path** argument. If however a required module cannot be 
found on the module-path the java program will not be run. An error message will be displayed.

```
Error occurred during initialization of boot layer
java.lang.module.FindException: Module lmen.mod2 not found
```

#### Packaging the modules compiled classes as an archive

Java modules compile classes can be archived in a single file that is called *Modular java archive*. This is a regular jar file that 
also archives the module-info.class at the root level and might have module specific configuration on the MANIFEST.MF file. 

To build an *Modular java archive* it is used the jar JDK command with new arguments dedicated to modules.

##### Module 1
```
$ ~/jdk9/jdk-9/bin/jar --create --file lmen-mod1.jar  *
```

##### Module 2
```
$ ~/jdk9/jdk-9/bin/jar --create --file lmen-mod2.jar --module-version 1.0.0 *

$ ~/jdk9/jdk-9/bin/jar --list --file lmen-mod2.jar
META-INF/
META-INF/MANIFEST.MF
module-info.class
lmen/
lmen/pt/
lmen/pt/two/
lmen/pt/two/Class2.class

$ ~/jdk9/jdk-9/bin/jar --describe-module --file lmen-mod2.jar
module lmen.mod2@1.0.0 (module-info.class)
  requires mandated java.base
  requires lmen.mod1
  contains lmen.pt.two
```

#### Executing from modular java archives

To execute a java module archive is like the same as when the compiled classes are placed on the file system. 
The java program is used with the **--module-path** argument pointing to the directory where the jar file is placed. It can also point to the jar path.

```
$ ~/jdk9/jdk-9/bin/java --module-path libs --module lmen.mod2/lmen.pt.two.Class2
This is module two main method, calling class from module 1
This is package one class1 calling 
this is internal class1

$ ~/jdk9/jdk-9/bin/java --module-path libs/lmen-mod1.jar:libs/lmen-mod2.jar --module lmen.mod2/lmen.pt.two.Class2
Picked up JAVA_TOOL_OPTIONS: 
This is module two main method, calling class from module 1
This is package one class1 calling 
this is internal class1   
```

##### Classes not in a java module can access types defined by a java module.
There is no need to have all types defined in java modules for them to be accessed. For example a class in a regular jar (not module) can access the public types of any module.
To do this, the module path is combined with the class path and a the new argument **--add-modules** will tell what modules do the non modules classes required.
**--add-modules** is available on both the JDK java and javac programs. 
For instance to run the non module jar gui.jar that reads the lmen.mod1 java module only is necessary to do:

`
java --module-path mods --add-modules lmen.mod1 -jar gui.jar
`
## Java module versioning
A java module archive can have a version number assign to it (express on the MANIFEST.MF file ) but it seems it is not used while figuring out the modules to load at run time.  
## Services in the Java module system

### Until Java 9
Since java 6 the jdk has a standart facility to findout in the classpath the implementation of an interface. This is called the service loader. Very simply the service until java 8 works in this way:
* One jar, the service, defines an interface or abstract class which must be implementented by other jars, later this jar will find out all implementations:
* An implementator jar, the service provide, 
  * will implement the interface or extend the abstract classe 
  * create a file in META-INF/services where the name is the same as the service interface. The file content will be the full qualifier name of each class that implementents the service type.
* The class that wants to find out the service implementatuion will Call the `ServiceLoader.load(service.class) and then iterate the found service implementations. Note the load of service provider implementation is done in a static way. 

### With Java modules
If a module wants to define a service type and consume other modules that will implement the service type then is necessary to do:
* the service module will define in its module-info file:
  * add a line with **uses** _the service type full qualifier name_
* the service provider module will:
  * implement the service type 
  * add a line in the module-info (do not export the service type implementation):
    * **provides** _the service type full qualifier name_ **with** _the service implementation type full qualifier name_
* The service module will discover the service provider classes using the ServiceLoader.load(service.class) and then iterationg the found service implementations.

## How to implement Optional modules 

A module needs to specify on its module-info.java file all the modules it requires. the required modules must be available at compile time and at startup time. If they are not the module will not compile nor the application will start up. This is not very flexible.

What if a module needs the functionality of another module but this module is not available at compile time, only at run-time. Since there isn't the notion of a _optional module_ on the java module system, to implement such a use case is necessary to use Services and services providers.      

## Reflection in Java Module System
Befor the Java Modules System private members of a class could be accessed using the java Reflectin API.  

In a module this no longer applies. The types on non exporter packages aren't available for reflection. Only the public members of exported class packages are availabe for reflection. However the private members of an exported type could be available for reflection (using the reflection setAccessible(true)) if the module creator tells in the module-info file package is **exports private** .

## The unnamed modules

The jars defined in a class path are placed by the JVM 9 in a module called **unnamed**. The Java VM will create a module-info file automatically with the following settings:
* **requires** (or reads) every existing modules found on the module-path.
* **exports** export all packages found on the classpath. But because it is an unnamed module it cannot be required by other modules (to require is necessary a name). However an automatic module can read not only all the modules on the module-path but also the unnamed module (and in this way all packages in the class path).

# Implementation

**Class loaders**  are not changed. There are 3 classloads predefined:
* bootstrap loader
* plataform loader
* application loader
The class loaders are stil responsible for loading the classes. 
The module accessiblity restrictions are not enforced by the classloaders, but by the Java platform Module system and the JVM.
**Layer** a new concept, it associates classloaders to modules. A layer is a family of class loaders that together loaded the class that made the modules.
The JVM has a layer called **BootLayer** that groups together the 3 classloader mentioned before. At layer creation the layer tells which classloades should load a specif module.
External tools can create an hierachy of layers to organize modules.

## Migration 
How to use non module jar files like for instance jackson-core.jar in a java module application. 

Lets assume we have a java module that needs to access the types offered by a normal non modular jar, say jackson-core.jar.  
_In this case then the non-modular jar is added to the module-path._ The java virtual machine will automatically convert the non modular jar to a module jar achive (called the **automatic modules**). 
The JVM gives to the a **automatic module**  the following module caracteristics:
* the name module is derived from the jar file name
* **Exports all packages** available in the jar
* **Requires all modules** that mades the application (java plataform modules and user defined modules). This demand commes from the fact the jar might by reflection access any type defined in any module. Because statically is not possible know what the runtime refections calls will do, the automatically module requires every application module (plataform, user define or even another automatic module). 

### JDeps
The JDK 9 has a tool called Jdeps that analyzes a group of jar files and prints out a report with the packages present on each jar and also the dependencies between the packages across the jar files. 

For example hier is a summary of dependencies between 3 jackson jar files 
```
$ ~/jdk9/jdk-9/bin/jdeps -s jackson-*
jackson-annotations-2.8.8.jar -> java.base
jackson-core-2.8.8.jar -> java.base
jackson-databind-2.8.8.jar -> jackson-annotations-2.8.8.jar
jackson-databind-2.8.8.jar -> jackson-core-2.8.8.jar
jackson-databind-2.8.8.jar -> java.base
jackson-databind-2.8.8.jar -> java.desktop
jackson-databind-2.8.8.jar -> java.logging
jackson-databind-2.8.8.jar -> java.sql
jackson-databind-2.8.8.jar -> java.xml
``` 

The jdeps programm also has the capability to generate a module-info.java file for each jar analysed. This is useful as a starting point to begin the modulization of existing jars.

```
$ ~/jdk9/jdk-9/bin/jdeps --generate-module-info . jackson-*
writing to ./jackson.annotations/module-info.java
writing to ./jackson.databind/module-info.java
writing to ./jackson.core/module-info.java
```

 ## Stuff to see 
 ###Week modules 
  A module can be define as week meaning that all types are exported for reflections, that is, they can be accessed by the reflection API called by other modules. **Not sure if this is still valid**


## Jlink - java runtime distribution
The old way to distribute java was to use a JRE or JDK build and distributed by oracle. 

Now there is a new tool in java JDK 9 that links a set of modules and creates a specific java runtime with only those modules. With this tool is possible to define a java runtime as small as having only the java.base module por example, or create a java runtime without the java.desktop module, for example. 

To create a java runtime in myjava directory is necessary to do: 

```
jdk-9$ ./bin/jlink --module-path jmods --add-modules java.base --output myjava  
```

The **--add-modules** argument tells what are the starting point modules for the new distribuition. Beside this modules all modules that
 are required by the **--add-modules** (following the dependency graph) are also part of the java runtime distribuition. 

The new directory has a capable java runtime with only the java.base module. (Please note the java runtime image does not have any development tools like for example javac. It is more like a reduced JRE )

```
myjava$ ls
bin  conf  include  legal  lib  release

myjava/bin$ ./java --list-modules 
module java.base@9-ea
```