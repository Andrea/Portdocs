
As you might or might not know to make [Onikira: Demon Killer](www.onikira.com) we use an engine called [Duality](http://duality.adamslair.net/) and a few weeks ago we started to port the runtime to android. 
## Documenting the port of Duality game engine
### Starting out

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
![dependecygraph](http://i.imgur.com/SPZYwfx.jpg?1)

### 2nd June -  Input implementation

So after a refactor I noticed there were several not implemented methods, noticed a pronounced lag on the inputs, investigating...
At the same time trying to figure 

a)  Does Xamarin android support [Fast Memeber]
b) Will Binary serialization just work ( it seems like it should)
c) how to make the controller/gamepad work via USB

### 3rd June - Answers to some questions

We were wondering 

* Would Binary serialization work? so I created a scene, serialized it to Binary and tried to run it on the android device, it worked :D 
* Does Reflection work on android? I tried this as well and I was able to reflect on a type and create it. Also spiked FastMember support and it works
*  Started Fmod integration

### 4th June

* Working on the Fmod integration: 
   * Created binfing library with no issues
   * Adding the *.so (wasn't sure what was the architecture of the target hardware) to find out you can run 
                 ` adb shell cat /proc/cpuinfo`

   * If adb gives you errors about too many connected devices, make sure no emulators are running :D
   * in the end the jar file was not necesary, however I used one of the *.so, and you needed two :( the second one was on the **fmoddesignerapi** 
   * for the moment the project is loading from a directory I copied to the device and it's hardcoded,, loading from assets doesn't work on the ways I have tried
   
### 5th June

So the fastest way to deal with the asset problem has been to save the file to the device on init/load, not loving this (suggestions welcome) but it does work.


Now I am working on making [DualityScripts](https://github.com/BraveSirAndrew/DualityScripting) also be able to compile the C# and  F# scripts also target android.
Initially I was wondering if there was a way to target android with F# Compiler Services, but not the case, so I am just going to go ahead and create an android project that will be edited programatically to add all the script files and also edit the references and then build with msbuild :/

### 6th June

Today I spent some time  fixing a bug on the scripting plugin project, it was important to fix it because I needed to make sure that we had a good way to build the scripts that live in `Data/Scripts`the tricky thing here is that we need to be able to build for multiple targets in different conditions. 
So today I fixed that bug and went over what was the current state of how we build al these and determined that I can use an fsx file to build this but will need to rely on the existance of an android project file per language to produce the assemblies, so it all points to refactor the current script so that we use FAKE so that there is clean control over the targets

### 7th June

As I was saying yesterday this build script needs to work on the following environments

* teamcity build 
   *  target PC on release more
   *  target Android on release mode
* local build
   *  target Android on any mode
   
The hard thing about building these is that these DualityScripts are resources (ie xml files) that contain code and have Duality related dependencies

I know this is not stricly part of the port but it is part of a set of problems that present itself during this process and I think it is important to make sure people working on the project can work as effectively as possible with as little disruption possible

* ~~created the empty C# and F# projects~~
* ~~disasembled current script into Fake targets~~ this is a little harder than it might appear at first because of paths also I am hoping the build fails when something goes wrong :D

### 8th June

Checking paths and verifyign the build fails as it should 
(I also had to fix other unrelated issues )

## 9th of June

* Add build script for openTK to Teamcity (builds and pack and push to myget)

Build info
- for running xamamrin stuff on a build server via the command line you will need a business license
- you need to run a tool called `xamarin-componenet.exe login <your emall address>` so that the server knows your license info (found this on the [Greg Shackles talk about CI for xamarin](https://www.youtube.com/watch?v=Awl4vGo7Yj0))
- nuget can be a PITA , but we all knew that

## 10th june

* Package openTK into a nuget package
* Create other builds some gotchas:
    * Component model .exe :(
    * When using FAKE and TC you ll need a TEAMCITY_VERSION env variable
    * For nuget packages targeting android use  `target="lib\MonoAndroid`
    * don't mess up your paths :(

## 11th june

After some investigation, fixed the issues with package creation, just to discover ~~issues with MyGet too :|~~ my idiocy as I aparently managed to edit out a part of the url :( (uses package at the end when pushing the package)
Creating farseer package now, also created Duality package 


## 12th june

- Created the Fast member build and package
- Scrapped? copied? the teamcity server with a tool called [Htttrack](https://www.httrack.com/) so that I can show current status in http://batbuild.github.io/PortStatus/  not happy with the results
Need to come up with a plan for the rest of the port.     

## 13th june

Starting attempt to compile game project (code name Honourbound and this is the name I ll use here to refer to it)
Need to make the following available as android libs:

* ~~DisciplineOAk~~
* ~~Flow~~
* spine-runtime

Maybe
* Exceptionles
* scriptingPlugin
* protobuf-net
* lz4
* OgvVideo Player

Some notes tht I forgot to note before:
- VS + resharper will show code as failing or missing references when in reality everything is fine, the compiler will tell you the truth
- Sometimes something will not run on VS and will run and deploy fine on XS
- Nuget packages do work on android projects however they need to have `lib\MonoAndroid` as their target.
- On android, no resolution changing code
- On android, no editor or debugging support code 

### 15th June

By port I mean: created project and build scrips and build on TC and it builds and pushes a  nuget package
So I ported the following:

* Spine runtime
* Mercury particle engine (cross fingers it works :D)

## 20th June

Made the necesary classes available from Duality Scripting for Android and made the build availalbe

## 22nd June

Made changes to OgvVideo player so that it would be availabe from Android (ie compiles, but wont play video, for sure, bu tI gueess we ll get there)
- build server public
- Honourbound builds ( throwing a few exceptions on purpose on areas I had no idea what to do
- Learning how to launch the project (created a project launcher)
- Might be useful https://stackoverflow.com/questions/14501562/dynamically-load-dll-from-android-assets-folder

## 23rd June

- Checked asset size restriction is not a problem
- added the msbuild target so that it will copy files to /obj/ (to build the apk) linking won't work as the files need to be there

To add data to the apk using a msbuild task that copies the data from DualityData folder (path dependency) to obj/ as the data needs to be present (linking is not enough)

#25 June

Made honourbound android sln (only contains coreplugin and gameplay core plugin) 
I wasnt sure if removing the tests was a good idea, maybe It will bite me in my arse but... only one way to find out right?

#26th June  

Patching up the ogv video player because I am pretty sure we dont need to use it in production.



# Summary

We are porting Duality runtime to Android, all the code is available on this [user][1] on the respective android-port branch of each repository. If you are interested in this and want to help or have questions, let us know.

[1]:https://github.com/batbuild?tab=repositories
