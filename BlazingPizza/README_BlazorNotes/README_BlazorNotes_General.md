Notes about Blazor: General
===========================
Simon Elms, 2 Dec 2022

Adding debugging components for VS Code
---------------------------------------
VS Code cannot debug a Blazor server app out of the box.  Debugging components need to be added.

Symptoms:
* Hit F5 or select Run > Start Debugging.  
* You will be prompted to select a debugger.  Select the suggested one (probably .NET 5+ and .NET Core).
* Nothing happens.
* If you try debugging again (F5, etc), you will be prompted to select a debugger again.

Solution:
* Select Run > Add Configuration... 
* You will be prompted to select a debugger.  Select the suggested one (probably .NET 5+ and .NET Core).
* A .vscode folder will be added to the project, containing the following files:
	* launch.json
	* tasks.json
After this F5 and debugging will work.

Directives
----------
### @code directive
Specifies a code block

### Member access directives
eg `@special.Description`.  For reading the value of a member defined in the @code block in the HTML web page.  Note that you can bind to both properties and methods.

### Control-flow directives
@if else, @foreach, @for, @while and @do while

### @using directive
Equivalent of C# using directive, for specifying namespaces that will be used in a component.

### @inject directive
Injects a C# object into a component from the DI container (may instantiate it each time, or pass in a scoped instance, or a shared singleton instance).

### @implements directive
Specifies that the Blazor component implements an interface.  eg `@implements IDisposable`

### @typeparam directive
Specifies the type parameter for a component parameter.  Multiple @typeparam directives can be added to the same component, if it has multiple parameters with type parameters.
eg

	@typeparam TItem

	@foreach(TItem item in Data ?? Array.Empty<TItem>())
	{
		<p>@ChildContent(item)</p>
	}


	@code {
	  [Parameter]
	  public IEnumerable<TItem> Data { get; set; }

	  [Parameter]
	  public RenderFragment<TItem> ChildContent { get; set; }
	}


OnInitialized/OnInitializedAsync method
------------------------------------------
Fires when component initialization is complete.  Page has received parameter values but has not yet been rendered.  Defined in component base class but can be overridden for a specific page. 

Project Structure
-----------------
Folders and files that appear in a vanilla project created from a Blazor project template.

See MS Learn page "ASP.NET Core Blazor project structure", https://learn.microsoft.com/en-us/aspnet/core/blazor/project-structure

### _Imports.razor
Razor directives to automatically import into each component page.  eg @using directives for namespaces that may be used in each component.

Displaying an Alert
-------------------
Blazor has no equivalent of a JavaScript alert.  You can call a JavaScript alert via `await IJSRuntime.InvokeVoidAsync("alert", msg);`

### Example

	@page "/counter"
	@inject IJSRuntime JS

	<h1>Counter</h1>

	<p id="currentCount">Current count: @currentCount</p>

	<button class="btn btn-primary" @onclick='mouseEvent => HandleClick(mouseEvent, "Hello")'>Click me</button>

	@code {
		private int currentCount = 0;

		private async Task HandleClick(MouseEventArgs e, string msg)
		{
			if (e.CtrlKey) // Ctrl key pressed as well
			{
				await JS.InvokeVoidAsync("alert", msg);
				currentCount += 5;
			}
			else
			{
				currentCount++;
			}
		}
	}
	
Component lifecycle
-------------------
See MS Learn article "ASP.NET Core Razor component lifecycle", https://learn.microsoft.com/en-us/aspnet/core/blazor/components/lifecycle

Calling StateHasChanged() in a component lets it know its state has changed.  This may cause the component to be re-rendered.  StateHasChanged() is called automatically for EventCallback methods.