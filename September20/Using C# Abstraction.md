# Abstraction

One of the things that is imminently useful within software development, and by no means exclusive to .NET, is the ability to have an abstraction of something. For example, coffee. (If you're following along with my other blogs here, you might notice a disturbing trend now.) Famous companies named after first mates and dipping pastries all peddle this fantastic beverage. You may drink it sweetened, but I prefer it straight. All of these shops craft their own unique drinks, but they are all known as coffee. That's because while a caramel machiato is functionally different from a double americano, they both somehow fall into this grander category called "coffee".

In software development, we call it abstraction. In object oriented programming, we treat what we interact with, design, and store data in our code as objects. However, this short hand has an additional analogue to the real world. In .NET there are two different types of abstraction. In ascending order of abstract-ness (it's totally a word. But don't look it up.), you have concrete classes, abstract classes, and interfaces.

In what is commonly referred to as a concrete class, the details of what the thing is are inherently tied to the thing itself. Using your phone as an example, imagine a world in which every phone had the same exact apps, same exact code, same exact functionality. It'd be kinda boring, but at least we'd all rest easy that the dang thing is pretty easily replaced.

Now, an abstract class is a collection of ideas and actions that something can do, but they're over written by the concrete representation if needed. Using cell phones again, you could have the cell phone object that sends messages using SMS, one that overrides the send feature with a fancy thing called IMessages (blue bubbles unite!), and one that overrides that functionality with something just called Messages (which is technically a better implementation, but does anyone use it?). With this sort of abstraction, the messaging application can still use the default option in combination with it's specific flavor as well. Something like this:

```c#
public abstract class CellPhone
{
    public virtual void SendMessage()
    {
        Console.WriteLine("SMS Sent");
    }
}

public class FruitPhone : CellPhone
{
    public override void SendMessage()
    {
        Console.WriteLine("Fancy message sent with a blue bubble, how fancy.");
        base.SendMessage();
    }
}

public class RobotPhone : CellPhone
{
    public override void SendMessage()
    {
        Console.WriteLine("Sent RCS Message with awesome features");
        base.SendMessage();
    }
}
```

Essentially, what we've done here is have three different devices handle the same functionality, but differently with a fallback to a standardized default protocol. This is very handy because as long as something is expecting a cellphone, you can send it a FruitPhone or a RobotPhone and it will call that SendMessage() method. So, realistically if you have a method that sends messages and it receives the CellPhone abstraction like so:

```c#
SendMessage(CellPhone phone)
{
    phone.SendMessage();
}
```

It will take either a FruitPhone or a RobotPhone and send the message. The phone is the only thing that needs to know how to send the message. Think of this level of abstraction as a means of defining your object while needing to ensure that certain, core functionality is shared across all possible implementations.

Now we're up to my favorite, interfaces. Interfaces are super handy, but they require a little more work. They're a level of abstract that requires anything that inherits it to implement everything. As a practice in .NET, they are prefixed with an `i` to help spot them in your code. Let's take our last project of coffee making and start to simplify some parts of it.

## Why Abstraction

Abstraction makes your life easier, specifically because of how we created the method that brewed our coffee. One of the interesting things about abstraction is that it serves to reduce the amount of information you need to know about something in order to use it. Look at how our switch operates from the last blog:

```c#
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
```

I'm cringing at this, and I wrote it! If we needed to add a new coffee type, we're going to have to make changes here to handle that. Additionally, if we need to add a step to the PodSystem, we're going to need to change it. Aye-ya-yaye. In software development, this is called "coupling", and it's a no-no. If at all possible, keep this from happening in your code because chasing these things down long term will be difficult, time consuming, and confusing. I'm willing to accept difficulty, but confusion and time consumption is definitely not something I would wish on someone else, so let's tidy up my mistake.

> Short hand of "knowledge"
> I forget where I picked this up, but the way I like to think about the usefulness of abstraction comes down to "knowledge". The more one piece of code needs to know about how another piece of code operates, the more dependent or coupled they are. This coupling increases complexity and changing things becomes more difficult as systems grow. I will keep using this terminology, because it's been very helpful to me in thinking through how to work on these sorts of problems.

First, if you're following along, go ahead and navigate to your Brew folder and make a new file called Coffee.cs.

```shell
cd Brew
touch Coffee.cs
cd ..
code .
```

Our goal at this moment is to move the nested classes out from the brewing class. Thinking in terms of what different aspect of the application should know, we want to setup the brew to be agnostic: able to handle a french press, a podsystem, or even be able to take in a brand new coffee we want to make without needing to worry about updating the brewer. To start, let's open up our Coffee

First, let's move our brewstyle enumeration into this file. And, let's add another one.

```c#
public enum BrewStyle
{
    FrenchPress, PodSystem, FrisbeeBrew
}

public enum Granularity
{
    Fine = 0, KindaFine = 1, Medium = 3, KindaCoarse = 4, Coarse = 5, Chunky = 6, WholeBean = 7
}
```

If we keep relying on concretely represented classes, adding a FrisbeeBrew will be sort of the last straw. So, let's write up a representation of all things coffee, and for this we're going to use an interface.

```c#
public interface iCoffee
{
    int Temperature { get; set; }
    BrewStyle Style { get; }
    Granularity GrindGranularity { get; set; }
    iCoffee Brew();
    iCoffee Pour();
}
```

With interfaces, we simply define what properties the class will have and what things the application can do. In this case, all of our coffees will have a set of instructions to `Brew()` and `Pour()`. By convention, interfaces get an `i` in front of their name. Additionaly, we've give the property of `Temperature`.

Now, let's refactor our French Press.

```c#
public class FrenchPress : iCoffee
{
    public int Temperature { get; set; } = 70;
    public BrewStyle Style => BrewStyle.FrenchPress;
    public Granularity GrindGranularity { get; set; } = Granularity.WholeBean;
    public iCoffee Brew()
    {
        BoilWater();
        GrindBeans();
        WaitAMinute();
        return this;
    }

    private void WaitAMinute()
    {
        for (int i = 0; i < 100; i++)
            Write($"\r{GetChar(i)}  ");
        Console.WriteLine();
    }

    private char GetChar(int position) =>
        (position % 4) switch
        {
            0 => '|',
            1 => '/',
            2 => '-',
            3 => '\\'
        };

    private void GrindBeans()
    {
        Console.WriteLine("Grinding beans.");
        while (GrindGranularity != Granularity.KindaCoarse)
            Write($"\rCurrent granularity: {--GrindGranularity}.");

        Console.WriteLine();
    }
    private void BoilWater()
    {
        Console.WriteLine("Boiling water.");
        while (Temperature < 200)
            Write($"\rCurrent temperature is {++Temperature}°F.");
        Console.WriteLine();
    }

    private void Write(string line)
    {
        Console.Write(line);
        System.Threading.Thread.Sleep(100);
    }

    public iCoffee Pour()
    {
        Console.WriteLine("Pouring.");
        Console.WriteLine("Enjoy!");
        return this;
    }
}
```

There's a few things to notice here. First, the class definition has a `:` attached to it. This means several things in C#, this one in particular means "implements". By and large, that `:` bit means "inherits", however with an interface, there's nothing to inherit. With C#, usually we use inheritance as a means of taking predefined functionality and applying it to another class, or in the case of abstraction taking predefined functionality and having the option to override it. When inheriting from an interface, there's nothing to inherit other than these general descriptions, so it's more common to refer to this as "implementing".

The method `Brew()` has been implemented to call several internal methods, as has `Pour()`. Remember the earlier note about what things "know"? Well, with this methodology, everything labelled "public" is determined to be things that are needed to be known by what is using our code. In this case, the interface says that things will need to know the coffee's temperature and how to `Brew` and `Pour` the coffee. Everything else? None of the external code's business. We're having those accessible only privately, internal to the coffee.

Let's add into this file the other two types of coffee and take a look at how we can use this technique of interfaces to make our brewing system cleaner:

```c#
public class PodSystem : iCoffee
{
    public int Temperature { get; set; }
    public BrewStyle Style => BrewStyle.PodSystem;
    public Granularity GrindGranularity { get; set; } = Granularity.KindaFine;
    public iCoffee Brew()
    {
        InsertPod();
        PushButton();
        return this;
    }

    private void InsertPod()
    {
        Console.WriteLine("Inserting pod.");
    }

    private void PushButton()
    {
        Console.WriteLine("Button pressed.");
        Console.WriteLine("Waiting for coffee.");
        Console.WriteLine("Done.");
    }

    public iCoffee Pour()
    {
        return this;
    }
}

public class FrisbeeBrew : iCoffee
{
    public int Temperature { get; set; } = 70;
    public BrewStyle Style => BrewStyle.FrisbeeBrew;
    public Granularity GrindGranularity { get; set; } = Granularity.WholeBean;
    public iCoffee Brew()
    {
        BoilWater();
        GrindBeans();
        Console.WriteLine("Stirring.");
        Stir();
        WaitAMinute();
        return this;
    }

    private void Write(string line)
    {
        Console.Write(line);
        System.Threading.Thread.Sleep(100);
    }
    private void BoilWater()
    {
        Console.WriteLine("Boiling water.");
        while (Temperature < 200)
            Write($"\rCurrent temperature is {++Temperature}°F.");
        Console.WriteLine();
    }
    private void GrindBeans()
    {
        Console.WriteLine("Grinding beans.");

        while (GrindGranularity != Granularity.Fine)
            Write($"\rCurrent granularity: {--GrindGranularity}.".PadRight(50));

        Console.WriteLine();
    }

    private void Stir()
    {
        for (int i = 0; i < 50000; i++)
            Console.Write($"\r{GetChar(i)}  ");

        Console.WriteLine();
    }

    private void WaitAMinute()
    {
        for (int i = 0; i < 100; i++)
            Write($"\r{GetChar(i)}  ");

        Console.WriteLine();
    }

    private char GetChar(int position) =>
        (position % 4) switch
        {
            0 => '|',
            1 => '/',
            2 => '-',
            3 => '\\'
        };
    public iCoffee Pour()
    {
        Console.WriteLine("Pouring.");
        return this;
    }
}
```

Now, before you judge, we'll get to some of this copy and pasted code a little later. For now, the important thing is that we now have three different classes implementing the same interface, but doing different things.

Now, how on earth are we going to use it? My first gut instinct with interfaces was to just new up the concrete instances and assign them to a variable that's using the interface like so:

```c#
iCoffee coffee = new FrenchPress();
```

Which, is perfectly valid and certainly not a _bad_ thing to do. And there are more than a few places where this would be what you would do, but for our purposes, we're going to be using a factory pattern. This will allow for us to generate what we need, when we need it.

```sh
cd Brew
touch CoffeeFactory.cs
cd ..
```

Open up CoffeFactory.cs and add the following:

```c#
using System;
using System.Collections.Generic;
using System.Linq;

namespace Brewing
{
    public class CoffeeFactory
    {
        public static List<iCoffee> coffees = new List<iCoffee>();
        internal static iCoffee Create(BrewStyle coffeeType)
        {
            if (!coffees.Exists(coffee => coffee.Style == coffeeType))
                coffees.Add(CoffeeSwitch(coffeeType));

            return coffees.First(coffee => coffee.Style == coffeeType);
        }

        internal static iCoffee CoffeeSwitch(BrewStyle coffeeType) =>
            coffeeType switch
            {
                BrewStyle.FrenchPress => new FrenchPress(),
                BrewStyle.PodSystem => new PodSystem(),
                BrewStyle.FrisbeeBrew => new FrisbeeBrew()
            };
    }
}
```

We're creating a static factory that takes in what BrewStyle we need. In the interest of simplicity, we ensure that there's only one coffee of each type being being given to our coffee brewer by using a list of coffees. We first check to see if there are any coffees in existence that have the needed BrewStyle. If there isn't one, we ask the CoffeeSwitch to give us a new one, add it to the list, then return the correct coffee by BrewStyle. Nice and clean. And, if we need to add more coffees, we have one place that we need to add it as an option.

Now, we need to go back to Brew.cs and do some more work. Some of my personal favorite work. Deleting as much as possible.

```c#
public class Brew : System.CommandLine.Command
{
    public Brew(string name, string description = null) : base(name, description)
    {
        Add(new Option<BrewStyle>("--coffeeType"));
        Handler = CommandHandler.Create<BrewStyle>(BrewCoffee);
    }

    public void BrewCoffee(BrewStyle coffeeType)
    {
        var coffee = CoffeeFactory.Create(coffeeType);
        coffee.Brew();
        coffee.Pour();
    }
}
```

Look at that. Now, we can add as many different types of coffee as we want and the brewing mechanism won't change. It won't have to because it doesn't know how to brew the coffee, that knowledge is kept in the coffee class. It also doesn't even need to know what type of coffee it is, our switch is now gone. The BrewCoffee method takes the data from the CLI implementation, passes it agnostically to the BrewCoffee() method, and without knowing anything about French Press, PodSystem, or FrisbeeBrewing makes every variation.

Let's do a quick check on our Program.cs:

```c#
static void Main(string[] args)
{
    var rootCommand = new RootCommand() { new Brewing.Brew("brew") };
    rootCommand.InvokeAsync(args).Wait();
}
```

Let's test it out, now. With this setup, you can now use the dotnet CLI to test the application.

```sh
dotnet run -- brew --coffeeType FrenchPress
```

This brews a french press. Let's try another coffeetype:

```sh
dotnet run -- brew --coffeType PodSystem
```

And lastly, let's try the new coffee: Frisbee Brew!

```sh
dotnet run -- brew --coffeeType FrisbeeBrew
```

## Cool, coffee?

Yes, coffee and factories. If you're like me, seeing these pieces all written up and together helps. Help yourself to the example over at [github](https://github.com/zackimball/coffeemaker/). Feel free to fork it and clone locally if you want to tinker and extend this.
