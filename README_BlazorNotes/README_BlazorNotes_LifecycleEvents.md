Notes about Blazor: Lifecycle Events
====================================
Simon Elms, 4 Dec 2022

See also MS Learn article "ASP.NET Core Razor component lifecycle", https://learn.microsoft.com/en-us/aspnet/core/blazor/components/lifecycle

Events that can occur during the lifecycle of a component, and methods that can handle those events:

!(README_BlazorNotes_component-lifecycle.png)

	Component instance created
	|											UI event	External event
	V												|				|
SetParametersAsync									-------  --------
	|													  |  |
	V										await		  |  |
OnInitialized / OnInitializedAync	--------------------  |  |	
	|												   |  |  |
	V										await	   |  |  |
OnParametersSet / OnParametersSetAsync -------------   |  |  |
	|											   |   |  |  |
	|											   |   |  |  |
	V						True				   V   V  V  V
	|<-- BuildRenderTree <------ ShouldRender <-- StateHasChanged
	V	
	Render the UI
	|
	V
OnAfterRender / OnAfterRenderAsync
	|
	V
	Remove from render tree
	|
	V
Dispose / DisposeAsync

Each await operation during OnInitializedAync and OnParametersSetAsync indicates the state of the component has changed, possibly triggering an immediate page render.  The page may be rendered multiple times before initialization is complete so code must take this into account.  For example, data may not be fully loaded from the backend when the page is rendered, so handle the "no data yet" situation (eg display "Loading..." message).

When a user visits a page containing a Blazor component the Blazor runtime creates a new instance of the component via the default contructor.  Then the following methods are run.

SetParametersAsync method
-------------------------
Called after the Blazor component default constructor has completed, when a new instance of the component has been created.  Inherited from ComponentBase but can be overriden if needed.  Sets the component parameter values, including route parameters, if the component has parameters.  Will still run even if the component has no parameters.

If the parameters are changed by the component's parent after the component has been constructed, SetParametersAsync will run again.

Parameters for the Blazor component are available in SetParametersAsync via a ParameterView object.  Normally they would be passed through to the base.SetParametersAsync method to set the parameter values but they can be handled in SetParametersAsync (eg validated before being passed through).

### Example

	@page "/fetchdata"

	...
	@inject WeatherForecastService ForecastService
	...

	<p>@validationMsg</p>
	...

	@code {
		...
		private string validationMsg;
		...

		[Parameter]
		public DateTime DateParam { get; set; }
		
		public override async Task SetParametersAsync(ParameterView parameters)
		{
			if (parameters.TryGetValue<string>(nameof(DateParam), out var value))
			{
				if (value is null)
				{
					validationMsg = "The date should not be null. Defaulting to Now";
					DateParam = DateTime.Now; // default to current date and time if not set
				}
			}

			await base.SetParametersAsync(parameters);
		}
		...
	}
	
OnInitialized / OnInitializedAync methods
-----------------------------------------
Inherited from ComponentBase but can be overriden if needed.  Run after the component has been constructed and the parameters have been set.

May run multiple times, depending on the value of the app's render-mode property:

* render-mode = Server: Only runs once for the component instance.  If component parameters are modified SetParametersAsync will run again but OnInitialized / OnInitializedAync will not.  So if you need to handle parameters being changed, do it in SetParametersAsync, not OnInitialized / OnInitializedAync.

* render-mode = ServerPrerendered: OnInitialized / OnInitializedAync run twice, once during prerender to generate the static output of the page, then again once the SignalR connection has been established with the browser.  For expensive initialization tasks, eg loading data from a web service to set the component state, cache the state info during the prerender to avoid retrieving it again the second time around.  

During prerender there is no connection to the browser so OnInitialized / OnInitializedAync cannot call JavaScript or perform any action that depends on the connection.  Any such logic, which requires a connection to the client browser, should be run in OnAfterRender / OnAfterRenderAsync.

### Example of caching state
Uses the null coalescing assignment operator to call GetForecastAsync.  Will only make the call if forecasts is null.

	@code {
		private WeatherForecast[] forecasts;
		...
		protected override async Task OnInitializedAsync()
		{
			await base.OnInitializedAsync();
			forecasts ??= await ForecastService.GetForecastAsync(DateTime.Now);
		}
		...
	}
	
Dependencies are injected before OnInitialized / OnInitializedAync run so can be used in OnInitialized / OnInitializedAync.

OnParametersSet / OnParametersSetAsync methods
----------------------------------------------
Run after OnInitialized / OnInitializedAync if this is the first time the component is being rendered, or after SetParametersAsync if it's not.  

