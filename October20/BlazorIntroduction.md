# The Blazor Bullet Journal

Blazor is the new cool framework provided by the .NET foundation. It supports a few different options, both a server side option and a SPA styled option that allows for you to write C# for the client side of your application. As with any framework, it brings a host of new toys to developers. 

Another, far lower technology I use frequently is a Bullet Journal. Seriously, it's about the only system that's ever worked for me. A lot of that comes from the fact that it works the way I took notes before, but has many "extension" methods that help clarify and maintain how I can best be productive. I've tried dozens of to do apps, but nothing's really stuck. So, in keeping with developer tradition, I'm building a better mousetrap. And there's thousands of to do apps, so why not just use my own?

## Some Notes

I'm going to be tracking this at github and versioning them to match blogs. That way, you can catch up and see what's changed between versions. This also isn't exactly a tutorial series, mistakes will be made. Essentially, the goal is to document how the process goes over time. No doubt along the way I'll be trying new things out. Also, as things go wrong, feel free to let me know what you'd do differently in the comments section.

### Stack

We're going to split this up into three layers. The database is going to be Mongo, since I've never used it and it seems really uncommon with .NET and I'm curious as to why that is. Additionally, I'm going to be using Docker containers because that's the
