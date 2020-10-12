# Why do this?

So, one of the cool things about software is that there's normally more than one right way to solve a problem. With so many different ways of thinking about problems and so many different solutions, there's always something new around the corner. To start with, let's leave the project we have been working on behind and start expanding that code into a more robust coffee maker.

```sh
cp -r coffeemaker coffeemakerv2
cd coffeemakerv2
rm -rf .git
```

If you're using PowerShell, this will be a little different, but in shell systems this will copy, recusively, all of the files from our coffeemaker to coffeemakerv2. From there, we're switching to that new directory and clearing out the git repo, resetting it.

## Dependency Injection

What our last topic did was decrease the complexity of our code by reducing the amount of knowledge different parts of the application needed in order to operate. One of the many reasons that I try to do this is to move control back into the hands of a central location. If you know where everything is coming from, you'll know where to go to change them. Of course, there's an extent to which this is probably overkill for a small console app like we have here. However, let me tell you how many times this has saved me time...

![Approximately 10 Hours Later](../images/More%20System.Commandline/Approximately%2010%20Hours%20Later.jpg)

...and that is why you never deploy code from a boat. True story.

Anyway, the idea of dependency injection is that you create a container in which you place objects to pass between classes. There's a host of different tools to pass things between classes in .NET, but DI is easily my favorite because it keeps knowledge where it belongs and adds a bit of magic to your code, and who couldn't use more of that.

> The group at Dotnet has been doing great work in their documentation and trainings, this is going to be based on their example over at their [Github Page](https://github.com/dotnet/command-line-api/tree/main/samples/HostingPlayground).

In our previous entry, we did some cleaning up that made our lives a little easier should we extend things. There's actually another step that we can take, and let's grab a few packages that we're going to need.

```sh
dotnet add package Microsoft.Extensions.Logging --version 3.1.8
dotnet add package Microsoft.Extensions.Hosting --version 3.1.8
```
