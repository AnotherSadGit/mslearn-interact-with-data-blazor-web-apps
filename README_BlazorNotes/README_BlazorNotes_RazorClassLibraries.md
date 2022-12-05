Notes about Blazor: Razor Class Libraries
=========================================
Simon Elms, 5 Dec 2022

Razor class libraries allow UI components to be shared between Blazor apps.

A Razor class library can contain:
* Razor components
* Pages
* HTML
* CSS files
* JavaScript
* Images
* Other static content

Razor class libraries can be packaged as NuGet packages.

Creating a Razor class library
------------------------------

### Via Visual Studio
File > New Project > Razor Class Library


### Via dotnet CLI
dotnet new razorclasslib -o ProjectFolderName -f net6.0 -n MyProjectName

No need to add -n if the project name and project folder name are going to be the same.  -n (--name) defaults to the foler name specified by -o (--output).

The Razor class library is created with the following default components:
* A single component, Component1.razor
* A CSS file, Component1.razor.css, in the same folder as Component1.razor.  This will be combined with CSS files in the host application project.
* Static content, such as images and JavaScript files, in a wwwroot folder
* .NET code, which executes the functions in the JavaScript file.

Difference from a standard class library
----------------------------------------
* Project file contains a reference to Microsoft.NET.Sdk.Razor
* SupportedPlatform specifies browser, meaning the Razor class library can be used in the browser as a WebAssembly
* PackageReference to Microsoft.AspNetCore.Components.Web gives access to the Razor components shipped with the framework. 

### Example project file

	<Project Sdk="Microsoft.NET.Sdk.Razor">

	  <PropertyGroup>
		<TargetFramework>net6.0</TargetFramework>
		<Nullable>enable</Nullable>
		<ImplicitUsings>enable</ImplicitUsings>
	  </PropertyGroup>

	  
	  <ItemGroup>
		<SupportedPlatform Include="browser" />
	  </ItemGroup>

	  <ItemGroup>
		<PackageReference Include="Microsoft.AspNetCore.Components.Web" Version="6.0.0" />
	  </ItemGroup>

	</Project> 

Referencing static assets in wwwroot folder
-------------------------------------------

### From within the Razor class library
There is no need to include the path to the wwwroot folder when referencing files within it from the CSS files.  

#### Example
The background.png image file in wwwroot is referenced from the Component1.razor.css file as:

	.my-component {
		background-image: url('background.png');
	}
	
### From Blazor applications
Requires a path of the format:

	/_content/{PACKAGE_ID}/{PATH_AND_FILENAME_INSIDE_WWWROOT}
	
Referencing a Razor class library
---------------------------------
If the Razor class library is on disk next to the Blazor application referencing it, then a straight-forward project reference can be used in Visual Studio.

Via the dotnet CLI, the command is: `dotnet add reference ../MyClassLibrary`

If the Razor class library is packaged in a NuGet package, add it via the Visual Studio package manager or via the dotnet CLI command `dotnet add package MyClassLibrary`.

Packaging Razor class library as NuGet package
----------------------------------------------
NuGet package requires the following fields in the .csproj file:
* PackageId: Uniquely identifies the package in the NuGet repository.  Defaults to AssemblyName of the class library.
* Version: Major.Minor.Patch[-Suffix], where -Suffix is optional, indicating a pre-release version (eg "-alpha").  Defaults to 1.0.0
* Authors: Defaults to AssemblyName.
* Company: Defaults to AssemblyName.

Optional package properties:
See "MSBuild pack target", https://learn.microsoft.com/en-us/nuget/reference/msbuild-targets#pack-target, for full list of properties.

Optional package properties include:
* Description
* Copyright notice
* Licensing information
* Icons
* Project URLs

### Example .csproj file

	<PropertyGroup>
		<PackageId>Learn.MyFirstLibrary</PackageId>
		<Version>0.1.0-alpha1</Version>
		<Authors>Susan Developer, Terry Programmer</Authors>
		<Company>AdventureWorks</Company>
	</PropertyGroup>

### Pack the project
To pack the package and add it to the output folder of the project:
* Visual Studio: Solution Explorer > right click on project file > Pack
* Dotnet CLI: dotnet pack

To configure the project to generate a package on build: 
Via `<GeneratePackageOnBuild>True</GeneratePackageOnBuild>` in the same PropertyGroup as the other NuGet package properties.



