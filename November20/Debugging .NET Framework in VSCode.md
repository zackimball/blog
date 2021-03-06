# Why would you do this?

Have you used VS Code? 

But seriously, there are lots of reasons to love Visual Studio, especially with the Community option available as that version is an extremely robust environment. However, there's a fundamental difference in purpose that makes me prefer VS Code, at least for most of what I do on the daily. As a developer, we mostly edit text, that's it. Actually, maybe we mostly just think about text, but that's another topic. Visual Studio is a full-bore, IDE. It's something that combines a lot of tools into one, super dense software package. VS Code is at it's heart a text editor. A very fancy, extremely powerful, super feature reach text editor that blurs the line between text editor and IDE, but still, a text editor. This means most of your tools will be BYO. I think that's why people love it. It's a simpler concept, and you're not inundated with hundreds of features that you're just not going to need all that often. I develop using both, but my "Daily Driver" as it were is VS Code. I think both serve their purpose in my workflow.

However, support for C# in VS Code is pretty much dedicated exclusively for .NET Core out of the box. What they don't tell you on the back of the box, is that it is actually possible to run and work with .NET framework apps in VS Code. Which is where we get to some of your BYO tooling. It's no doubt installed on your machine already, because who doesn't have Visual Studio if you're working with Framework. The assumption underpinning this blog is you have on hand some .NET framework project, but you're like me and have started to appreciate a simpler environment that you can get just about everything done in without having much in the way of features that are adding what feels like clutter when you don't need them. 

If you get stuck, I've got the basic working version up on Git, don't worry about what the application, it's basically just a stubbed out project from one of Visual Studio's templates. [Git: blog.VSCodeDemo](https://github.com/zackimball/blog.vscodeDemo)

Well, hold on. There's some caveats. And whether or not they're dealbreakers for you will really depend on how into working with `.csproj`/`.sln` files or manually managing your NuGet packages you are. It's not a dealbreaker for me, and I'm going to try to find some ways around that, but these are just not going to be things you have Visual Studio hold your hand through. It's not a deal breaker for many, but for some, it's not for you and that's fine. In fact, generally speaking if you're trying to work on a Framework project, a lot of what you would need the Nuget Package Manager for is already done. I will have some details on how to use the Nuget CLI, but that is a deep topic that if there's enough interest in I will tackle later. We are basically going to be using it just to pull in packages currently configured for the project.

## Adding a few things to your path

Since we're abandoning the comfortable shell of built in tools in Visual Studio, there's a few things that we're going to take with us from that installation. you can install these manually, however I'm assuming that if you're trying to convert ot VSCode, you've got Visual Studio installed. Let me know in the comments if there's any interests in walking through this from the perspective of someone brand spanking new to dotnet. 

The most important thing that you're going to need to add to your path is `msbuild.exe`. For me, that path was below, but it depends on the version of Visual Studio you've got:

```
C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin
```

However, there are a few different places that this could be installed with different versions. This is more than likely where yours is if you've installed Visual Studio 2019. 

Next, load up IIS Express into your path as well.

```
c:\Program Files\IIS Express\
```

Lastly, add Chrome to your command line. We're gonna need it to trigger a startup task later:

```
C:\Program Files (x86)\Google\Chrome\Application.
```

And boom, there you have it. These are all the items you need for your path. Nuget.exe should be accessible from your command line already if you have Visual Studio installed. 

## Starting a Project

If you're like me, Visual Studio will still have a place in your toolkit if you're still working in .NET Framework. It is what it is, the .NET Foundation has sort of moved on from here. You can make your own custom templates for the `dotnet` CLI, but that falls way outside of scope for this tutorial. You could also build your own solution and project files, but that also falls way out of scope for this article. The goal here is to get you up and running with most of the basics you're used to having. With that in mind, I cheat on this step BIG TIME when starting a new framework project (although, this blog is really the only time in recent memory that I've needed it). Crack open Visual Studio and start a new project.

For the purpose of following along, the project is going to be an "ASP.NET Web Application (.NET Framework)" project. We're going to call it `VSCodeDemo` if you're following along, and we're targeting Framework 4.8. Also for the purpose of this walk through, it's going to be a "Web Forms" type of project.

And now we will ditch Visual Studio for VS Code. We're got some serious work to do, but from here on out we're in CLIs and VS Code, so get comfy.