These methods are always called even if the component has no parameters.

Useful for initialization tasks that depend on parameters, eg calculating computed properties.  Better to do these tasks here than in a constructor since a constructor is synchronous and if the tasks are long-running it would affect the responsiveness of the page.

OnAfterRender / OnAfterRenderAsync methods
------------------------------------------
Run every time the view in the UI represented by the component needs to update.  

UI view updates when:
* The state of the component changes (eg when OnInitialized / OnInitializedAync or OnParametersSet / OnParametersSetAsync run);
* A UI event fires;
* Code calls the StateHasChanged method of the component.

When OnAfterRender / OnAfterRenderAsync run the UI is fully functional and it's possible to interact with elements of the DOM and with JavaScript.  Call JavaScript via JS interop in OnAfterRender / OnAfterRenderAsync.

### StateHasChanged, ShouldRender and BuildRenderTree methods
The StateHasChanged method lets Blazor know the component's state has changed and to rerender the component, if required.  

StateHasChanged can be called automatically, for EventCallback methods, or manually from component code.  

StateHasChanged calls the ShouldRender method of the component, which determines whether the change in state requires a rerender.

By default ShouldRender returns true on state change but the method can be overridden to perform a check that may return false.

If ShouldRender returns true then the BuildRenderTree method is called to generated a model used to update the DOM.  Like the other methods BuildRenderTree is overridable, if needed.

Once BuildRenderTree has run the component view is rendered and the UI updated.  After that OnAfterRender / OnAfterRenderAsync run.

### firstRender parameter for OnAfterRender / OnAfterRenderAsync
True the first time OnAfterRender / OnAfterRenderAsync run for a component, false after that.  Useful for one-time operations that would be time-consuming to perform each time the component is rendered.

NOTE: The first time OnAfterRender / OnAfterRenderAsync run is not the same as the prerender step in OnInitialized / OnInitializedAync.  The first time OnAfterRender / OnAfterRenderAsync run the page will be fully active in the browser and the SignalR connection will have been made.  So it's possible to call JavaScript when firstRender is true.

#### Example
JS interop runs but only the first time the component view is rendered.

	@page "/fetchdata"

	...
	@inject IJSRuntime JS
	...

	@code {
		private WeatherForecast[] forecasts;
		private DotNetObjectReference<FetchData> objRef;
		private ElementReference graphPlaceholder;

		...

		protected override async Task OnAfterRenderAsync(bool firstRender)
		{
			if (firstRender)
			{
				objRef = DotNetObjectReference.Create(this);
				await JS.InvokeVoidAsync("populateObjectRef", objRef);

				await JS.InvokeVoidAsync("changeTitle", "Weather Forecast");
				var forecastTemperatures = (from t in forecasts
											select t.TemperatureC).ToArray();

				await JS.InvokeVoidAsync("showGraph", TimeSpan.FromSeconds(30), graphPlaceholder, forecastTemperatures);
			}
		}
	}
	
Dispose / DisposeAsync methods
------------------------------
For releasing unmanaged resources.

Need an @implements directive, either IDisposable for the Dispose method or IAsyncDisposable for the DisposeAsync method.

### Example

	@page "..."
	@using System.Timers
	@implements IDisposable

	...

	@code {
		private Timer timer;

		protected override async Task OnInitializedAsync()
		{
			await base.OnInitializedAsync();
			timer ??= new Timer(1000);
			...
		}

		...

		public void Dispose() => timer?.Dispose();
	}
	
Exception handling in lifecycle methods
---------------------------------------
If a lifecycle method fails it closes the SignalR connection to the browser, breaking the app.  So catch any exceptions.

### Example

	@page "/fetchdata"

	...
	@using Microsoft.Extensions.Logging
	@inject WeatherForecastService ForecastService
	@inject ILogger<FetchData> Logger

	<h1>Weather forecast</h1>

	<p>This component demonstrates fetching data from a service.</p>

	@if (loadFailed)
	{
		<h1>Sorry, we could not load weather forecast data due to an error.</h1>
	} 
	else if (forecasts == null)
	{
		<p><em>Loading...</em></p>
	}
	else
	{
		... // display a table of weather forecast data
	}

	@code {
		private WeatherForecast[] forecasts;
		private bool loadFailed = false;
		...

		protected override async Task OnInitializedAsync()
		{
			try
			{
				await base.OnInitializedAsync();
				forecasts ??= await ForecastService.GetForecastAsync(DateTime.Now);
			}
			catch (Exception e)
			{
				loadFailed = true;
				Logger.LogWarning(e, "Failed to load weather forecast data");
			}
		}
		...
	}
	
