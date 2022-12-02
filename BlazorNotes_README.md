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
* Create POCO class representing values to pass or application state.
* Register class as scoped service in DI container.
* Inject the class into any component to read or write its properties.
* Can be used by any component at any level, not just for passing values from a parent component to its descendants.

#### Example
POCO class representing state: Services/OrderState.cs

Registering class as scoped service in DI Container, in Program.cs:

	using BlazingPizza.Services;

	builder.Services.AddScoped<OrderState>();
	
Inject the class representing application state into Index.razor: 

	@using BlazingPizza.Services
	@inject OrderState OrderState
	
Access members of the class representing state in Index.razor:

    @if (specials != null)
    {
      @foreach (var special in specials)
      {
        <li @onclick="@(() => OrderState.ShowConfigurePizzaDialog(special))" style="background-image: url('@special.ImageUrl')">
          <div class="pizza-info">
              <span class="title">@special.Name</span>
                  @special.Description
                <span class="price">@special.GetFormattedBasePrice()</span>
          </div>
        </li>
      }
    }
	
and 

@if (OrderState.ShowingConfigureDialog)
{
    <ConfigurePizzaDialog
      Pizza="OrderState.ConfiguringPizza" />
}

Event handling
--------------
Steps:

1. Add event handler methods (in this example OrderState CancelConfigurePizzaDialog and ConfirmConfigurePizzaDialog methods).  These can be plain methods without parameters (no need for EventArgs).

2. Add EventCallback parameters to the component which will raise the events.  In this example ConfigurePizzaDialog.razor will raise the events so in its @code block:

	@code {
		[Parameter] public Pizza Pizza { get; set; }
		[Parameter] public EventCallback OnCancel { get; set; }
		[Parameter] public EventCallback OnConfirm { get; set; }
	}

3. Wire up the events to the EventCallback parameters in the component that will raise the events.  In this example, the onclick events for the Cancel and Order buttons in the ConfigurePizzaDialog.razor component are wired up to the EventCallback parameters:

        <div class="dialog-buttons">
            <button class="btn btn-secondary mr-auto" @onclick="OnCancel">Cancel</button>
            <span class="mr-center">
                Price: <span class="price">@(Pizza.GetFormattedTotalPrice())</span>
            </span>
            <button class="btn btn-success ml-auto" @onclick="OnConfirm">Order ></button>
        </div>

4. In the parent component, when calling the child that will raise the events, wire up the child's EventCallback parameters to the event handler methods.  In this case the parent is Index.razor.  When calling the child component, ConfigurePizzaDialog, it specifies the event handler methods for the OnCancel and OnConfirm ConfigurePizzaDialog EventCallback parameters:

	@if (OrderState.ShowingConfigureDialog)
	{
		<ConfigurePizzaDialog
		  Pizza="OrderState.ConfiguringPizza"
		  OnCancel="OrderState.CancelConfigurePizzaDialog"
		  OnConfirm="OrderState.ConfirmConfigurePizzaDialog" />
	}