## First Steps

Let's go through a few of the things we're going to need to have down pat in order to keep this working. First, test our `msbuild` setup.

```
cd VSCodeDemo
msbuild VSCodeDemo.sln -m
```

That secoond command will run all of your build steps. If you ever need assistance sorting out what's available to you with MSBuild, you can run the following command and see all the subcommands available to you:

```
msbuild /?
```

All of your normal operations that are available to you in VS are available here for the most part. You can runa  clean, build, rebuild, etc. as well as include parallel builds for your larger projects. In my experience, running this outside of VS is far faster that running it within the IDE. Your mileage may vary, but this has generally been a breath of fresh air for me. Another item that I include here is the `-m` flag, which is not needed here but in future, this will provide you the ability to build multiple `.csproj` projects in parallel.

Let's also test our IIS Express setup quickly before loading into VSCode. VS makes this easy for us by including a pre-built config from the template files. These are stored in the root of your directory under the `.vs` folder. To get there, navigate to that directory:

```
cd .vs/VSCodeDemo/config
ls
```

IIS Express accepts a lot of parameters, and before we dig into those, let's gather some IIS basics. When IIS is installed, the default directory it will look for configs for is in `{user folder}\Documents\IISExpress`. This is the way it was done before we had fancy IDEs with local configs built specifically for the site we're working on. Most of the flags you use with the `iisexpress` application will attempt to read from this configuration first. This is why we're working in that hidden directory (`.vs`). We will clean this up in a future step, but this is where you start. Additionally, any time you start a project using VS, a default site is created in the config called `WebSite1`, with the site index of 1, see below from the `applicationhost.config` in our local config folder:

```xml
<configuration>
    [...]
    <system.applicationHost>
        <sites>
            <site name="WebSite1" id="1" serverAutoStart="true">
                <application path="/">
                    <virtualDirectory path="/" physicalPath="%IIS_SITES_HOME%\WebSite1" />
                </application>
                <bindings>
                    <binding protocol="http" bindingInformation=":8080:localhost" />
                </bindings>
            </site>
            <site name="VSCodeDemo" id="2">
                <application path="/" applicationPool="Clr4IntegratedAppPool">
                    <virtualDirectory path="/" physicalPath="{path to user folder}\source\repos\VSCodeDemo\VSCodeDemo" />
                </application>
                <bindings>
                    <binding protocol="http" bindingInformation="*:53933:localhost" />
                    <binding protocol="https" bindingInformation="*:44358:localhost" />
                </bindings>
            </site>
            [...]
        </sites>
        [...]
    </system.applicationHost>
    [...]
</configuration>
```

That site does not have anything tied to it, and we'll just be taking it out later, and these configs are just the defaults. We won't be tweaking them very much, and are shown for reference. The key takeaway at the moment is that sites in these configs have two important properties: names and IDs. With this information, we can return to our consoles and target this using the `iisexpress` tool.

```
iisexpress /config:applicationhost.config /site:VSCodeDemo
```

IIS Express accepts parameters using the syntax `/{parameter}:{value}`. With that, we're passing in the config parameter and pointing it at the local `applicationhost.config` give to us by the template and passing in our site name VSCodeDemo. Alternatively, we could pass in the `/siteid:2` parameter to target this application. You'll see a new tray icon for IIS Express and now you could open this up in a browser from the tray application or opening the browser and navigating to `localhost:{port in the config}`. With all of that setup, we're now ready to start working on our VSCode configuration.

## Setup In VSCode

And this is what we've all been waiting for. 

```
code .
```

Bam, blog's done, have fun, you're a wizard, Harry!

And while that's fairly true, there's this nasty little thing called debugging that will be much easier if we can set it up so we can debug it in VSCode. We could rely on the ASP.NET debugging pages, but that's going to be a nightmare, plus all of these configs are everywhere and that's going to be an absolute headache to manage, and we don't like those.

First things first, if you haven't, install the C# extension. Out of the box, it will only support Core projects, and that's one of the caveats here. You'll lose some debug functionality, but I haven't felt that pinch. In VSCode, open up `VSCodeDemo/VSCodeDemo.csproj`.

Look for this section and this is where we will be making some changes:
```
[...]
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <DebugType>full</DebugType>
    <Optimize>false</Optimize>
  </PropertyGroup>
[...]
```

