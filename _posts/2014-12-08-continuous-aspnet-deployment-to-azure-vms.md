---
layout: post
title: "Continuous ASP.NET deployment to Azure VMs"
description: ""
category: 
tags: dotnet, azure, windows, microsoft, deployment
---
{% include JB/setup %}

<small>In a hurry? Jump to <a href="#solution">my solution</a>.</small>

## Environment

We're working on transitioning from a per-customer client/server environment, 
where each customer has a copy of SQL Server functioning as the server and all 
other machines connect to it as clients, to a modern web-based client/server 
setup. We'll still have some Windows apps here and there, but the bulk of our 
functionality will run in a browser.

With the current software being a messy VB6 application using a third-party 
Visual Source Safe clone for source control, we don't use any modern software 
development things with it, such as unit tests or continuous integration.

Moving to the modern .NET stack is allowing us to easily use these tools, and 
starting a fresh code base allows us to more easily write tests.

We are very much a Microsoft shop. At least at this point in our development, 
we're working on Windows installs, inside Visual Studio, writing C#, committing 
our code to [Visual Studio Online](http://visualstudio.com), and using VMs and 
hosted SQL Server on [Azure](http://windowsazure.com). The only piece of our 
stack that isn't as MS as it can be is source control; we're using 
[git](http://www.visualstudio.com/en-us/get-started/share-your-code-in-git-vs.aspx) 
instead of Team Foundation Server. (That said, I believe I read that Microsoft 
had a custom git server, and it's hosted by them and fully supported by Visual 
Studio.)

## Continuous Integration

Setting up continuous integration in this environment was trivial. It takes 
[a few clicks](http://www.visualstudio.com/en-us/get-started/build-your-apps-vs.aspx) 
inside Visual Studio to get set up.

Once that's configured, every push will cause a build controller to grab a 
fresh copy of your code and build it. I even set up 
[notifications](http://msdn.microsoft.com/en-us/library/ms181725.aspx) that 
email my entire team if one of us breaks a build.

## Continuous Deployment

After building our basic project structure and setting up a CI build, my next 
task was to get our build to automatically deploy to a development server. This 
way, every time we commit new code, the development site is live. At any given 
moment, it may or may not function properly, but our bosses could easily check 
out new features as we finish them without having to tie up our workstations or 
wait until we have time to deploy it on a server.

This is where I ran into trouble.

I spent countless hours trying to get my CI builds to deploy. Our commit log (I 
didn't think to do this on a throwaway project) is filled with commits simply 
saying "test" where I changed the content of <kbd>Index.cshtml</kbd> to see if 
a deploy worked. Each month, VSO accounts get 60 minutes of build time; I hit 
that limit multiple times and wasn't able to continue trying new builds. I lost 
track of the number of searches I made to try to figure out how to make this 
happen.

Almost every tutorial or [Stack Overflow](http://stackoverflow.com) question I 
found differed from our environment in one of two ways:

1. They were deploying to an on-site IIS server, which may allow them to use [UNC](https://en.wikipedia.org/wiki/Path_(computing)#Uniform_Naming_Convention) paths instead of [Web Deploy](www.iis.net/downloads/microsoft/web-deploy) to deploy the application
2. They were deploying to Azure websites instead of Azure VMs

In theory, every Web Deploy tutorial I found should have worked, but they never 
did.

## Solution

My suggested setup for continuous deployments in our environment has two bits, 
one for setting up the server and one for setting up the build. You should be 
able to configure Web Deploy on an existing IIS install and have everything 
work, but since my build definition wasn't quite right I'm not sure if my 
previous instructions for WebDeploy were right or not. If you can get away with 
creating a new VM, it's easier to let Visual Studio set it up for you anyway.

These instructions are valid today (8 December 2014) using Visual Studio 2013. 
Newer versions of VS and changes to Visual Studio Online or Azure may require 
modifications.

### Development Server

1. In Visual Studio, right click on your web project and choose _Publish..._
2. Expand _More Options_ and choose _Microsoft Azure Virtual Machines_
3. Create a new VM
    1. Set a DNS name; if you choose _example_, you'll get _example.cloudapp.net_
    2. Choose a **public image** matching the environment you want
        1. I chose _Windows Server 2012 R2 Datacenter, September 2014_, which is the latest at this time
        2. Leave _Enable IIS and Web Deploy_ checked
    3. Select a VM size
    4. Specify a username and password to be set up as the default administrator user; for this example, I'll go with _user_ and _password_
    5. Select a location
    6. Click OK
4. Visual Studio will tell you that the VM is being created; click OK to dismiss the prompt
5. After the VM is created and configured, right click on the web project and choose _Publish..._ again
6. Click _Publish_
7. Visit your site (http://example.cloudapp.net) and verify that the project was deployed
8. Once it's properly deployed, RDP into your server and set the admin user's password to not expire

If you choose to use an existing server or create one outside of Azure, you 
will need to 
[configure Web Deploy](http://www.iis.net/learn/manage/remote-administration/configuring-remote-administration-and-feature-delegation-in-iis-7) 
and 
[create a publish profile](http://msdn.microsoft.com/en-us/library/dd465337(v=vs.110).aspx).

### Build Definition

If you don't have a CI build definition already, 
[create one](http://www.visualstudio.com/en-us/get-started/build-your-apps-vs.aspx). 
Ensure you select _continuous integration_ on the _Trigger_ tab.

Once you have a CI build definition, edit it (open the Team Explorer pane, 
click the Home button if necessary, click Builds, right click on the definition 
and edit).

Go to the _Process_ tab and look for the option to specify arguments to 
MSBuild. I can't tell you exactly where this is as it depends on what template 
the project was created with.

Once you find it, specify the following arguments:

+ <kbd>/p:VisualStudioVersion=12.0</kbd>
+ <kbd>/p:DeployOnBuild=true</kbd>
+ <kbd>/p:AllowUntrustedCertificate=true</kbd>
+ <kbd>/p:PublishProfile=</kbd>example<kbd>.pubxml</kbd>
+ <kbd>/p:UserName=</kbd>user
+ <kbd>/p:Password=</kbd>password

Here are those options in a single string to simplify copying and pasting:

    /p:VisualStudioVersion=12.0 /p:DeployOnBuild=true /p:AllowUntrustedCertificate=true /p:PublishProfile=example.pubxml /p:UserName=user /p:Password=password

Be sure to set the correct values for the publish profile and user credentials.

The setting I was doing incorrectly was _PublishProfile_. Most things I saw 
simply had <kbd>example</kbd> and skipped the <kbd>.pubxml</kbd> This resulted 
in MSBuild not attempting to deploy the site, which in turn resulted in me
banging my head against the wall.
