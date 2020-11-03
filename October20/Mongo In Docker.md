# Why? Why would you do this?

For an upcoming project that will be coming to my blog, I really want to use MongoDB. As a .NET developer, you can imagine that the first tool I reach for nine times out of ten will be SQL server for persistence. It's sort of the go-to for .NET. But there's so many other databases out there, and I've always wanted to try to learn a bit more about NoSQL databases, specifically Mongo. So, that's what we're going with.

But why a container? Well, as many of you have, I've routinely looked over the fence at what other developers do and when it comes to databases, they're just a hastle to work with. Period. I mean, what if I'm tweaking a table while someone needs it? Or, god forbid, what if I drop a table or accidentally update records and forget my rollback? I don't really want to hope the backups are good. Seriously, I've dropped a table before and that's just a mess. Even if your backups are good, it's a huge pain in the butt. Enter containers.

### What's a container?

Well, let's start with an application on a Virtual Machine. If you have an API posted up on a virtual machine, the "whole package" is the API you're running and it's software, the whole operating system (the "Guest" operating system), the Hypervisor the guest operating system uses to talk to the Host operating system, and the underlying hardware. If you're familiar with Virtual Box, for example, you could run a Node.js application hosted on a very skinny Arch image hosted on the Virtual box hypervisor that's hosted on an Ubuntu server. It's all a bit much, to be frank, but there are a lot of benefits to be derived by doing this. One of which is you can, theoretically, manage to have multiple instances of the API running and use a load balancer to feed different VMs, and if you're like me you're now thinking about that and cringing because holy crap that's a lot of work.

Containers, at a very superficial level, attain many of the benefits of VMs with far, far fewer resources dedicated to them, and they're a lot easier to maintain. The magic, essentially, is that instead of a full VM, the container image shares the resources of the guest operating system across smaller, isolated packages called containers that are running your application. 

This allows for the enhanced flexibility and consistency of environment without managing and maintained a much more complicated hypervisor. This also means that if you're using a service that will let you run containers (like Azure, AWS, or the like), the environment you're building in, testing in, breaking things in, and generally bringing havoc to locally will match what is in production.

With containerization, there's a lot of promises, but persistence is absolutely not one of them. Containers should be considered throw away items. This does not work for data you need to keep. At least, not in production. This is why you hear everyone scream that database containers are for development only. This disposability is exactly what I want when I'm testing against a database, though. I want to set it up and tear it down completely. Additionally, Docker does support mapping local storage to container storage, so, for development, this is more than sufficient for what we need to do. So that's what we're going to do: persist the data locally for now by mapping a local folder to be read inside the container, and now we will have a shiny new Mongo database to put through it's paces in isolation, and if someone else was working on the database at the same time, who cares? They're on their machine, I'm on mine, I can drop all the collections I want and run as expensive of a query as I want while testing. 

Additionally, there are GUID based options for everything we're about to do, but frankly I've been pushing hard to use the command line as much as possible recently and I have totally come to feel like it's the easiest way to do things these days.

## Installing Docker

