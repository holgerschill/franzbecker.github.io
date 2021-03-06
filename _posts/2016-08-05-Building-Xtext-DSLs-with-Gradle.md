---
layout: post
title: "Building Xtext DSLs with Gradle"
tags: ["gradle", "xtext", "dsl"]
---

When it comes to picking a tool for building Xtext projects many projects still prefer Maven Tycho. With the rise of Gradle as the primary build tool for many companies the game is gradually changing and developers exposed to Gradle hardly want to turn back to Maven (a full comparision between Gradle and Maven can be found [here](https://gradle.org/maven_vs_gradle/)). However, developers not familiar with it have reservations to learn Gradle as it feels a bit fuzzy at first.

The goal of this post is to show how an Xtext project can be built using Gradle and give a short overview of the artifacts generated by Xtext's new project wizard.

## Creating an Xtext Gradle project

Xtext comes with a project wizard that can generate a basic project setup using Gradle. In the following an [Eclipse Neon DSL](http://www.eclipse.org/downloads/packages/eclipse-ide-java-and-dsl-developers/neonr) distribution with Xtext 2.10.0 is used.

When calling "New Xtext Project" wizard, at the first page we need to enter some meta-data for the project itself. Use the following values:

- Project name: `com.itemis.xtext.example.domainmodel`
- Language name: `com.itemis.xtext.example.domainmodel.Domainmodel`
- Language extensions: `dmodel`

<img src="{{ site.baseUrl }}/img/posts/2016-07-29-Building-Xtext-DSLs-with-Gradle/new_wizard_page1.png"/>

At the next page, select the following values:

- Generic IDE Support
- Testing Support
- Preferred Build System: Gradle
- Source Layout: Maven/Gradle

<img src="{{ site.baseUrl }}/img/posts/2016-07-29-Building-Xtext-DSLs-with-Gradle/new_wizard_page2.png" />


After clicking on finish the wizard will create a directory structure that looks like this:

```
└── com.itemis.xtext.example.domainmodel.parent
    ├── com.itemis.xtext.example.domainmodel
    │   ├── build.gradle
    │   └── src
    │       └── ...
    ├── com.itemis.xtext.example.domainmodel.ide
    │   ├── build.gradle
    │   └── src
    │       └── ...
    ├── gradle
    │   ├── maven-deployment.gradle
    │   ├── source-layout.gradle
    │   └── wrapper
    │       ├── gradle-wrapper.jar
    │       └── gradle-wrapper.properties
    ├── build.gradle
    ├── gradlew
    ├── gradlew.bat
    └── settings.gradle
```

The root project `com.itemis.xtext.example.domainmodel.parent` wraps the subprojects together which consist, in this example, of the runtime and the generic IDE project. At this point, building Eclipse plugins is not yet supported by the generator of the wizard. It works for IntelliJ and the web integration, though.

The artifacts that differ from the Maven Tycho setting can be separated into two parts: the Gradle Wrapper (`gradlew`, `gradlew.bat`, `gradle/wrapper/*`) and the Gradle build scripts (`*.gradle`) and will be discussed separately in the following.

## The Gradle Wrapper

The Gradle Wrapper is a wonderful tool that gets rid of the pain that comes with installing a build tool and keeping it in sync across a team of developers. It is the preffered way of starting a Gradle build and has two main advantages:

- Everyone can build your project, no need to install Gradle upfront
- Everyone in your team is using the same version of Gradle, including your CI server

Try it yourself with the generated Xtext project. Simply navigate to 
`com.itemis.xtext.example.domainmodel.parent` and execute `./gradlew build` (on Unix-like platforms) or `gradlew build` (on Windows).

The script will automatically download the configured Gradle distribution and execute a build. The files that belong to the Gradle Wrapper are:

```
└── com.itemis.xtext.example.domainmodel.parent
    ├── gradle
    │   └── wrapper
    │       ├── gradle-wrapper.jar
    │       └── gradle-wrapper.properties
    ├── gradlew
    └── gradlew.bat
```

At the root level `gradlew` and `gradlew.bat` are the scripts that are platform-dependent and run the Gradle Wrapper that resides in the `gradle-wrapper.jar`. The `gradle-wrapper.properties` configures the Gradle distribution that is used, that is the version and other properties. You can find more information [here](https://docs.gradle.org/current/userguide/gradle_wrapper.html).

Pro tip: Instead of typing `./gradlew` all the time try defining a bash function as explained [here](http://blog.franzbecker.io/2016/03/28/gradle-bash-function/).

## Gradle build scripts

The build scripts (`*.gradle`) define the Gradle build for the project. They may contain configuration but also executable code. Conceptually, one can think of Gradle as in between Ant and Maven. Everything is scriptable but for most tasks there are useful plugins that need little to no configuration in order to work properly (convention over configuration).

The scripts generated by the wizard are:

```
└── com.itemis.xtext.example.domainmodel.parent
    ├── com.itemis.xtext.example.domainmodel
    │   └── build.gradle
    ├── com.itemis.xtext.example.domainmodel.ide
    │   └── build.gradle
    ├── gradle
    │   ├── maven-deployment.gradle
    │   └── source-layout.gradle
    ├── build.gradle
    └── settings.gradle
```

The main script for each project / subproject is the `build.gradle`. Unlike Maven's `pom.xml`, the `build.gradle` is optional for subprojects if no additional information is required. Subprojects are configured in the `settings.gradle`. 

The `settings.gradle` can be used to configure certain properties upfront and is evaluated by Gradle before the `build.gradle`. Subprojects are defined by using the following syntax:

```
include 'com.itemis.xtext.example.domainmodel'
include 'com.itemis.xtext.example.domainmodel.ide'
```

In Gradle, build logic can be extracted easily into separate build scripts and plugins. To include them into a project the "apply" syntax can be used:

```
apply plugin: 'java'
apply plugin: 'org.xtext.xtend'
apply from: "${rootDir}/gradle/source-layout.gradle"
apply from: "${rootDir}/gradle/maven-deployment.gradle"
```

This should give you enough information about the structure of the build to go on and understand the remaining details yourself. To learn more about Gradle you should check out their excellent [user guide](https://docs.gradle.org/current/userguide/userguide.html). 

One more remark: there is no need to keep the shown artifacts below `com.itemis.xtext.example.domainmodel.parent`. You can move everything up one level without the need to change any build logic.

## Summary

This blog post explained how an Xtext project can be set up with Gradle as a build system and gave a short overview of the artifacts that are generated by Xtext's project wizard.