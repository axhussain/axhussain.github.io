---
layout: post
title: "Dependency Injection: How To Handle Multiple Implementations Using Config"
tags: [c#, dependency-injection]
comments: true
excerpt_separator: <!--more-->
---

I work for a company that has multiple sites across Europe. I recently came across a problem where we had a single application that needed to be installed in each office, but each country's workflow is slightly different. The solution was to have a separate class for each country and to inject the correct class at runtime.

However, we use the "[build once, deploy many](https://medium.com/buildit/build-once-deploy-everywhere-part-1-706d7affaf0f)" strategy which meant we couldn't use DI in the traditional manner. I'll elaborate on this point below.

<!--more-->

## Traditional ASP.NET Core Dependency Injection

A typical ASP.NET Core app registers your services' interfaces and corresponding implementations in `Program.cs`:

```c#
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices(services =>
            {
                services.AddScoped<IMyService, MyServiceA>();
            })
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

However, as you can see, the type registration is in code, thus if we wanted to replace `MyServiceA` with `MyServiceB` we would need to rebuild the app. This approach would not work at my company because, as I've already mentioned, we "build once" via our CI/CD pipeline.

Below we'll look over a couple of potential solutions.

## Using `HostBuilderContext`

We can use the second overload of `ConfigureServices` which takes a `HostBuilderContext` in addition to the `IServiceCollection`:

```c#
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureServices((host, services) =>
        {
            if (host.HostingEnvironment.IsEnvironment("Germany"))
            {
                services.AddScoped<IMyService, MyServiceDE>();
            }
            else
            {
                services.AddScoped<IMyService, MyServiceUK>();
            }
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```
The `HostingEnvironment` class has a number of methods you can use to check your current environment. By default .NET Core has three built-in environments:
- Development
- Staging
- Production

Each of these have corresponding methods: `IsDevelopment()`, `IsStaging()` and `IsProduction()`.

In my example above, I'm checking for a custom environment name which is set in the `ASPNETCORE_ENVIRONMENT` environment variable on the hosting server.

You could also check the `EnvironmentName` property in an `if` or `switch` statement, e.g.

```c#
switch (host.HostingEnvironment.EnvironmentName)    
{
    case "Germany":
        services.AddScoped<IMyService, MyServiceDE>();
        break;
    case "UK":
        services.AddScoped<IMyService, MyServiceUK>();
        break;
    default:
        services.AddScoped<IMyService, MyServiceGlobal>();
        break;
}
```

This method will probably be suitable for my needs, but for the sake of completeness, I wanted to describe another solution.

## Choosing an Implementation at Runtime

You may or may not know that it's possible to register multiple implementations for the same interface, after which, we can use a config setting to select which implementation we want at runtime.

Let's start by registering our implementations:

```c#
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureServices((host, services) =>
        {
            services.AddScoped<IMyService, MyServiceDE>();
            services.AddScoped<IMyService, MyServiceUK>();
            services.AddScoped<IMyService, MyServiceGlobal>();
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

Then in `appsettings.json` we create a key/value to set which implementation we want:

```js
{
  "AppSettings": {
    "CurrentService": "MyServiceDE"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

Now, in our MVC controllers there's two things to note. The first is that we'll be using the [`IOptions` pattern](https://andrewlock.net/how-to-use-the-ioptions-pattern-for-configuration-in-asp-net-core-rc2/) for injecting config, and the second is that instead of having a parameter of type `IMyService` we'll have an `IEnumerable<IMyService>`, e.g:

```c#
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;
    private readonly AppSettings _appSettings { get; set; }
    private readonly IEnumerable<IMyService> _myServices { get; set; }

    public HomeController(ILogger<HomeController> logger, IOptions<AppSettings> settings, IEnumerable<IMyService> myServices)
    {
        _logger = logger;
        _appSettings = settings.Value;
        _myServices = myServices
    }

    public IActionResult Index()
    {
        return View();
    }
}
```

This allows you to select the implementation by the name that you set in the config[^1]:

```c#
public IActionResult Index()
{
    var myService = _myServices.FirstOrDefault(h => h.GetType().Name == AppSettings.CurrentService);
    // use the service
    return View();
}
```

Using this method, I can build once and then let our CI/CD server amend the value of `CurrentService` in `appsettings.json` depending on which country the app is being deployed to.

[^1]: {% include citation.html key="di-multiple-implementations" %}
