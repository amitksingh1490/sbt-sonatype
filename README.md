sbt-sonatype plugin
======

A sbt plugin for automating release processes at Sonatype Nexus. This plugin enables two-step release of your Scala/Java projects:

 * First, `publishSigned` (with [sbt-pgp plugin](http://www.scala-sbt.org/sbt-pgp/))
 * Next, `sonatypeRelease` to perform the close and release steps in Sonatype Nexus repository. 
 * That's all. Your project will be synchoronized to Maven central in a few hours. No need to enter the web interface of [Sonatype Nexus repository](http://oss.sonatype.org/).


Deploying to Sonatype repository is required for synchronizing your projects to the [Maven central repository](http://repo1.maven.org/maven2/).

## Prerequisites
 
 * Create a Sonatype Repository account 
   * Follow the instruction in [Sonatype OSS Maven Repository Usage Guide](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide). 
     * Create a Sonatype account
     * Create a GPG key
     * Open a JIRA ticket to get a permission for synchronizing your project to Maven central.


 * Related articles:
    * [Deploying to Sonatype - sbt Documentation](http://www.scala-sbt.org/release/docs/Community/Using-Sonatype.html)
    * [Publishing SBT projects to Nexus](http://www.cakesolutions.net/teamblogs/2012/01/28/publishing-sbt-projects-to-nexus/)

## Usage

sbt-sonatype is built for sbt-0.13.x.

### project/plugins.sbt

Import ***sbt-sonatype*** plugin to your project.
```scala
addSbtPlugin("org.xerial.sbt" % "sbt-sonatype" % "0.2.0")
```

 * If downloading the plugin fails, check the repository in the Maven central: <http://repo1.maven.org/maven2/org/xerial/sbt/sbt-sonatype_2.10_0.13>.
 It will be synchronized every ~4 hours.


### $HOME/.sbt/(sbt-version)/sonatype.sbt

Set Sonatype account information (user name and password) in the global sbt settings. Never include this setting file to your project. 

```scala
credentials += Credentials("Sonatype Nexus Repository Manager",
	    "oss.sonatype.org",
	    "(Sonatype user name)",
	    "(Sonatype password)"
```

### $HOME/.sbt/(sbt-version)/plugins/gpg.sbt

Add [sbt-pgp plugin](http://www.scala-sbt.org/sbt-pgp/) in order to use `publish-signed` command.

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-pgp" % "0.8.1")
```

### build.sbt

Import `SonatypeKeys._` and add `xerial.sbt.Sonatype.sonatypeSettings` to your sbt settings. The important settings are:

  * `profileName` 
     * This is your Sonatype acount profile name, e.g. `org.xerial`. If you do not set this value, it will be the same with the `organization` value.
  * `pomExtra`
     * A fragment of Maven's pom.xml. At least you need to define url, licenses, scm and deverlopers tags in this XML to satisfy [Maven central sync requirements](https://docs.sonatype.org/display/Repository/Central+Sync+Requirements).
  

```scala
import SonatypeKeys._

// Import default settings. This changes publishTo settings to use the Sonatype repository and add several commands for publishing.
sonatypeSettings

// Your project orgnization (package name)
organization := "org.xerial.example" 

// Your profile name of the sonatype account. The default is the same with the organization 
profileName := "org.xerial" 

// Project version. Only release version (w/o SNAPSHOT suffix) can be promoted.
version := "0.1" 

// To sync with Maven central, you need to supply the following information:
pomExtra := {
  <url>(your project URL)</url>
  <!-- License of your choice. -->
  <licenses>
    <license>
      <name>Apache 2</name>
      <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
    </license>
  </licenses>
  <!-- SCM information. Modify the follwing settings: -->
  <scm>
    <connection>scm:git:github.com/xerial/sbt-sonatype.git</connection>
    <developerConnection>scm:git:git@github.com:xerial/sbt-sonatype.git</developerConnection>
    <url>github.com/xerial/sbt-sonatype.git</url>
  </scm>
  <!-- Developer contact information -->
  <developers>
    <developer>
      <id>(your favorite id)</id>
      <name>(your name)</name>
      <url>(your web page)</url>
    </developer>
  </developers>
}
```

## Publishing Your Artifact

The general steps for publishing your artifact to Maven Central are as follows: 

 * `publish-signed` to deploy your artifact to staging repository at Sonatype.
 * `close` your staging repository at Sonatype. This step verifiles Maven central sync requiement, GPG-signature, javadoc and source code presence, pom.xml settings, etc.
 * `promote` verifies the closed repository so that it can be synched with Maven central. 
   * `release-sonatype` will do both `close` and `promote` in one step.

Note: If your project version has "SNAPSHOT" suffix, your project will be published to the [snapshot repository](http://oss.sonatype.org/content/repositories/snapshots) of Sonatype, and you cannot use `release-sonatype` command. 

### Command Line Usage

Publish a GPG-signed artifact to Sonatype:
```
$ sbt publishSigned
```

Do close and promote at once:
```
$ sbt sonatypeRelease
```
This command accesses [Sonatype Nexus REST API](https://oss.sonatype.org/nexus-staging-plugin/default/docs/index.html), then sends close and promote commands. 


## Available Commands

* __sonatypeList__
  * Show the list of staging repositories.
* __sonatypeClose__ (repositoryId)?
  * Close a staging repository.
* __sonatypePromote__ (repositoryId)?
  * Promote a staging repository.
* __sonatypeDrop__ (repositoryId)?
  * Drop a staging repository.
* __sonatypeRelease__ (repositoryId)?
  * Close and promote a staging repository.
* __sonatypeReleaseAll__
  * Close and promote all staging repositories (Useful for cross-building projects)
* __sonatypeStagingProfiles__
  * Show the list of staging profiles, which include profileName information.
* __sonatypeLog__
  * Show the staging activity logs

