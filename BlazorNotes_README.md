Notes about Blazor
==================
Simon Elms, 2 Dec 2022

Directives
----------
### @page directive
Specifies the route to the page

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
                 
Methods of sharing information across components
------------------------------------------------
### Component parameters
* Define parameter in child component via \[Parameter\] attribute on property in @code block.
* In parent component set child's parameter value via attributes on child component tag.  May have to include C# code to set the values of parameters that are reference types.
* Limitation: Only for passing values to immediate children, not to grandchildren and components lower in hierarchy. 

#### Example
Child component - ConfigurePizzaDialog.razor:
	@code {
		[Parameter] public Pizza Pizza { get; set; }
	}
	
Parent component - Index.razor:
	@if (showingConfigureDialog)
	{
		<ConfigurePizzaDialog Pizza="configuringPizza" />
	}

	@code {
		Pizza configuringPizza;
		bool showingConfigureDialog;

		void ShowConfigurePizzaDialog(PizzaSpecial special)
		{
			configuringPizza = new Pizza()
			{
				Special = special,
				SpecialId = special.Id,
				Size = Pizza.DefaultSize
			};

			showingConfigureDialog = true;
		}
	}
		  
### Cascading parameters
* For passing values to all components lower in hierarchy, not just immediate children.
* In parent component: \<CascadingValue Name="xxx" Value="yyy"\> element, containing the child elements that can access the cascading value.
* In child element @code block: Add a member decorated with \[CascadingParameter\(Name="xxx"\)\] attribute, which is set to the parent component CascadingValue Value.
* Limitation: Only for passing values to descendants included within the parent's \<CascadingValue\> element.
		  
### AppState
* Create POCO class representing values to pass.
* Register class as scoped service in DI container.
* Inject the class into any component to read or write its properties.
* Can be used by any component at any level, not just for passing values from a parent component to its descendants.