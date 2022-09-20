---
title: "Understanding Gradle for Android developers"
published: true
---

![Gradle Build Tool](/assets/gradle-org-hero.webp)
*[Source](https://gradle.org)*

Hello! For a long time I wanted to migrate my [Bookcrossing Mobile](https://github.com/fobo66/BookcrossingMobile) app to Gradle Kotlin DSL. But I've used remote Groovy function for loading API keys from `.properties` files. Here is a [gist](https://gist.github.com/fobo66/17d5116b5c7bccf5f28036f401f3c09d). Despite it's a simple function, I didn't wanted to write this for each of my projects, thus this gist was created.

I wanted to share this code in more common way, via plugin, so I decided to create my own [plugin](https://github.com/fobo66/propertiesLoader). After I was done with it, I realized that it never would work in the way I wanted it to work because of how Gradle works. This situation made me think about how little I know and understand about Gradle. I've asked colleagues, and turned out that they prefer not to touch Gradle scripts much and only vaguely understand how it works.

Official docs are not clear enough and spread across different pages, and articles across the internet are often outdated. Getting the knowledge about Gradle helped me understand how Android build system works, how can we effectively configure our builds and how to effectively automate some tedious tasks, as well as debunk some myths about it. So, I decided to write this article.

For me to properly understand Gradle, it was helpful to really understand the concept of the build system. Historical references helped me much here, along with studying GNU `make` and Gradle sources. So I decided to use similar approach in this article, though I don't recommend you to dive into the source code.

First of all, I'll explain a little bit about what are build systems and what are they useful for, with small historical reference to GNU `make` as an example, then I walk you through the main features of Gradle that I found useful but unclear, providing some real-life examples.

This is not a promotional article, neither it's a comprehensive analysis of the Gradle features. I will point out to some things which I find important and underrated, and describe them with some pros and contras that I see.

To make a most out of this article, you need to know any programming language, but familiarity with Java and Android development will help, since I will use a lot of terms related to it.

## What is a build system

According to [wiki](https://en.wikipedia.org/wiki/Build_automation), build automation system (or build system, for short) is "_the process of automating the creation of a software build and the associated processes including: compiling computer source code into binary code, packaging binary code, and running automated tests_". This definition may not tell you enough, so let's look into an example.

Imagine your first Hello World application, when you just started to learn your programming language. It probably was deadly simple, written in one file and run from the command line. However, when complexity of the problem you solve grows, one file is often not enough. You may end up using multiple files, stored chaotically across the folder with your project. And when you need to use some third-party dependencies, command for compiling stuff will grow significantly, allowing pesky mistakes spill into it when someone will try to compile your project on different machine. You may end up with some `.sh` or `.cmd` files that will contain command that is required to build your project, but these scripts are not guaranteed to work on other OS or even in other folder, and you may end up spending more time fixing shell script for build than writing code.

This was even more painful in the late 1970s, when C programming language was on the rise. Software engineers were managing complex projects with lots of different files, and that files may require special treatment, e.g. generating some code before compilation. It was quite difficult to keep track of the files that need to be recompiled when changes occur, and  no one wanted to fully recompile whole project when only one file was changed, because it can take ages. That inspired Stuart Feldman to create [`make`](https://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.39.7058), one of the first build automation systems.

General idea behind `make` was this: gather source code files, together with dependencies and resources, compile them and produce executable file. If some files were changed, `make` will recompile only those files and produce new executable. These steps can be easily applied not only to the software projects, but for anything that involves files.

To tell `make` what to do, special text file is used. It's called `Makefile` and it contains a set of rules on which `make` will treat your software. With `Makefile` you can configure all the build process however you want. Wanna add static analysis before compilation, or need to convert all the images to WebP format before producing final executable file? It's a bunch of lines of shell commands in `Makefile`.

`make` helps to deal with a lot of source code files in a systematic way, and you can be sure that your software will be build on different machine exactly as it's built on yours, regardless where you put folder with your project on disk.

As an additional benefit, the rise of `make` helped create a set of conventions around project file organization that are easier for build system to process and simpler for humans to work with.

So, the concept of the build automation system can be summarized like this: _perform commands defined in config file on the given set of files._

## What is special about Gradle

`make` may look like it's a perfect fit for the job as the general purpose build system, but why there are the hell lot of other build systems?

Well, the common complaint on `make` was that it has over-complicated `Makefile` format, so project configuration often takes some time, and if you make some mistake in it, it's quite difficult to properly debug it and locate the problem. In addition, `make` doesn't offer any solution for managing dependencies: you need to manually maintain them in order somehow, and that often leads to heavily outdated dependencies and inability to update them, unless you keep an eye on them constantly. Gradle addresses these issues in its own fashion, and allows you to fully control your build process.

But with this power goes along a great amount of opaque stuff that you end up using like magic, without truly understanding why and thus limiting control and unnecessarily increasing complexity. I will uncover some of the concepts that I've misunderstood, but keep in mind that there is much more about the Gradle than I can cover in this article.

First of all, Gradle uses a bunch of terms specific to it do define its core components, so learning their exact meaning and their connections between each other will be useful. You can check [this article](https://docs.gradle.org/current/userguide/what_is_gradle.html) for the intro of the Gradle components. I will touch a few components I've got wrong initially, so you won't make the same mistakes.

### Caching

Important Gradle feature for Android developers is build cache support. It basically means that task won't be executed if its input files were not changed. Caching system is robust and you can be sure that it works just fine all the time. I've seen some developers don't trust Gradle caches and execute `clean` task on CI every build, increasing build times with no reason. For example, if you run Gradle build on different branches of your projects that have different changes in the code but with same caches, Gradle will detect changes and run tasks that have been really changed, e.g. it won't compress resources if they are same for both branches. Gradle uses virtual file system to detect changes, and since version 6.5, even OS-level file system watching. More info [here](https://blog.gradle.org/introducing-file-system-watching).

Dependency caching works in the similar way. For example, if your CI machine performs builds on different branches of your projects that have different version of the same library but with the same cache, builds will be executed correctly, because Gradle will download newer version and use it, and builds with older versions will use older version. Same goes for different Gradle versions. In short, Gradle caches dependencies by version.

However, cleaning of caches is useful to perform once in a while to ensure health of the CI machine, so it won't run out of disc space when you don't expect it. For example, GitLab has [support for cleaning up caches of its Docker executor](https://docs.gitlab.com/runner/executors/docker.html#clearing-docker-cache) and recommends to do it once per week.

### Plugins

Gradle Plugin system is great, because it allows you to automate project setup of your app for production, e.g. signing, API keys, localization and everything specific to your business. However, Gradle provides a lot of different ways to setup these things, thus devs often abuse these features, which leads to tedious scripts that are hard to maintain.

Basically, clean folder with Gradle project will contain almost no tasks, because there is no plugins applied in build.gradle file. If you want to have Java project build by Gradle, you need to apply `java` plugin first. However, if you start project from Intellij IDEA, you will get initial Gradle setup just after creating project and you will most likely have basic plugins for your language applied already. But default plugins are not always enough, and you may need to extend your project with extras that will suit your needs.

Best way to add some additional steps into your build is using plugins. In Gradle terms, plugin is a collection of tasks. You can define tasks' dependencies (e.g. in what order they should run) and tasks themselves via Java, Groovy or Kotlin code. Tasks will be run in the specified order when you apply your plugin to the needed module, e.g. `app` in case of Android. As a bonus, you will get syntax highlighting and ability to write tests for your build logic.

For example, if you need to decrypt some sensitive info inside your repo before building the app to include it into app resources (e.g. API keys), you can use a plugin for that. This plugin may expose one task that will use OpenSSL to decrypt file provided to it as an input, and plugin will specify that this task will run before anything else.

Another example: you can set up a plugin to customize JUnit HTML reports to add screenshots of the app after failed tests, so you can see why your UI test has failed. It's really helpful when you run tests on CI in headless mode, and in logs there is only exception log from Espresso that it failed to locate some button on the screen, and on screenshot it's visible that all of a sudden permission dialog appeared in front of the app. True story. (There is a way to handle this in a better fashion, but this approach is still viable if you need more flexibility in customizing JUnit reports than JUnit itself provides)

Another benefit of using plugins is the possibility to extract their source code from the main project. It's especially important when project is big. For this you have two options:

* Use `buildSrc` plugin folder.
* Publish plugin to Gradle Plugin repo or your own private Maven repo.

`buildSrc` folder is a special beast. This is a default module to define any logic that you need for other modules to build. It can contain some code that will be compiled before any other module and will be available in the build scripts for modules. You can also define plugins and tasks in `buildSrc`'s build script. For Android, people often use `buildSrc` to store dependencies' versions and package IDs to use in app's `build.gradle` in the nice way. One of the possible way to do this is described [here](https://proandroiddev.com/gradle-dependency-management-with-kotlin-94eed4df9a28). However, you can do more advanced things with `buildSrc`. As an example, you can setup API key loading from trusted source via `buildSrc` plugin and a bunch of tasks, so your keys won't leak into git history and will be loaded automatically, without manual setup, which is particularly useful on CI. You may find `buildSrc` especially useful if you don't want to fiddle with deployment of your plugin or if it's needed only for your internal stuff.

However, changes in `buildSrc` cause cache invalidation for any other module, so it may not be particularly suitable for build logic that change frequently or for huge multi-module projects. Also, Project Structure tool in Android Studio, as well as [Dependabot](https://help.github.com/en/github/administering-a-repository/keeping-your-dependencies-updated-automatically), don't recognize dependencies declared in `buildSrc` module, so you would need to keep an eye on the versions manually, and prepare for increased build times for your modules when you decide to update even one dependency version. But, as mentioned [here](https://docs.gradle.org/current/userguide/composite_builds.html#current_limitations_and_future_work), Gradle team plans to make `buildSrc` an included build, so it won't invalidate caches in the future.

## Conclusion

We tried to understand build system in general and Gradle in particular, and looked into features of Gradle build system that are often confusing - cache and plugins system.

For further reading, you can check [this repo](https://github.com/android/gradle-recipes/) with a lot of examples of Gradle tasks and plugins for Android development.

Hope you found this article useful. If you have anything to add or want to discuss some point regarding this article, feel free to open an issue on Github or reach me out in Twitter. Cheers!
