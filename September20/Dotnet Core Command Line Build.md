# System.CommandLine

One of the most common applications I've found myself writing has been backend services. Writing little console applications that get posted up a schedule and shift things around. Recently, however, I had the distinct pleasure of writing some developer tools for a custom framework I was working in. This requried a basic console app, but we would need to be able to assign it to the path and send in parameterized values. This lead to the discovery of Dotnet Core's take on a command line parser.

It may just be me, but command line parsing is annoying. I don't like doing it, and I always reach for what felt like the indomitable package CommandLineParser, which I encourage you to checkout over on [NuGet](https://www.nuget.org/packages/CommandLineParser/2.9.0-preview1). It's very handy, and someday maybe I'll circle back. However, there's something to be said to using "native" options where possible.

## Setup

Carrying forward from our last time, let's start a new project. And while we're at it, let's make some coffee. We're going to use git to track our progress, but feel free to skip anything prefixed "git" if you won't be while following along with that.

```shell
mkdir coffeemaker
git init
cd coffeemaker
dotnet new console
```

This creates a new console project, and as you're in the coffeemaker directory, it'll be labelled "coffeemaker".

With recent changes in .NET Core, our package manager console has been moved into a new home: you guessed it, the .NET CLI. It's rather convenient.

```shell
dotnet nuget --help

--OUTPUT--

NuGet Command Line 5.7.0.7

Usage: dotnet nuget [options] [command]

Options:
  -h|--help  Show help information
  --version  Show version information

Commands:
  add      Add a NuGet source.
  delete   Deletes a package from the server.
  disable  Disable a NuGet source.
  enable   Enable a NuGet source.
  list     List configured NuGet sources.
  locals   Clears or lists local NuGet resources such as http requests cache, packages folder, plugin operations cache  or machine-wide global packages folder.
  push     Pushes a package to the server and publishes it.
  remove   Remove a NuGet source.
  update   Update a NuGet source.

Use "dotnet nuget [command] --help" for more information about a command.
--/OUTPUT
```

Without invoking the package manger console in Visual Studio, we can handle pretty much all of our operations when it comes to package management. It's almost like Node and NPM!

> But, that's cool, uh, how to I find things?
> NuGet has a discoverability issue. Really does, not a huge fan of this part, but with so many projects on Git these days, it's very easy to find new things there. Nuget's site has searches, but it's a workflow breaker. Sort of. There is an extension available that I found through [GitHub](https://github.com/billpratt/dotnet-search) that will add searching, but I haven't tried it yet. This is solely to illustrate the best way to find NuGet packages is.....not NuGet....

Since we know the package we want to add, we're going to go ahead and add it.

```shell
dotnet add package System.CommandLine --version 2.0.0-beta1.20427.1
```

And when we `tree` this directory:

```bash
tree
--OUTPUT--
.
├── Program.cs
├── bin
│   └── Debug
│       └── netcoreapp3.1
├── coffeemaker.csproj
└── obj
    ├── Debug
    │   └── netcoreapp3.1
    │       ├── coffeemaker.AssemblyInfo.cs
    │       ├── coffeemaker.AssemblyInfoInputs.cache
    │       └── coffeemaker.assets.cache
    ├── coffeemaker.csproj.nuget.dgspec.json
    ├── coffeemaker.csproj.nuget.g.props
    ├── coffeemaker.csproj.nuget.g.targets
    ├── project.assets.json
    └── project.nuget.cache
```

Taking a look at it, everything seems to be in a nice, tight location as if we built everything using Visual Studio. There's a few interesting bits in here I'd like to tuck into for a second.

The AssemblyInfo has been moved to the obj\Debug\{framework} folder. This is excellent if you're targeting multiple Core standards. The AssemblyInfo.cs file is what keeps all of your compile information: copyrights, application names, versioning, etc. When we installed the System.CommandLine, several items were added to help track that information, the most import of which is in project.assets.json. This is what tracks the version of dependencies your using, their dependencies, where they're located (since you've installed them locally, they'll be relative paths), etc. Essentially, this means that if you build this application and source control it, there's no need to include them in your source control. Speaking of, before we forget:

```shell
dotnet new gitignore
```

The folks building the CLI really thought of everything. I am not a fan of making all kinds of updates to the .gitignore. I always forget things, but here we can, in one line, grab pretty much everything we need. If you find yourself with some time on your hands, take a look at it. It's an excellent example of what you should keep out of your source repo as they just produce bloat for commits.

# That was fun, where's my coffee

Before we get too far along, let's start by adding a place to keep all of our coffee items tidy:

```shell
mkdir Brew
cd Brew
touch Brew.cs
code Brew.cs
```

Pop open Brew.cs in your favorite text editor, the above example will open it in VS Code. I've used nearly a half dozen different editors, and while it took some getting used to, I highly recommend giving it a chance. It's so light and customizable, I barely think of using other tools any more. Except maybe Vim, but that's more of my white whale. I have a complicated relationship with Vim.

