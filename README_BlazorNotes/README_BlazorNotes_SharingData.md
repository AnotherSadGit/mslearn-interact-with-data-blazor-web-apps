Notes about Blazor: Sharing Data between Components
===================================================
Simon Elms, 2 Dec 2022

There are three basic mechanisms for sharing data between components:
1. Component parameters
2. Cascading parameters
3. AppState

Component parameters
--------------------
* Define parameter in child component via \[Parameter\] attribute on property in @code block.
* In parent component set child's parameter value via attributes on child component tag.  May have to include C# code to set the values of parameters that are reference types.
* Limitation: Only for passing values to immediate children, not to grandchildren and components lower in hierarchy. 

### Example
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
		  
Cascading parameters
--------------------
* For passing values to all components lower in hierarchy, not just immediate children.
* In parent component: \<CascadingValue Name="xxx" Value="yyy"\> element, containing the child elements that can access the cascading value.
* In child element @code block: Add a member decorated with \[CascadingParameter\(Name="xxx"\)\] attribute, which is set to the parent component CascadingValue Value.
* Limitation: Only for passing values to descendants included within the parent's \<CascadingValue\> element.
		  
AppState
--------
* Create POCO class representing values to pass or application state.
* Register class as scoped service in DI container.
* Inject the class into any component to read or write its properties.
* Can be used by any component at any level, not just for passing values from a parent component to its descendants.

### Example
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