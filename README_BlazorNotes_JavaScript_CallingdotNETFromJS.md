Notes about Blazor: JavaScript Interoperability - Calling .NET Code from JavaScript
===================================================================================
Simon Elms, 4 Dec 2022

Via the invokeMethod and invokeMethodAsync methods of the DotNet class in the JS interop library.

invokeMethodAsync is better to use for keeping your web app responsive.  It returns a JavaScript promise.

The .NET method being called must be public and have a JSInvokable attribute.  Any parameters must be serializable as JSON.

When calling an asynchronous method the return type must be one of:
* void
* Task
* Task<T>, where T is serializable as JSON.

Calling a .NET static method
----------------------------
When calling a static method specify:
* The .NET assembly containing the class
* The method name
* Any arguments the method requires

### Example
JavaScript calling .NET static method CalculateAveragesAsync to calculate average temperature in degrees C and F.

Blazor code:

	@code {
		private static WeatherForecast[] forecasts; // NOTE THIS IS STATIC - CAUSES A BUG IN ENVIRONMENT WITH MULTIPLE USERS

		...

		[JSInvokable]
		public static async Task<decimal[]> CalculateAveragesAsync()
		{
			var forecastTemperatures = from f in forecasts
									   select (f.TemperatureF, f.TemperatureC);

			var avgF = await Task.FromResult(forecastTemperatures.Average(t => t.TemperatureF));
			var avgC = await Task.FromResult(forecastTemperatures.Average(t => t.TemperatureC));

			return new[] { (decimal)avgF, (decimal)avgC };
		}
	}
	
JavaScript function calculateAverages in the Pages/_Host.cshtml file:

	<script>
		...
		// Replace "<Blazor app assembly name>" with the name of the .NET assembly containing 
		// CalculateAveragesAsync method (ie the Blazor app's assembly name).
		window.calculateAverages = async () => {
			var averages = await DotNet.invokeMethodAsync('<Blazor app assembly name>', 'CalculateAveragesAsync');
			$('#avgF').html(averages[0]);
			$('#avgC').html(averages[1]);
		};
	</script>
	
Calling JavaScript function calculateAverages:

	...
	<table class="table">
		<thead>
			...
		</thead>
		<tbody>
			...
		</tbody>
		<tfoot>
			<tr>
				<td><button onclick="calculateAverages()">Averages</button></td>
				<td></td>
				<td id="avgC"></td>
				<td></td>
				<td id="avgF"></td>
				<td></td>
				<td></td>
			</tr>
		</tfoot>
	</table>
	...
	
Calling a .NET instance method
------------------------------
When calling an instance method specify:
* An object representing the instance containing the method to run.  Use DotNetReference to create the object reference and make this available to JavaScript.
* The method name
* Any arguments the method requires
Also dispose of the object reference in the .NET code once its no longer needed.

### Example
Modify CalculateAveragesAsync to call an instance .NET method.

Blazor code (additions to page used in static method call example):
Note the object reference created via DotNetObjectReference which is passed into the InvokeVoidAsync method.

	...
	@implements IDisposable

	<h1>Weather forecast</h1>
	...

	@code {
		...
		private DotNetObjectReference<FetchData> objRef;
		
		...

		// Create the object reference once rendering is complete, because JS interop is not available until rendering is complete.
		protected override async Task OnAfterRenderAsync(bool firstRender)
		{
			if (firstRender)
			{
				objRef = DotNetObjectReference.Create(this);
				await JS.InvokeVoidAsync("populateObjectRef", objRef);
			}
		}

		...

		public void Dispose()
		{
			objRef?.Dispose();
		}
	}
	
Add object reference to the Pages/_Host.cshtml file:

	<script>
		...
		var objectRef;
		
		window.populateObjectRef = (ref) => {
			objectRef = ref;
		};

		...
	</script>

Modify the calculateAverages JavaScript function in Pages/_Host.cshtml to call the .NET method via the object reference:

	<script>
		...
		window.calculateAverages = async () => {
			var averages = await objectRef.invokeMethodAsync('CalculateAveragesAsync');
			$('#avgF').html(averages[0]);
			$('#avgC').html(averages[1]);
		};
	</script>