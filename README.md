#Cake

Cake (C# Make) is a build automation system using C#.

[![TeamCity CI Build Status](http://builds.nullreferenceexception.se/app/rest/builds/buildType:id:Cake_Continuous/statusIcon)](http://builds.nullreferenceexception.se/viewType.html?buildTypeId=Cake_Continuous&guest=1)

##Roadmap

The Cake engine is pretty much done, but there are still improvements to be made. I'm still experimenting with the script API to make it as easy and intuitive as possible, so expect changes along the road.

Currently basic MSBuild, xUnit, NuGet, ILMerge, NUnit, MSTest, compression and file system operations are implemented, but more features are planned. A roadmap can be found [here](https://github.com/cake-build/cake/issues/milestones).

For more information and examples of how to use Cake, see the [Wiki](https://github.com/cake-build/cake/wiki).

##Example

###1. Download Cake

```Batchfile
C:\Project> NuGet.exe install Cake -OutputDirectory Tools -ExcludeVersion
```

###2. Create build script

```CSharp
var target = Argument("target", "NuGet");
var configuration = Argument("configuration", "Release");

Task("Clean")
    .Does(() =>
{
    // Clean directories.
    CleanDirectory("./build");
    CleanDirectory("./build/bin");
    CleanDirectories("./src/**/bin/" + configuration);
});

Task("Restore-NuGet-Packages")
    .IsDependentOn("Clean")
    .Does(context =>
{
    // Restore NuGet packages.
    NuGetRestore("./src/Cake.sln");    
});

Task("Build")
    .IsDependentOn("Restore-NuGet-Packages")
    .Does(() =>
{
    MSBuild("./src/Cake.sln", s => 
        s.SetConfiguration(configuration));
});

Task("Unit-Tests")
    .IsDependentOn("Build")
    .Does(() =>
{
    XUnit("./src/**/bin/" + configuration + "/*.Tests.dll");
});

Task("Copy-Files")
    .IsDependentOn("Unit-Tests")
    .Does(() =>
{
    var sourcePath = "./src/Cake/bin/" + configuration;    
    var files = GetFiles(sourcePath + "/**/*.dll") + GetFiles(sourcePath + "/**/*.exe");
    var destinationPath = "./build/bin";

    CopyFiles(files, destinationPath);
});

Task("Pack")
    .IsDependentOn("Copy-Files")
    .Does(() =>
{   
    var root = "./build/bin";
    var output = "./build/" + configuration + ".zip";

    Zip(root, output);
});

Task("NuGet")
    .Description("Create NuGet package")
    .IsDependentOn("Pack")
    .Does(() =>
{
    // Create NuGet package.
    NuGetPack("./Cake.nuspec", new NuGetPackSettings {
        Version = "0.1.0",
        BasePath = "./build/bin",
        OutputDirectory = "./build",
        NoPackageAnalysis = true
    });
});

RunTarget(target);
```

###3. Run build script

```
C:\Project\Tools\Cake> Cake.exe ../../build.csx -verbosity=verbose -target=Pack
```

## Contributing

* Fork the repository.
* Make your feature addition or bug fix.
* Don't forget the unit tests.
* Send a pull request. Bonus for topic branches. *Funny .gif will be your reward.*

## License

Copyright (c) 2014, Patrik Svensson and contributors.   
Cake is provided as-is under the MIT license. For more information see `LICENSE`.

* For Roslyn, see https://roslyn.codeplex.com/license
* For Autofac, see https://github.com/autofac/Autofac/blob/master/LICENSE
* For NuGet.Core, see https://nuget.codeplex.com/license


