---
layout: post
title:  "Dynamics are worse than you thought"
date:  2023-07-30 16:20
categories: .Net C#
repo_url: https://github.com/DanielKiddTech/dotnetexamples/tree/main/dynamics-are-bad
tags: .Net C#
---
Many years ago I was working on a .NET Framework website that was developed mainly [C# dynamics](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic){:target="_blank"}.  The project was many years into its lifetime before I got to it, but as far as I know the main reason this was done was due to the choice of ORM.  [Massive](https://github.com/FransBouma/Massive){:target="_blank"} was chosen as it allows you to get up and running very quickly as you don't need to worry about any of those pesky models.

I am sure the initial idea was to use this as a starting point and then to move over to another ORM which was strongly typed, which did not happen as quickly as it should have.  About 7 years into the projects lifetime I started the effort of moving any new development to use [Dapper](https://github.com/DapperLib/Dapper){:target="_blank"} as the ORM of choice, we did also try and move some old sections of the code but were reluctant to touch any critical parts of the system unless the really needed changing.

The system was hosted on two load-balanced web-servers, which during our peak period would start to throw 503 errors and then eventually kill the IIS process entirely.  

We spent time reviewing our errors logs, with nothing unusual being logged.  We then started reviewing the Windows Performance Monitor to see if anything jumped out to us, at which point we came across the **.NET CLR Exceptions** counter.  It seemed that thousands of exceptions were being thrown, but nothing was being caught by our internal error handlers, which was very odd.

After some further digging, I came across a handy snippet of code that can catch errors before anything else:

{% highlight csharp %}

AppDomain.CurrentDomain.FirstChanceException += (sender, eventArgs) =>
{
    System.IO.File.AppendAllLines(".\\errorlog.txt",
                    new List<string> { $"{DateTime.Now.ToString("dd/MM/yy HH:mm:ss")}: {eventArgs.Exception.Message}. {eventArgs.Exception.StackTrace} {Environment.NewLine}" });

    Console.WriteLine($"{DateTime.Now.ToString("dd/MM/yy HH:mm:ss")}: {eventArgs.Exception.Message}. {eventArgs.Exception.StackTrace} {Environment.NewLine}");
};

{% endhighlight %}

When I then ran the solution locally I found hundreds of errors being logged for every simple page load that looked something like:

{% highlight text %}

'System.Dynamic.ExpandoObject' does not contain a definition for 'Name'.    at Microsoft.CSharp.RuntimeBinder.RuntimeBinder.BindProperty(ICSharpBinder payload, ArgumentObject argument, LocalVariableSymbol local, Expr optionalIndexerArguments) 

{% endhighlight %}

I never found an article matching my exact scenario, but the assumption here is every time that a dynamic property is being used during runtime, that .Net is internally trying to figure out what type is being used which seems to throw Exceptions internally, which is why the errors were never being caught by our internal logger, as the errors were being thrown and handled at a low level by the .NET CLR itself.

Intead of searching further for a definitive answer the simplest solution was to very quickly switch whatever I could away from using dynamics.  

Obviously on a 7 year old product, it would be naive and irresponsible to think I could convert the whole product over quickly with no issues.  Instead, I decided to take a phased approach.  First I tackled the core functionality that runs on every page load, this was somewhat trivial and allowed us to release a fix within a few days, immediately resolving the 503 errors we were receiving during our peak times.  After that, we started a procedure to convert any areas that were being updated as part of any other updates.

From this experience there were a few lessons learned:
1. Never start a product with the assumption thatt the approach or core of it can be changed at a later date, as this will rarely happen
2. When using a strongly-typed language, make use of the features it provides
3. Don't use dynamics for production-grade products 