```
[...]
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <DebugType>portable</DebugType>
    <Optimize>false</Optimize>
  </PropertyGroup>
[...]
```

Ensure that the `Optimize` is set to false, this caused me some issues when I didn't spot it as being an issue. Then, set the debug option to "portable". This converts your debug symbols from .NET Framework specific symbols to the newer open symbols. These are the symbols that can be read by the VSCode C# extension. 

Now, add a folder specific for the VSCode configuration in your project's root directory call `.vscode` and a `launch.json` file. For this config, we're going to add the following:

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": ".NET Core Attach",
            "type": "clr",
            "request": "attach",
            "processName": "iisexpress"
        }
    ]
}
```

The key portions to pay attention to here is that by default, VSCode will attempt to set this up with more detail specifically for .NET Core apps. I pull out everything that is in there that I don't use, but we will add a few back in as we go. The key attributes are this is specifying that we should use the normal, non-core `clr`, the request we will send from the debugger is the `attach` command, which will have the debugger attach to an already running application. With the debugger, we have the option to target a `processId` or `processName`, and we will only be running one instance of the process and won't know the PID in advance, so we're specifying the `processName` as `iisexpress`. This will have the debugger look for that process by name, attach to it and load the appropriate symbols for breakpoints and console writes. This will cover the .NET side of debugging, at least. I'm still working on JS debugging.

Set a breakpoint in your `About.aspx.cs` on the `Page_Load` method to test this out.

Now, to get this rolling, back to the commandline. What I normally do is have one terminal window open in the root and one open within the .vs\VSCodeDemo\Config directory. This allows for me to run my build steps and restart IIS Express without having to change directories so often. We will fix this in a minute, but this has been my workflow for a while and it's fairly comfortable, but I want to tweak it and we'll get to that in a minute. For now, open a second terminal, and navigate to your project's root folder.

```
msbuild VSCodeDemo.sln
```

This will build your solution. Now, switch to your other terminal, which should hopefully be in your VS config folder, and run:

```
iisexpress /config:applicationhost.config /site:VSCodeDemo
```

Navigate to the site in your browser and see that the breakpoint is hit. If it's hit, you're good to go! That means the debugger is able to listen to the new portable format of the debugging symbols and attached to the IIS instance with no problems. With that together, I would prefer to be able to start this site from somewhere other than this odd, buried directory that I'm going to forget about. Open up that `applicationhost.config` in VSCode and grab the site's specific details. Should look something like below:

```xml
<site name="VSCodeDemo" id="3">
    <application path="/" applicationPool="Clr4IntegratedAppPool">
        <virtualDirectory path="/" physicalPath="C:\Users\icarus\source\repos\VSCodeDemo\VSCodeDemo" />
    </application>
    <bindings>
        <binding protocol="http" bindingInformation="*:53933:localhost" />
        <binding protocol="https" bindingInformation="*:44358:localhost" />
    </bindings>
</site>
```

With the power of copying and pasting, grab this site and open the default IIS config for your machine. It's hosted in `/Documents/IIS Express/applicationhost.config`. Find the "sites" section and paste it in. You should now have a section that looks similar to below. Ensure that when you paste this in, you update the ID and the name to be distinct from what is already present.

```xml
<system.applicationHost>
    [...]
        <sites>
            <site name="WebSite1" id="1" serverAutoStart="true">
                <application path="/">
                    <virtualDirectory path="/" physicalPath="%IIS_SITES_HOME%\WebSite1" />
                </application>
                <bindings>
                    <binding protocol="http" bindingInformation=":8080:localhost" />
                </bindings>
            </site>
            [...]
            <site name="VSCodeDemo" id="2">
                <application path="/" applicationPool="Clr4IntegratedAppPool">
                    <virtualDirectory path="/" physicalPath="C:\Users\icarus\source\repos\VSCodeDemo\VSCodeDemo" />
                </application>
                <bindings>
                    <binding protocol="http" bindingInformation="*:53933:localhost" />
                    <binding protocol="https" bindingInformation="*:44358:localhost" />
                </bindings>
            </site>
            [...]
        </sites>
        <webLimits />
    [...]
    </system.applicationHost>
