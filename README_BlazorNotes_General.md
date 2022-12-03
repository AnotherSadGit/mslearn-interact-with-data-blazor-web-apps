Notes about Blazor: General
===========================
Simon Elms, 2 Dec 2022

Directives
----------
### @code directive
Specifies a code block

### Member access directives
eg `@special.Description`.  For reading the value of a member defined in the @code block in the HTML web page.  Note that you can bind to both properties and methods.

### Control-flow directives
@if else, @foreach, @for, @while and @do while

OnInitialized/OnInitializedAsync method
------------------------------------------
Fires when component initialization is complete.  Page has received parameter values but has not yet been rendered.  Defined in component base class but can be overridden for a specific page. 

Project Structure
-----------------
Folders and files that appear in a vanilla project created from a Blazor project template.

See MS Learn page "ASP.NET Core Blazor project structure", https://learn.microsoft.com/en-us/aspnet/core/blazor/project-structure

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
	

