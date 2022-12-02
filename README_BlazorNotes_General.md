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

Shared components
-----------------
Razor pages which can be called from other Razor pages.

The parent page incorporates a child component via a tag matching the child component page name.

#### Example
Shared component: ConfigurePizzaDialog.razor
  
Parent page calling shared component: Index.razor
Calls the shared component via a tag matching the child component page name:

	@if (showingConfigureDialog)
	{
		<ConfigurePizzaDialog Pizza="configuringPizza" />
	}