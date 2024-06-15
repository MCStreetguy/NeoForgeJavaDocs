# Generating NeoForge JavaDocs

This document describes my personal method for generating the NeoForge JavaDocs.
It is by no means a very stable or efficient approach and i make no guarantees that this will continue to work in the future.
It is based on a custom Gradle task, added to the official Forge repository source to generate a combined JavaDoc for all packages.

## Requirements

Please make sure that you have the corresponding Java Development Kit version installed for the Forge version you want to address!

## Preparation

1. Clone the [official NeoForge git repository](https://github.com/neoforged/NeoForge) and check out the tag of the version you want to address.  
   This can be simply done using the following command:  
   `git clone --depth=1 --branch=<TAG_NAME> https://github.com/neoforged/NeoForge.git`
2. Fully setup the workspace by running the `./gradlew setup` (or `.\gradlew.bat setup`) command in the project directory.
   This step is crucial as it downloads and decompiles the actual Minecraft sources required for the project to work.

## Generator Snippet

Open the `build.gradle` file from the project root directory in your favorite editor and append the following snippet at the end of the file:

```gradle
apply plugin: 'java'

subprojects {
    apply plugin: 'java'
}

afterEvaluate {
    def neoForgeVersion = "20.2.88"
    def exportedProjects = [
        ":",
        ":neoforge",
    ]

    task alljavadoc(type: Javadoc) {
        options.tags = [
            'apiNote:a:<em>API Note:</em>',
            'implSpec:a:<em>Implementation Requirements:</em>',
            'implNote:a:<em>Implementation Note:</em>'
        ]
        options.addStringOption('Xdoclint:none', '-private')
        options.addStringOption('locale', 'en_US')
        options.addStringOption('encoding', 'utf-8')
        options.addStringOption('docencoding', 'utf-8')
        options.addStringOption('charset', 'utf-8')
        options.addStringOption('windowtitle', "NeoForge ${neoForgeVersion} Modding API for Minecraft ${project.minecraft_version}")
        options.addStringOption('doctitle', "NeoForge ${neoForgeVersion} Modding API for Minecraft ${project.minecraft_version}")

        source exportedProjects.collect { project(it).sourceSets.main.allJava }
        classpath = files(exportedProjects.collect { project(it).sourceSets.main.compileClasspath })
        destinationDir = file("${buildDir}/docs/javadoc")
    }
}
```

You must adapt this snippet slightly:

1. Make sure all required projects are included in the `exportedProjects` list.
2. Also make sure that no projects are included, that do not exist in the workspace!
3. Change the NeoForge version in `windowtitle` and `doctitle` to the one you're buildings the docs for.
4. _(optional)_ Change the `destinationDir` path if you want to store the docs somewhere else than the default `build/docs/javadoc` directory.

## Building

All you still have to do now is to run `./gradlew alljavadoc` (or `.\gradlew.bat alljavadoc` respectively).
This will go through all projects and generate a combined JavaDoc for all of them in the configured location.

If you are working on a system with a non-english locale, you will need to force Java to use english for the JavaDoc by setting the env variable `JAVA_TOOL_OPTIONS="-Duser.language=en"` for Gradle prior to building.

Please note that a lot of warnings will be thrown during the process, but these can safely be ignored.
Most of them complain about missing reference links or inproper formatting and do not really affect the result.
As long as you get no errors you should be fine.

## Troubleshooting

If you run into any errors:

1. Make sure the `exportedProjects` list is correct! _This is the most common source of errors in this process._
2. Make sure all options have been changed correctly and that the overall syntax of the file is still valid.
