# Why would you do this?

Have you used VS Code? 

But seriously, there are lots of reasons to love Visual Studio, especially with the Community option available. With that in mind, however, it's hard to deny that VS Code is hands down a more comfortable place for a lot of developers. I myself spend as little time in Visual Studio as possible these days as more often than not, there's just a lot more overhead to loading and running Visual Studio than I feel is necessary for most tasks. They do a great job on that team, and the level of abstraction is amazing. But while making so many things easier, this makes some things harder as when you run Visual Studio for the first time, there's just so many buttons, menus, hotkeys, and functionality that you're just not going to know how to use at first. And to be honest, I don't use most of these things on a daily basis, which just feels like it's in the way. It's all helpful, but I am not the sort that likes to have all of his tools open and everywhere on the workbench.

However, support for C# is pretty much dedicated exclusively for .NET Core out of the box. What they don't tell you on the back of the box, is that it is actually possible to run and work with .NET framework apps in VS Code. It's just a little less supported than Visual Studio.

# All Right, I'm Sold

Well, hold on. There's some caveats. And whether or not they're dealbreakers for you will really depend on how into working with `.csproj`/`.sln` files or manually managing your NuGet packages you are. It's not a dealbreaker for me, and I'm going to try to find some ways around that, but these are just not going to be things you have Visual Studio hold your hand through. It's not a deal breaker for many, but for some, it's not for you and that's fine.


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

And boom, there you have it. These are all the items you need for your path. Nuget.exe should be accessible from your command line already if you have Visual Studio installed. 

## Starting a Project

If you're like me, Visual Studio will still have a place in your toolkit if you're still working in .NET Framework. It is what it is, the dotnet foundation has sort of moved on from here. You can make your own custom templates for the `dotnet` CLI, but that falls way outside of scope for this tutorial. You could also build your own solution and project files, but that also falls way out of scope for this article. The goal here is to get you up and running with most of the basics you're used to having. With that in mind, I cheat on this step BIG TIME when starting a new framework project. Crack open Visual Studio and start a new project.

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

IIS Express accepts parameters using the syntax `/{parameter}:{value}`. With that, we're passing in the config parameter and pointing it at the local `applicationhost.config` give to us by the template and passing in our site name VSCodeDemo. Althernatively, we could pass in the `/siteid:2` parameter to target this application. You'll see a new tray icon for IIS Express and now you could open this up in a browser from the tray application or opening the browser and navigating to `localhost:{port in the config}`. With all of that setup, we're now ready to start working on our VSCode configuration.

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

Ensure that the `Optimize` is set to false. Then, set the debug option to "portable". This converts your debug symbols from .NET Framework specific symbols to the newer open symbols. These are the symbols that can be read by the VSCode C# extension. 

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
            <site name="VSCodeDemo" id="3">
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

With this in your default IIS Express configuration, you can call if from any folder, without needing to specify the config file you're working in. You are now able to feed in the site index or the site name under the `/siteid:3` or `/siteName:VSCodeDemo`. And that's it! We're going to take a look at a few more things that I prefer to have next, but that's the basics. With this in place, you're good to go for most workflows. At least, these are the most essential portions of development for me that I use on the daily. All that I feel is missing from this picture is Nuget and one-button application starting. Believe it or not, Nuget was easy to transition to, so let's do that one next.

## Nuget

If you have VS installed, you more than likely already have the nuget cli installed. If you've used the nuget cli before, this will be very familiar to you. What I didn't know until very recently was that it could be used to install packages into .NET Framework projects. Navigate to your project's root directory, and give it a shot by adding Newtonsoft:

```
nuget install Newtonsoft.Json -Version 12.0.3
```

For me, this took immediately and I had my shiny new package rolling immediately. It does mean that instead of using the Package Manager GUI, you'll be using the nuget site to find packages and get the correct versions for your project, but this to me was a small trade off as I found the package manager GUI to be a little tough to use anyway. So, nuget's done and dusted. 

## VS Code Tasks

Now, the beast has risen it's ugly head. The hardest part of getting up and rolling in VS Code: Tasks. I've not managed to get this to 100% of Visual Studio, but from a day to day standpoint, what's we've done already covers about 95% of what I find myself doing. But let's see if we can't use some of VSCode's configuration goodies to get this a little further.