Up at the top of your new `Brew.cs` file, go ahead a pull in the libraries we will need.

```c#
using System;
using System.CommandLine;
using System.CommandLine.Invocation;
```

Adding a namespace, let's go with Brewing for now. One of the things that I place a premium on is organization, and namespaces help you do that. If you're familiar with some basics of object oriented programming, think of namespaces as the containers you keep the objects in. They should group similar things together. Like a toolbox. It won't do you much good unless you have a very small collection of tools to place the same tools for electrical work as wood work. That's what a namespace does for you.

```c#
namespace Brewing
{
}
```

Inside the brewing namespace, we're going to add our first command. since the application's name is coffeemaker, the way we'll input commands is as follows:

```shell
coffeemaker brew
```

So with this in mind, let's go over some ground rules on how to use System.CommandLine.

## System.CommandLine

This library is tailored to take in parameters from the shell. The elements it expects are:

```
[application] [verb] [options]
```

In our above example, with the coffeemaker, the application is coffeemaker, and brew is the first verb or action the application can do. Let's add a target action to take.

```
coffeemaker brew --Type PodSystem
```

With this arrangement, we're going to have the root command be the application. The child command, nested within the top level, is `brew`, and we're asking specificaly for the PodSystem style of coffee. We're in a hurry, after all. Just give us the gross bean water that gives life to our code and joy to our hearts! (Maybe I've had too much.)

Let's start wiring this action up inside of the Brewing namespace, shall we?

```c#
namespace Brewing
{
    public enum BrewStyle
    {
        FrenchPress,
        PodSystem
    }
    public class Brew : System.CommandLine.Command
    {
        public Brew(string name, string description = null) : base(name, description)
        {
            Add(new Option<BrewStyle>("--coffeeType"));
            Handler = CommandHandler.Create<BrewStyle>(BrewCoffee);
        }

        public void BrewCoffee(BrewStyle coffeeType)
        {
            switch (coffeeType)
            {
                case BrewStyle.FrenchPress:
                    var press = new FrenchPress();
                    press.Brew();
                    press.Press();
                    press.Pour();
                    break;
                case BrewStyle.PodSystem:
                    var pod = new PodSystem();
                    pod.Brew();
                    pod.Pour();
                    break;
            }
        }

        private class FrenchPress
        {
            public FrenchPress()
            {
            }

            internal void Brew()
            {
                Console.WriteLine("Brewing.");
            }

            internal void Press()
            {
                Console.WriteLine("Pressing.");
            }

            internal void Pour()
            {
                Console.WriteLine("Pouring.");
            }
        }

        private class PodSystem
        {
            public PodSystem()
            {

            }

            internal void Brew()
            {
                Console.WriteLine("Brewing.");
            }

            internal void Pour()
            {
                Console.WriteLine("Pouring.");
            }
        }
    }
}
```

Yes, that's a whole bunch of code, and yes, it's a bit of a mess. We have plans to fix that, but for now we're just trying to get our coffee out. Let's unpack our little brewer and look at the manual a bit before we toss it and forget about it.

```c#
public class Brew : System.CommandLine.Command
```

Within the `System.CommandLine` library, the rule of thumb I follow is commands are aligned with classes. This helps keep our code files focused and lean. Although there are plenty of reasons to break this rule, I prefer to start with this idea in mind.

Let's take a look at that constructor:

```c#
public Brew(string name, string description = null) : base(name, description)
{
    Add(new Option<BrewStyle>("--coffeeType"));
    Handler = CommandHandler.Create<BrewStyle>(BrewCoffee);
}
```

Commands are given names, these commands may or may not line up with the class name, so the library allows for you to name them separately. Maybe you have a class called `SuperSecretProprietary` and want to call it with `action`. Well, that's why we accept a name. The description is how the library will supply the using guide to the user should they call the command with --help or -h. We're going to keep wiring that up somewhere else for now.

The first thing we actively do is add an option `--coffeeType`. This is how we allow for the parameter to be called with the command to `brew` coffee. Lastly, we need a handler for our command, and for that we use the factory method Create. The `<BrewStyle>` bit is telling the library what parameters are needed to feed to the method we're marking for execution (`BrewCoffee`).

> Wait, what's that BrewStyle thing?

That is possibly one of my favorite parts of C#, the Enum. Enumerations are a means of aligning actions in your code quickly. It's like a shorthand for different ways of grouping your work. Instead of having a stack of "if" statements, it allows you to flag behaviors around a set of easy to read and think through options. We declare it like so:

```c#
public enum BrewStyle
{
    FrenchPress,
    PodSystem
}
```

This is how we indicate to the application there's a limited number of choices. There are other ways, but this is an extremely readable way to handle this.

