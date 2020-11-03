# LocalDB

I love working with databases, but I hate having to share them. I'm extremely selfish in that regard. There's an extent to which it may be a character flaw, but here's the deal: I don't want to impact other folks' workflows. That's it. That's the whole deal. A lot of times when I'm working with a database, I'm trying to load data or transform data. I don't like sharing because I can't stand the idea of targeting the wrong table. Trust me, I've done this before, you make a mistake and suddenly a table is just....gone. And you're not getting it back because you didn't add a rollback. And someone is working with pushing test data around in your application and now they're panicked because they just saw that table was out and they're calling the database person, and if you just dropped the table there's a chance that's you. You have a backup? 

There's a handful of ways to get around this, one I'll follow this up with later. it's way more chic and won't have the limitations this option will, but I don't see this option used that often despite how handy it is. There's an option available right out of the box from Visual Studio, even the community option has it. SqlLocalDB is a developer tool that is essentially a stripped down version of SQL Server Express available for development. Why would you use this instead of SQL Server Express? Well, ya got me there. I like using localdb because it's much easier to setup and is inherently locked by Windows to my user account. SQL Local DB is also incredibly easy to setup when compared to other options, and given how much it can accomplish for you, it will probably solve most of your needs for database work locally.

## Installing

Do you have a flavor of Visual Studio installed? Congrats! You probably have it, if not loads the data work work load using the Visual Studio installer. If not, Microsoft has a really solid walk through here: https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/sql-server-express-localdb?view=sql-server-ver15

(It's easiest through the Visual Studio installer, if you use VSCode, just install it that way, you'll get other dev tools that are packaged installation wise with Visual Studio).

## Using

Believe it or don't, all you pretty much need from here on out is you standard SQL tooling and the commands attached to `sqllocaldb`. First, let's see if there are any databases set up.

```sh
sqllocaldb info
```

This spitsout the databases setup during the install of local db. In most cases, there should be an MSSQLLocalDB. To work with this database, we will need to start it:

```sh
sqllocaldb start mssqllocaldb
```

Or if you want to create a new one:

```sh
sqllocaldb create BlogDatabase
```

This will create a new database to connect to call BlogDatabase.

From here, you can navigate and work with this database locally with nearly the same amount of tooling you're used to having. there are plenty of things that Local DB does not have, like the ability to install an SSISDB, but I've found it to handle so much of my day to day, it really is a tool I wish someone tapped me on the shoulder and said "hey, you should know about this" when I was getting started.

## Great...I have SQL but no data

Almost forgot. We're going to use the Northwind database example just to keep things easy. First, download it from their git: https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs

This is another tool I wish someone, ANYONE would have told me existed when I was learning anything related to SQL. I needed something to practice with desperately, learning SQL in production is not a good thing to do, but I couldn't figure out where to find or get this down, so I'm going to spend a little extra time here. If you're familiar with git, feel free to skip ahead.

We're going to use Azure Data Studio, so point an internet machine here and get it:
https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver15 

This is to SQL Server Management Studio what Visual Studio Code is to Visual Studio. It's lighter, has a different feature set, but has most of everything you'll want on the day to day. (It's also available on Mac, which is a huge plus for me because it means less context switching).

Once that's installed, you'll want to use Windows Authentication to connect to the database, and your connection string will be (localdb)\<database to connect to>, or from the examples above, the below examples:

```
(localdb)\MSSQLLocalDB
(local)\BlogDataBase
```

You'll open this up by selecting "New Connection" in the start page of Data Studio. 

![Sql LOCAL Db connection string entry](../images/SQL%20Local%20DB/Connection%20String%20Entry.png)

Since it is connected by default to your Windows credentials, you will need to set "Windows Authentication". 

Now that we're in, pull the Northwind files and let's walk through how to use this setup to setup the Northwind database. First, fork the repo into you're git (so you're not working out of their branches). Next, clone them locally and open the `instnwnd.sql` and `instpub.sql` files in Azure data studio. They should automatically connect to the database you just connected to. After you have them open, all you need to do is run them, and you will have a functional local db loaded with the Northwind databases.

## Well...that's great and all...

But yeah, this probably isn't what you have lying around. You probably do, however, either have or have the capability of having a .bak to restore a database from. If not, and with your DBA's permission, here's how you can get one. Log into that server with Azure Data studio. We're going to stick with the Northwind as it's what we have right now.

Under the server explorer tab for this database instance, locate the Northwind (or other) database you want a backup to restore from. Click the ellipses and select "Backup" from the drop down menu.

![Sql Backup menu option](../../images/SQL%20Local%20DB/Backup%20Options.png)

From there, you will need to select where you want to store your backup. Once that's done, return to the local database you'll be working in, in our case it's going to be the BlogDataBase we created.

![Restore from Backup](../images/SQL%20Local%20DB/Restore%20from%20Backup.png)

In the topmost dropdown, select "Backup File" from the "Restore From" dropdown. Select the "Backup File path" as the backup you just made. The rest should fill out itself from those backup files. When I did this, I needed to go to the "Options" tab and select "Overwrite the existing database (WITH REPLACE)" to manage the restore. And now, congratulations, you have a local database that you can run riot in and not really have any major risks associated with. You have isolation from your fellow testers and have the capability to just restore from a good backups in the event that things go completely sideways. 