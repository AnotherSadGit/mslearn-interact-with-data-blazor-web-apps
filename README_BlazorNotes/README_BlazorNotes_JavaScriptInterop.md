Notes about Blazor: JavaScript Interoperability
===============================================
Simon Elms, 3 Dec 2022

Can call JavaScript functions from .NET methods, and .NET methods from JavaScript functions.

Adding JavaScript to a Blazor app
---------------------------------
* Include in <script> tag at the bottom of the <body> in Pages/_Host.cshtml
* Do NOT add <script> tag to *. razor component page.
* Do NOT add <script> tag inside <head> element in a page as Blazor can only control content in the <body> element.
* Can add JavaScript directly in <script> element, or reference a separate JavaScript file.
* Separate JavaScript files should be added under the wwwroot folder of a Blazor app.  When referencing them in a <script> element, only include the path below the wwwroot folder, do not include the wwwroot folder name itself.

### Example

	<!DOCTYPE html>
	<html lang="en">
	<head>
		...
	</head>
	<body>
		...

		<script src="_framework/blazor.server.js"></script>
		<script src="js/gauge.min.js"></script>
		<script>
			window.myFunc = (param1, param2) => {
				...
		</script>
	</body>

The full path to the separate JavaScript file gauge.min.js is wwwroot/js/gauge.min.js.

Ensuring JavaScript files are copied to output directory
--------------------------------------------------------
Edit the .csproj file to add an <ItemGroup> element below the <Project> node.  Include the JavaScript file, with full path below the project root, and set <CopyToOutputDirectory> to Always.

This is particularly necessary when deploying to Azure.

### Example
Modified .csproj file:

	<Project Sdk="Microsoft.NET.Sdk.Web">

	  ...

	  <ItemGroup>
		<Content Include="wwwroot\js\node_modules\canvas-gauges\gauge.min.js">
		  <CopyToOutputDirectory>Always</CopyToOutputDirectory>
		</Content>
	  </ItemGroup>

	  ...

	</Project>


Loading JavaScript files dynamically
------------------------------------
For example, if you need to load different files in different environments.

Can speed up initial page load if the load occurs on an event that fires after the page has been rendered.  Requires setting the autostart attribute of the _framework/blazor.server.js script to false.  This defers initialization of JS interop.  Then have to initialize JS interop manually, via a script, and chain it to a promise that loads the desired script.

### Example
The following JavaScript adds a <script> element to the current page, which will load script file js/gauge.min.js to the current document.

	<body>
		...

		<script src="_framework/blazor.server.js" autostart="false"></script>
		<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
		<script>
		  window.loadscript = async () => {
			  await Blazor.start()
			  $('<script/>', 
				  {src: 'js/gauge.min.js'}
			   ).appendTo('head');
		  };
		</script>
	</body>
	
