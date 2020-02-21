# Example Rift mod

This is a minimal Rift project. The only use of it is to be copied to start a new project. 

## Requirements:
- OpenJDK 8. Currently Minecraft doesn't support Java 9 and above.

## Gradle tasks
- Development:
    ```sh
    ./gradlew setupDecompWorkspace [eclipse,idea]
    ```
    After this, import in your IDE as Gradle project.

- Build:
    ```sh
    ./gradlew build
    ```

## Getting started

Important files in MDK:

```sh - sh to highlight comments
src / main / java /      # code 

src / main / resources / 
        riftmod.json     # what classes to instantiate and call 
        pack.mcmeta      # resource pack description
        assets / modid / # blockstates/, sounds/, textures/, etc 

run /                    # dev env runs here; screenshots/, mods/, etc 

build /                  # build files 
        libs /           # compiled archives 

build.gradle             # how to build & dependencies 
```
<br>
From there, you'll probably want to change a few things:  

- `build.gradle`
    ```groovy
    /* rest omitted */
    group 'example' // your root package, e.g. 'org.dimdev.halflogs'
    version '1.0'
    archivesBaseName = 'Example'
    ```

- `src/main/resources/riftmod.json`
    ```json
    {
        "id": "example",
        "name": "Example",
        "authors": [
            "ExampleAuthor"
        ],
        "listeners": [
            "example.ExampleListener",
            "example.ModInitializationListener"
        ]
    } 
    ```
    To load listener only on specific side or to change loading order:
    ```json
    {
        "listeners": [{
            "class": "example.ExampleListener",
	        "side": "client",
	        "priority": 10
        }]
    }
    ```
    Inner classes require `$` instead of `.`, e.g. `example.Example$InnerClass`. 
    

- `pack.mcmeta`
    ```json
    {
        "pack": {
            "pack_format": 4,
            "description": "Example"
        }
    }
    ```
    This description is shown at *Select Resource Packs* screen. If mod doesn't have any resources, you can safely delete this file.

## Mixins
- If you want to change the name of `mixins.modid.json`, you need to change all these places:
- `build.gradle`
    ```gradle
    mixin {
        defaultObfuscationEnv notch
        add sourceSets.main, 'mixins.example.refmap.json' //change 'example' into your modid
    }
    ```

- `src/main/resources/mixins.example.json`
    ```json
    {
      "required": true,
      "minVersion": "0.7.11",
      "compatibilityLevel": "JAVA_8",
      "target": "@env(DEFAULT)",
      "package": "example.mixin",
      "refmap": "mixins.example.refmap.json",
      "mixins": [
      ],
      "client": [
        "MixinLoadingGui"
      ],
      "server": [
      ]
    }
    ```
    
- `src/main/java/example/mixin/ModInitializationListener.java`
    ```java
    @Override
    public void onInitialization() {
        MixinBootstrap.init();
        Mixins.addConfiguration("mixins.example.json");
    }
    ```

## Tips

- Changing `src/main/java` to `src` and `src/main/resources` to `resources`:
    ```gradle
    sourceSets {
        main {
            java.srcDirs = ['src']
            resources.srcDirs = ['resources']
        }
    }
    ```
- Running under Intellij IDEA:<br>
    Click *Add Configuration...*, set *Main class* to be `GradleStart`, with *Program arguments* `--tweakClass org.dimdev.riftloader.launch.RiftLoaderClientTweaker`(run client) or `--tweakClass org.dimdev.riftloader.launch.RiftLoaderServerTweaker`(run server)

- Running Server under Eclipse:<br>
    Open *Launch Configurations*, select *yourmod*\_client, switch to *Arguments* tab, change program arguments from `...Client...` to `...Server...`:
    ```
    --tweakClass org.dimdev.riftloader.launch.RiftLoaderServerTweaker
    ```
