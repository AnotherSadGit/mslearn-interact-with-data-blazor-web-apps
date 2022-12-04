Notes about Blazor: JavaScript Interoperability - Calling JavaScript from .NET Code
===================================================================================
Simon Elms, 4 Dec 2022

For Blazor Server apps, requires a SignalR connection from the server to the browser.

In the Blazor component, inject IJSRuntime below the @page directive.

### Example

	@page "/fetchdata"

	@using MyWebApplication.Data
	@inject WeatherForecastService ForecastService
	@inject IJSRuntime JS

	<h1>Weather forecast</h1>
	...
	
IJSRuntime includes two methods for calling JavaScript functions:
* InvokeAsync: For calling JavaScript functions that return a value.
* InvokeVoidAsync: For calling void functions that do not return a value.

Both methods are asynchronous so must be called with await keyword.

Both methods take the name of the JavaScript function followed by the arguments it requires. 

The JavaScript function being called must be part of the window scope, or a sub-scoped under window.

The arguments must be serializable into JSON.

Calling JavaScript from .NET cannot occur until rendering is complete.  To determine whether rendering has finished use OnAfterRender event in Blazor code.

### Example
Using JavaScript to change the <title> within the <head> element of a page.  This has to be done via JavaScript as a Blazor Server component cannot access the <head> element.

JavaScript:

	window.changeTitle = async (newTitle) => {
		document.title = newTitle;
	};
	
Blazor component:

	@code {
		...

		protected override async Task OnAfterRenderAsync(bool firstRender)
		{
			if (firstRender)
			{
				await JS.InvokeVoidAsync("changeTitle", "Weather Forecast");
			}
		}
		...
	}
	
Passing a Blazor element to JavaScript
--------------------------------------
Blazor keeps a server-side copy of the DOM so if a JavaScript script modifies the DOM on the client-side it may get out of sync with the Blazor copy of the DOM.  To avoid this add a placeholder element to the Blazor component, like a <div>.  Blazor sees it as empty so won't track its contents.

Pass an element from Blazor to JavaScript via an ElementReference object:

* Declare an ElementReference field in Blazor code
* Link it to an element in the Blazor component via a @ref attribute, set to the name of the ElementReference field.

### Example
The empty <div> that will be passed to JavaScript has a @ref attribute.  The JavaScript function being called is showGraph from the Plotly library.

	@page "/fetchdata"
	...
	@inject IJSRuntime JS

	<h1>Weather forecast</h1>

	...
		<table class="table">
			...
		</table>

		<div @ref=graphPlaceholder></div>
	}

	@code {
		...
		private ElementReference graphPlaceholder;
		
		protected override async Task OnAfterRenderAsync(bool firstRender)
		{
			if (firstRender)
			{
				...
				var forecastTemperatures = (from t in forecasts
											select t.TemperatureC).ToArray();

				await JS.InvokeVoidAsync("showGraph", graphPlaceholder, forecastTemperatures);
			}
		}

		...
	}
	
This is the Pages/_Host.cshtml file.  It includes two script elements.  The first loads the Plotly library.  The second creates the showGraph function that uses the Plotly library:

	<body>
		...
		<script src="https://cdn.plot.ly/plotly-2.3.1.min.js"></script>
		<script>
		  window.showGraph = (graphDiv, data) => {
			  var plotData = {y: data, type: 'lines'};
			  Plotly.newPlot(graphDiv, [plotData], {title: 'Celsius Temperatures by Day'});
		  }
		</script>
	</body>
	
Handling loss of connectivity between browser and server
--------------------------------------------------------
Blazor communicates with the browser via SignalR.  If there is a loss of connectivity while calling InvokeAsync or InvokeVoidAsync IJSRuntime will throw a JSException.  This needs to be handled in Blazor code.

IJSRuntime has a default timeout of 60 seconds before it throws an exception on loss of connectivity.

### Changing the timeout globally
Set the JSInteropDefaultCallTimeout option in the services.AddServerSideBlazor method in Program.cs.

#### Example
	// Add services to the container.
	...
	builder.Services.AddServerSideBlazor(options => options.JSInteropDefaultCallTimeout = TimeSpan.FromSeconds(120));
	...
	
### Changing the timeout for a particular call
Specify the timeout as a second parameter to the InvokeAsync or InvokeVoidAsync methods.

#### Example
    await JS.InvokeVoidAsync("showGraph", TimeSpan.FromSeconds(30), graphPlaceholder, forecastTemperatures);