```

With this in your default IIS Express configuration, you can call if from any folder, without needing to specify the config file you're working in. You are now able to feed in the site index or the site name under the `/siteid:2` or `/siteName:VSCodeDemo`. And that's it! We're going to take a look at a few more things that I prefer to have next, but that's the basics. With this in place, you're good to go for most workflows. At least, these are the most essential portions of development for me that I use on the daily. All that I feel is missing from this picture is Nuget and one-button application starting. Believe it or not, Nuget was easy to transition to, so let's do that one next.

## Nuget

If you have VS installed, you more than likely already have the nuget cli installed. If you've used the nuget cli before, this will be very familiar to you. What I didn't know until very recently was that it could be used to install packages into .NET Framework projects. Navigate to your project's root directory, and give it a shot by adding Newtonsoft:

```
nuget install Newtonsoft.Json -Version 12.0.3
```

For me, this took immediately and I had my shiny new package rolling immediately. It does mean that instead of using the Package Manager GUI, you'll be using the nuget site to find packages and get the correct versions for your project, but this to me was a small trade off as I found the package manager GUI to be a little tough to use anyway. If you have pulled down an already established Framework project, you can do a `nuget restore` command in the root of your project and it will pull in those packages as well.

With that, nuget's done and dusted.

## VS Code Tasks

Now, the beast has risen it's ugly head. The hardest part of getting up and rolling in VS Code: Tasks. I've not managed to get this to 100% of Visual Studio, but from a day to day standpoint, what's we've done already covers about 95% of what I find myself doing. In general, it's been easier to run the commands manually in my terminal, but VSCode has this really interesting set of tooling around starting up the application, so I went ahead and worked out how to get the process running using this method. I, personally, would probably use the set up that has been put into place so far, but this is the last mile so let's finish this.

VS Code relies on a launch.json to manage your debugging/application runnning. For this application, this is how I've configured that launch.json:

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach to Chrome",
            "port": 9222,
            "request": "attach",
            "type": "pwa-chrome",
            "webRoot": "${workspaceFolder}"
        },
        {
            "name": "Start App",
            "type": "clr",
            "request": "attach",
            "processName": "iisexpress",
            "preLaunchTask": "startChromeDebug",
            "postDebugTask": "iisStop"
        }
    ],
    "compounds":[
        {
            "name": "Launch apps and attach debuggers",
            "configurations": [ "Start App","Attach to Chrome"]
        }
    ]
}
```

As a basic rundowm, there are two configurations present: one attaches the debugger to Chrome, and the other calls all of the startup processes. In this case, they are referencing a `tasks.json` file that runs various commands that I've placed in there. These are chained together in a way I will show in a minute. The important thing to note here is that there are two tasks in the "Start App" configuration. There is the pre-launch task, which is run before trying to attach the debugger, and there is the "postDebugTask", which is a cleanup process to stop the instance of IISExpress I am running. 

Initially when I was putting this together, though, I couldn't figure out how to attach both debuggers (the chrome debugger and the C# debugger). These configurations were run either one or the other, which was workable but not the full picture I was hoping for. Turns out, VS Code supports running multiple debug configurations through something they called "compounds". In this section, we define the combination we want to run, in this case first "Start App" followed by "Attach to Chrome". The first one will run scripting to build the application, start up IIS Express, and start Chrome with the debugging port open. The second one will attach the Chrome debugging extension to the instance of Chrome we just opened. Additionally, I've added a tear down task to the "Start App" configuration that will stop the instance of IIS Express we're running. 