I'm not going to get into details on this one, mostly because I set this up on a Mac, and the way you set Docker up is very system dependant. I will be cheating this section horribly by providing you a link for Docker's docs on the subject. Especially for Windows folks, you have to make a judgment call whether or not you want to use WSL or instead use the Windows Docker. I recommend using WSL, but to each their own (that's what containerization is all about: flexibility).

Docker setup link:
https://docs.docker.com/get-docker/

I also will be using Docker Compose, which simplifies a lot of the configuration of your containers. Install is here:
https://docs.docker.com/compose/install/

Go ahead and tuck into all of that, it's some good reading and this will be here when you're done.

## Docker Basics

### Getting Prepped

First, we're going to need to pull down a fresh mongo image:

```
docker image pull mongo
```

This tells the CLI to pull the latest and greatest mongo DB image. You'll see a bunch of things pulled down, and that might take a moment depnding on your internet connection. Congrats, you've got a shiny new image to prep.

Create the directory you'll be working, for us, mongodockerdemo because I'm super creative. Once you're inside that directory, add a subdirectory for mongo-volume, a docker-compose file, and an initialization script.

```
mkdir mongo-volume
touch docker-compose.yml
touch init-mongo.js
```

What? Surely, I'm kidding, a bad joke. 'init-mongo.js'? But it's a database? 

Correct, one that was written to be extremely, painstakingly Javascript friendly. *Cue A Whole New World*.

Yes, friends, it's time to slip the surly bonds of SQL and Tables and dance the skies of object oriented databases. There's no way this ends badly for us. (I will find a way, I assure you.)

Pop open the docker-compose file and let's crack on.

## Docker-Compose

First things first, if you're new to yaml, whitespaces are important. Spaces, trailing or leading, tabs, all of those annoying things that you can't readily spot in a text editor matter. Because of that, there will be a complete version of this that you can download from my github. Chances are instead of missing a semi colon you have an additional space, and...it's gonna be a while before you find it. Not autobiographical.

Docker compose is built around having your workflow mocked up in a single config that will start up multiple containers for you. We'll only be using one, for now.

Up at the top of your `docker-compose.yml`, specify the version of compose file we're working with today:

```yaml
version: '3.1'
```

Under that line, we will start to define our containers as services:

```yaml
services:
    mongodb:
        image: 'mongo'
```

Our first service, which will be named mongodb in compose, will use the mongo image (which we went ahead and pulled down earlier).

```yaml
        container_name: 'DemoData'
```
Because, again, being super creative, we're naming the container that will be built "DemoData". 

Now, with Mongo, there's some important environment setup items that we will need to tell it in order for it to setup. One is the init database, then the root username/password. God forbid you push this to a production instance, at least change these values.

```yaml
        environment:
            MONGO_INITDB_DATABASE: admin
            MONGO_INITDB_ROOT_USERNAME: someSuperUserName
            MONGO_INITDB_ROOT_PASSWORD: someSuperAwesomePassword
```

This next part is the important part, it's what will allow for us to push data back and forth to the MongoDB and pick up where we left off next time. Usually, you would need to type a fairly long command into the command line to handle this, but with Docker Compose, it's three more lines of configuration:

```yaml
        volumes:
            - ./init-mongo.js:/docker-entrypoint-initdb.d/mongo-init.js
            - ./mongo-volume:/data/db
        ports:
            - '27020:27017'
```
The mapping syntax uses a [local resource] : [container resource] syntax. So, what we've done here is map two things to be copied into the container as it's brought online. The first one is the init-mongo.js script. That's going to be pushed into the location the mongo db looks for it's initialization script.  The second volume we're operating with is creating a bridge between our development machine and the container in the form of mapping a local folder to the directory in the container that MongoDB will be writing all of it's database files. Lastly, we're mapping the default ports on the container inside of the database to the port we'll access it from externally.

All together for the copy and paste folks:
```yaml
version: '3.1'
services:
    mongodb:
        image: 'mongo'
        container_name: 'DemoData'
        environment:
            MONGO_INITDB_DATABASE: admin
            MONGO_INITDB_ROOT_USERNAME: someSuperUserName
            MONGO_INITDB_ROOT_PASSWORD: someSuperAwesomePassword
        volumes:
            - ./init-mongo.js:/docker-entrypoint-initdb.d/mongo-init.js
            - ./mongo-volume:/data/db
        ports:
            - '27020:27017'
```

## Mongo Initialization Script

This will only run the first time you're initializing your container. 

First, let's create a user so that we're not using the root user all of the time:

```js
db.createUser(
    {
        user: "NotRootUser",
        pwd: "NotRootPass",
        roles:[
            {
                role:"readWrite",
                db:"admin"
            },
            {
                role:"readWrite",
                db:"NotAdminData"
            }
        ]
    }
)
```
This will create a user "NotRootUser" with the associated password, and give them read/write on the admin database as well as the NotAdminData database.

Additionally, let's create an applicaiton user account for our future application:

```js
db.createUser(
    {
        user: "ApplicationUser",
        pwd: "SuperDuperLongPassword",
        roles: [{ role: "readWrite", db:"NotAdminData"}]
    }
)
```

And let's add a basic database and collection for later while we're at it:

```js
var NotAdminData = db.getSiblingDB('NotAdminData')
NotAdminData.createCollection('NotAdminCollection')
```

And of course, if you prefer to copy and paste instead of just pulling the repo, I've got you here as well:

```js
db.createUser(
    {
        user: "NotRootUser",
        pwd:"NotRootPass",
        roles:[
            {
                role:"readWrite",
                db:"admin"
            },
            {
                role:"readWrite",
                db:"NotAdminData"
            }
        ]
    }
)

db.createUser(
    {
        user: "ApplicationUser",
        pwd: "SuperDuperLongPassword",
        roles: [{ role: "readWrite", db:"NotAdminData"}]
    }
)

var NotAdminData = db.getSiblingDB('NotAdminData')
NotAdminData.createCollection('NotAdminCollection')
```

## Starting your container

Now, it's time to summon your database:
```bash
docker-compose up
```
This will tell the docker-compose to look for that yaml file, read the config, and build you a container that follows those specifications. Shiny, right?

## Comment Section Sound Off

All right, fair's fair, this was all about MongoDB, but there is a SQL server container available from Microsoft for Docker. Is there anyone out there who would like to see someone struggle bus through setting one up? Maybe that's a more important use case? Usually I want feedback for next steps on projects I work on here, but this one already has a purpose and will be used for that.