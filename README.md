
As you might or might not know to make [Onikira: Demon Killer](www.onikira.com) we use an engine called [Duality](http://duality.adamslair.net/) and a few weeks ago we started to port the runtime to android. 

## Starting out

As we started we made a list of things that we needed to look at

* Shaders
    * GLSL
        * All shaders need to be update to min #version 300
        * Fixed function methods like ftransform need to be removed and replaced with manual multiplication of transformation matrices
        * Shader in and out variables need to be defined for passing data from the app through the vertex shaders and into the fragment shaders
    * Engine
        * the engine should by default pass a bunch of predefined variables to each shader
        * predefined variables should be defined for common things like modelViewProj
        * the Shader class should try and pass all common variables to the contained shader at bind time
        * the engine needs to have a central place where common variables live that can be written to by the draw device when variables change (BeginRendering most likely) and read from by the shader classes.
* Rendering
    * Currently uses fixed function vertex attribute bindings in the Vertex* classes. These need to be updated to use generic vertex attributes.
* Windowing
    * need to specify the min GL version when creating the GameWindow class (DualityLauncher)

These were a conversation starter. 


### First steps

### OpenTK

The first thing we did was to try to get OpenTk running. Here the awesome [Dean Ellis](https://twitter.com/infspacestudios) gave us a huge hand by talking to us and then pushing some code to the repo. 

Duality uses a fork of OpenTK, so to port it we forked too [here][1].

* add openTk.Android project 
* add a sample project to know everything is working
* ...


### Farseer 

Duality depends on Farseer so we also had to fork this project. The process so far has been super simple



### Difficulties

One of the main hurdles we found so far was to 

## Diary

Since we started I have been keeping a "diary" of what is going on for future reference. I want to try to document the process to hep any other porting efforts.

### 12th May

Today we finally got a texture rendering and using OpenGL 4.4 via OpenTK.
We also decided that all the repositories will have their master untouched (as it was from any original fork) and that all android related changes will go into the branch 'android-port'

### 13th May 

Today we are working on other default shaders rendering and input.

### 14th May

Today I am the only one working on the port... boo. I had to create a special project to use some hardware related jar and that brought me into the fun realm of consuming a jar from C#.
To do that first I created an Android Java Binding Library following the steps [here](https://developer.xamarin.com/guides/android/advanced_topics/java_integration_overview/binding_a_java_library_(.jar)/) I was able to build this however it wasn't working as reported [here](https://forums.xamarin.com/discussion/42051/issues-binding-jar-file-a-few-classes-not-available#latest) 

### 15th May - 31st May

... forgot to document until now (sorry :( )
we made progress on the library binding, in the end we wrote the binding code by hand :/ 
Can't remember what other changes happened. Re-booting the diary

### 1st June - input clean up

Cleaning up the input interface, the question here is should it go on the android branch or on the master? it seems like the android branch would be a better place, as it will.. ideally in the future be merged into the main

Maybe knowing how things depend on each other would help
![dependecygraph](https://cloud.githubusercontent.com/assets/3103/7919719/305fa5da-0892-11e5-9679-9c5f94602037.jpg)

# Summary

We are porting Duality runtime to Android, all the code is available on this [user][1] on the respective android-port branch of each repository. If you are interested in this and want to help or have questions, let us know.

[1]:https://github.com/batbuild?tab=repositories