Now, let's take a look at these tasks.

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "msbuild",
            "type": "shell",
            "command": "msbuild",
            "args": [
                "/p:Configuration=Debug",
                "/t:build",
                "-m"
            ],
            "presentation": {
                "reveal": "silent"
            },
            "problemMatcher": "$msCompile"
        },
        {
            "label":"iisStart",
            "type":"shell",
            "dependsOn":"msbuild",
            "command":"Start-Process",
            "args": ["iisexpress","/siteid:2"],
            "presentation": {
                "reveal": "silent",
            }
        },
        {
            "label":"iisStop",
            "type":"shell",
            "command":"Stop-Process",
            "args": ["-Name","iisexpress"]
        },
        {
            "label":"startChromeDebug",
            "type":"shell",
            "command":"chrome",
            "dependsOn":"iisStart",
            "args":[
                "https://localhost:44315/",
                "--remote-debugging-port=9222"
            ]
        }
    ]
}
```

The first task is the msbuild task, which will call the msbuild exetuable and run it to build our debug configuration.

```json
{
    "label": "msbuild",
    "type": "shell",
    "command": "msbuild",
    "args": [
        "/p:Configuration=Debug",
        "/t:build",
        "-m"
    ],
    "presentation": {
        "reveal": "silent"
    },
    "problemMatcher": "$msCompile"
}
```

To break this down, what happens with the task is it is fed into your CLI, in my case and in most it'll be Power Shell by default. The label is what is used to identify it within the your configurations in `launch.config`. The command is, in this case, the `MSBuild.exe`, which we've added to our path (for me, it was located here: `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin` since it's installed alongside of Visual Studio Community 2019 there). Your `args` are built off of what parameters are fed into the application. Basically, if you can learn how to use a CLI tool, it can be configured here. The flags we're using, as it's for a debug build, specify the flags `/p` for "property", which will locate the "Configuration" property in our `.csproj` file and build accordingly. The `/t` flag will target a process, here we're targeting build, but you can also use "rebuild" or "clean" if needed. Essentially, here is replicated the "Build" button within Visual Studio.

```json
 {
    "label":"iisStart",
    "type":"shell",
    "dependsOn":"msbuild",
    "command":"Start-Process",
    "args": ["iisexpress","/siteid:2"],
    "presentation": {
        "reveal": "silent",
    }
}
```

Now, since we're using the PowerShell CLI, in addition to calling the the `msbuild.exe` application, we can also add in PowerShell commands. Setup is the same: the label we're going to call it, the type being "shell" to designate it as a CLI task. The first "new" item is what allows us to chain tasks together in VSCode. The `dependsOn` flag specifies what other tasks need to precede this task in order for it to work. Also, since IISExpress runs continuously until stopped, instead of just calling `iisexpress` from the command line as we normally do, we're going to call `Start-Process`, which allows for control to return to the remaining tasks after the process is started. The args, in this case, are the same ones we've been using to run the site so far. The `presentation/review` flag, in this case at least in my testing, doesn't seem to do what I think it does (I thought it ran it in the background, I believe this is just supressing errors).

```json
{
    "label":"iisStop",
    "type":"shell",
    "command":"Stop-Process",
    "args": ["-Name","iisexpress"]
},
```

This piece uses PowerShell's `Stop-Process` call in order to end IISExpress when we're done. This is referenced in the `postDebug` flag inside of our `launch.json`. This task works the same, so if you had multiple tear down processes you could effectively configure them here with `dependsOn` flags.

```json
{
    "label":"startChromeDebug",
    "type":"shell",
    "command":"chrome",
    "dependsOn":"iisStart",
    "args":[
        "https://localhost:44315/",
        "--remote-debugging-port=9222"
    ]
}
```

The final task we're adding requires for you to add your Chrome browser to your path so that you can call it from your CLI. For me, it was located here:  The parameter you'll need to add is the site's URL from your ApplicationHost.config and the flag enabling your remote debugging port. 

Phew, that was a lot of configuration. However, if you go to your "Debug" tab in your VSCode workspace, you now have multiple configurations available, including in this case a new one labeled "Launch Apps and Debuggers". That's our new compound task that combines ALLLL of this business together, starts your IIS Express, builds the things, starts Chrome, and attaches both the .NET debugger and your Chrome debug tooling.

## Regrets

I have some regrets here, really I do. I thought this would be way simpler and honestly this blog has sucked up a ton of my time with headaches. The vast majority of this time was spent trying to figure out how to get IIS working, I'm really not that great at PowerShell or CLI tooling. Working at it every day, a lot of it gets easier, but for me it still seems simpler (if less efficient) to just run a manual build in the CLI followed by starting IISExpress. It's easy enough to configure the debugger to attach to the IIS Express instance without all of these tasks being wired up, but I feel like these tasks will make someone's QOL better. And I know for a FACT there's dozens of blogs out there getting into each piece of this in major detail, but I have always struggled to understand all of this because often times what I see is the author explaining the thing that they were stuck on instead of walking through the whole process. That's what I wanted to do here, but trust me, I have to thank the internet and all of you command line commandos out there who had snippets of various pieces of the process. 
