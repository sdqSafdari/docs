## Apache Maven
- Has a three built-in build lifecycles
	- default
	- clean
	- site
- each life-cycle comprises sequence of phases.
- for example default build lifecycle phases:
	- validate
	- compile
	- test
	- package
	- verify
	- install
	- deploy
- each phase bundled with zero or more goals
- in maven goal is a smallest unit of work
- each goal belong to one plugin which illustrated as bellow
```
plugin:goal
e.g.
mvn help:effective-pom
mvn clean dependency:copy-dependencies
```
### POM
##### POM stands for "Project Object Model" .It is an XML representation of a `Maven project` held in a file named *pom.xml* .It is an XML file that contains information about the project and configuration details used by Maven to build the project. like OOP , pom files support inheritance . the parent of all pom files is `super-pom` which has a default configurations .
>*maven project* could be a single pom.xml file without any other file
> e.g parent-pom that has a packaging type *pom*


> Each packaging ( "jar","war","ear","pom" )  contains a list of goals to bind to a particular phase.
> For example A project that is purely metadata (packaging value is pom) only binds goals to the `install` and `deploy` phases 

- super-pom
	- The Super POM is Maven's default POM. All POMs extend the Super POM unless explicitly set, meaning the 	   configuration specified in the Super POM is inherited by the POMs you created for your projects.
- effective-pom
	- The pom used for build (POM that results from the application of interpolation, inheritance and active 	   profiles) a project .
	> project interpolation : insert properties into place holders
- parent-pom
	- A pom which other poms can inherit its ( dependencies , properties , goal bindings )
- aggregate-pom
	- cascade your maven commands to its modules

### Plugin
- add goals to phases
##### plugins can contain information that indicates which lifecycle phase to bind a goal to. Note that adding the plugin on its own is not enough information - you must also specify the goals you want to run as part of your build.   
#### surefire plugin VS failsafe plugin   
The Failsafe Plugin is designed to run integration tests while the Surefire Plugin is designed to run unit tests.
The key point is that Surefire and Failsafe can both execute JUnit integration tests.     
The important difference is how they interact with Maven's lifecycle and how failures are handled, not what kind of JUnit test they can execute.    
The most important distinction is failure handling. Failsafe intentionally doesn't fail the build during `integration-test` phase;     
this allows Maven to continue into `post-integration-test` phase, where you can shut down containers, application servers, databases, etc.      

Code coverage tools such as Jacoco and execution container plugins such as Tomcat, Cargo, and Docker bind goals to the pre-integration-test phase to prepare the integration test container environment. These plugins also bind goals to the post-integration-test phase to collect coverage statistics or decommission the integration test container.      
The Failsafe Plugin is used during the **integration-test** and **verify** phases of the build lifecycle to execute the integration tests of an application. The Failsafe Plugin will not fail the build during the integration-test phase, thus enabling the post-integration-test phase to execute.      
The Failsafe Plugin has only two goals:
- failsafe:integration-test runs the integration tests of an application.
- failsafe:verify verifies that the integration tests of an application passed.   

The Maven Failsafe Plugin will scan for test classes whose fully qualified names match the following patterns.   
- "**/IT*.java"
- "**/*IT.java"
- "**/*ITCase.java"

## Maven dependency management
- maven project classpath libs(dependencies) management
- in maven project we have  **`transitive dependency`** and **`direct dependency`** .
- both kind of dependencies will be in a maven project classpath libs .
- **transitive dependency** is a dependency of your maven project dependency.

Maven dependencies are also Maven projects, so they also have their own “pom.xml” file. This is how dependencies declare their own dependencies, which we call the “transitive dependencies”

- there should be **only one dependency version** in classpath lib, here are approachs to choose the right version taken by maven :
    - nearest definition (Dependency mediation)
    - dependency management
> dependency management takes precedence over dependency mediation for transitive dependencies.
	


<pre>  A
  ├── B
  │   └── C
  │       └── D 2.0
  └── E
      └── D 1.0</pre>
      
in the preceding example {C,D} were transitive dependencies and due to maven nearest definition D version 1.0 will be chosen for project classpath.

## Refrences
- [apache maven docs](https://maven.apache.org/guides/)
- [maven dependency mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
- [maven surefire docs](https://maven.apache.org/surefire/maven-failsafe-plugin/)
