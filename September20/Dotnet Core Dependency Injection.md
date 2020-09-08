# What is DI?

Dependency Injection is a practice in software development that causes magic and sparks joy. Essentially, dependency injection is a means to decrease your code's dependency on libraries by creating an abstraction that can be interacted with.

That was a lot of buzzwords, and while fun, buzzwords tend to obscure what on earth you're talking about. Boiling it all down, what dependency injection attempts to do is take a series of tasks done by one object and repeat it between classes. By taking the behavior of one object and making it generic, you can then rely on a sketch of what that object does and without changing anything in one class, change the behavior of what the object does. It gives your code super powers.

Which brings us to coffee. With apologies to tea drinkers, coffee is a magical beverage that can be made any number of ways, have all kinds of different flavors, and be enjoyed in different forms. However, I've not come across a coffee maker that does all of the things that I want it to do. I'm a huge coffee junky, it's probably unhealthy the number of different ways I've learned to make it. Pod machines, brewers, stove top espresso, this frisbee company's one offering in a tube, and so on. They're all exciting and different, taste different, but still are coffee makers. IRL, you can't just combine all of these and have the One Machine to Rule Them All, much to the dislike of anyone who has ever lived with me. But, with a slick trick called dependency injection, an application can. Much like in my kitchen, in code space is a premium and repeating yourself makes things awkward.

## Getting Started

Do you know how to setup a project in dotnet? Feel free to skip over this. There might be something interesting, but if you're like me you may have come across this needing something in a hurry, so just go right on through to the next section.

Still here? Cool. There's a boatload of different ways to handle this, but we're going to keep with the theme of my last posting and start with the command line. Make the directory you need and add a new project.

```
mkdir coffeemaker
cd coffeemaker
git init coffeemaker
```

This makes your directory, switches to it, and initializes a git repo. That last one is good for me, because I lose things quickly, but is entirely optional. Now we have a clean, empty space that reeks of sweet delicious minimalism. Let's put a stop to that.

```
dotnet new gitignore
dotnet new console
```

First, we add in a gitignore, complements of dotnet's CLI. This is essential because the folks at the .NET foundation already know there's a whole bunch of things that are going to be built that you won't want in source control. Then, we want to create a new console appliction. We're keeping this dead simple for now.

With that, we've got this handy template loaded and are ready to start adding the things we will need to build the One Coffee Maker to Rule Them All.

```
dotnet add package Microsoft.Extensions.DependencyInjection
dotnet add package System.CommandLine
```

This uses dotnet's built in package manager, NuGet, to get a few things we're going to need. The first is the dependency injection tool and the second is the command line parsing tool made by the devs for .NET. There's a whole bunch of these available, and they all have their perks, but we're going to stick with what's built in for now as it's used in a lot of .NET's templates (like the ASP/ASP.MVC/Blazor templates. Stay tuned!)

With that out of the way, this should be what your current tree looks like:

```
.
├── Program.cs
├── README.md
├── coffeemaker.csproj
└── obj
    ├── coffeemaker.csproj.nuget.dgspec.json
    ├── coffeemaker.csproj.nuget.g.props
    ├── coffeemaker.csproj.nuget.g.targets
    ├── project.assets.json
    └── project.nuget.cache
```

> Note: We're going to be using VS Code to work through this example, but you can use anyy editor you like. Things may work a little differently, though.

# Where's the super powers?

Hold up, wait a minute. Let's define an object first. Let's keep things tidy and make a new directory to keep all of our coffee types in.

```
mkdir coffee
```

And add our first brew type.

```
touch coffee/coffee.cs
code .
```

And now we have the application roughed out. Open up coffee.cs and make your first class of coffeee:

```c#
using System;

namespace Coffee
{
    public class Cuppa
    {
        public float Temperature { get; set; }
        public bool Brewed { get; set; } = false;
        public bool Ground { get; set; } = false;
        public bool Poured { get; set; } = false;
        public int AmountOfCoffeeFlOz { get; set; } = 8;

        public Cuppa()
        {

        }

        public void Grind()
        {
            Console.WriteLine("Grinding coffee");
            Console.WriteLine("Coffee grinder goes whirrr");
            Ground = true;
            Console.WriteLine("Coffee is ground.");
        }
        public void Brew()
        {
            Console.WriteLine("Brewing coffee");
            Brewed = true;
            Console.WriteLine("Coffee brewed.");
        }

        public void Pour()
        {
            Console.WriteLine("Coffee pouring.");
            Poured = true;
            Console.WriteLine("Coffee is poured");
        }
    }
}
```

And make a new coffee maker:

```
mkdir maker
touch maker/coffeemaker.cs
```

And in your coffeemaker.cs file, make your first coffee maker:

```c#
using System;
using Coffee;

namespace CoffeeMaker
{
    public class CoffeeMake
    {
        public Cuppa MakeCoffee()
        {
            var cup = new Cuppa();
            cup.Grind();
            cup.Brew();
            cup.Pour();

            return cup;
        }
    }
}
```

And in your Program.cs, make some coffee!

```c#
using System;
using CoffeeMaker;

namespace coffeemaker
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Making the coffee.");
            var maker = new CoffeeMake();
            maker.MakeCoffee();
        }
    }
}
```

Congratulations, you've got a coffee maker! But, what happens if you want to have an espresso maker. You could make a brand new class called Espresso and build it exactly like how we just built the Coffee.cs, however that doesn't keep the kitchen counter tidy (and that's a premium for this example.)

Pausing here, let's think about what your Program has to know to work right now. If we add a new type of coffee without abstraction, it will need to know more about that coffee than perhaps we want to. We'd have to add some complexity to handle that. In this case, if we wanted to make espresso or a french press, the program itself will need to be aware of this as well as the difference between the two. If we feed a parameter into the args, we will need to read it, determine what kind of coffee to make, what coffee maker to use, etc. Every time we add a new coffee, we'll need to manually update the main program file. This is tedious after a while, after all there are so many types of coffee out there. What if we could just pass that information in directly?