```c#
public void BrewCoffee(BrewStyle coffeeType)
{
    switch (coffeeType)
    {
        case BrewStyle.FrenchPress:
            var press = new FrenchPress();
            press.Brew();
            press.Press();
            press.Pour();
            break;
        case BrewStyle.PodSystem:
            var pod = new PodSystem();
            pod.Brew();
            pod.Pour();
            break;
    }
}
```

Now, this method is not going to win any prizes for elegance. But, it's going to get us started. Now, a word about CommandLine's behavior: it is extremely important that the paramter lines up with the name of the variable that's being used locally. What I mean by that is System.CommandLine looks for a parameter in BrewCoffee based off of the option name "--coffeeType" and the typing of the parameter (`BrewStyle`). Make sure these match, otherwise you will do like I do and spend entirely too long wondering why you're not getting your PodSystem brewed.

Additionally, we define two internal classes, FrenchPress and PodSystem, we use the parameter fed in to determine what actions should be taken. In this case, they're just a few methods that will report back to the console that actions have been taken.

With this, we have our first command written as well as our first parameter that can be accepted. But we have no way to actually use them from the command line just yet. Let's open up Program.cs and do a few quick tweaks and see the fruit of our labor.

```c#
namespace coffeemaker
{
    class Program
    {
        static void Main(string[] args)
        {
            var rootCommand = new RootCommand() { new Brewing.Brew("brew") };
            rootCommand.InvokeAsync(args).Wait();
        }
    }
}
```

What's happening here is we initialize the root command, which in this case as we are not naming it or providing a description defaults to the name of the application. Using a handy technique called Collection Initialization, we add a new command, our trusty Brewing.Brew() command, naming it "brew". Lastly, we invoke it, passing in the args and wait for it to finish executing. Using the System.CommandLine vocabulary, the parent command is the root command of the applicaiton, with it's child command of "brew" being able to be called.

If you're using VS Code or a similar editor, it's no doubt shouting at you right now. There's probably red squiggles and highlights right around the Brewing.Brew line, and it looks very unhappy and the program will not build. This is because we've asked it to do something without the right toolboxes on the workbench. Let's fix that by telling Program.cs what toolboxes to look into for the tools it needs:

```c#
using System;
using System.CommandLine;
using Brewing;
```

There we go, all better. Now, the application is less angry and will build.

This is where the dotnet CLI comes back in with a really cool feature. The dotnet CLI allows for you to pass in all of these arguments and commands straight into the run command. Let's take a look at how these things all work together.

# CLI to Victory

```shell
dotnet build
```

There are a few ways to run this command, but for a simple project like ours we only need to be in the root directory of our application where the csproj file is. The build command will look for a csproj, and using that will build the application. If you have multiple projects, you can build just the project you're working with by specifying the project:

```shell
dotnet build coffeemaker.csproj
```

Additionally, sometimes things get stuck. Maybe there's a reference that doesn't quite work correctly, maybe you just need to check that you're building the most current version. For that, you can do a clean:

```shell
dotnet clean
```

This removes all of the built items. If you're coming from a Visual Studio background, the Rebuild option is handled by combining these two commands.

Now, to run the application, oddly enough we will use this:

```shell
dotnet run
```

Right away, the application fails and provides some helpful hints as to why. Right there in red letters is the message that we've forgotten a required command. Down at the bottom of the console output, there's a list of commands that are available to you, let's try that again, using the parameter functions of dotnet run.

```shell
dotnet run -- brew
```

The run command uses the double dash to separate dotnet CLI commands and what is fed into your application. If everything has gone accoring to plan, you were just notified that a tasty batch of FrenchPress is ready to drink. But, as I said, we're in a hurry. There's never enough time in the day and we needed a quick hit of that sweet delicious coffee without waiting for a french press. Let's add our last parameters in and re-run the application.

```shell
dotnet run -- brew --coffeeType PodSystem
```

Now, we're passing the coffee type as a string and selecting that enumerator to feed into the switch statement above. We get a quick report that the two steps needed have been complete and we have our pod system coffee.

# Next Steps

Definitely checkout the System.CommandLine [github](https://github.com/dotnet/command-line-api/blob/main/docs/How-To.md#Add-a-subcommand). I'm going to be using this heavily to begin building out the One Coffee Maker to Rule Them All. Another path we might travel is to create an extension to the .NET core CLI that begins to include some git functionality. At least, that's a topic that is interesting to me.

As always, I would love to hear from you all. Coding is best when collaboritive. Where did this help you? Where could it have been more clear? What kinds of coffee should we include in the final product?

In our next episode, we'll begin by doing some house keeping. Refactoring is an extremely important aspect of writing code and I hope to share some of my process when I do that. How do you keep things tidy? What bits would you move in this application so far to help others get to coding more quickly?
