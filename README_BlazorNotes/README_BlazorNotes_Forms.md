Notes about Blazor: Forms
=========================
Simon Elms, 3 Dec 2022

<EditForm> Blazor component
---------------------------
Replacement for standard <form> element.

### Advantages of EditForm over form
* Data binding: Link an object to the EditForm
* Validation: Via attributes.
* Form submission: HTML form sends post request to a form handler when it's submitted.  The form handler performs the submit and displays the results.  In Blazor use a C# event handler wired to the @onsubmit event to perform the submit.
* Input elements: EditForm can use traditional <input> elements with a submit button.  However, it can also use Blazor components which include validation and data binding.

Data Binding
------------
Via Editform Model attribute.  Assign a C# object to the Model.  Input elements in the EditForm are bound to properties or fields of the Model object via the @bind-Value parameter.

Data binding is two-way - updates to the EditForm will be pushed into the Model, and updates to the Model will be displayed in the EditForm.

### Example

	@page "/fetchdata"

	@using WebApplication.Data
	@inject WeatherForecastService ForecastService

	<h1>Weather forecast</h1>

	<input type="number" width="2" min="0" max="@upperIndex" @onchange="ChangeForecast" value="@index"/>

	<EditForm Model=@currentForecast>
		<InputDate @bind-Value=currentForecast.Date></InputDate>
		<InputNumber @bind-Value=currentForecast.TemperatureC></InputNumber>
		<InputText @bind-Value=currentForecast.Summary></InputText>
	</EditForm>

	@code {
		private WeatherForecast[] forecasts;
		private WeatherForecast currentForecast;
		private int index = 0;
		private int upperIndex = 0;

		protected override async Task OnInitializedAsync()
		{
			forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
			currentForecast = forecasts[index];
			upperIndex = forecasts.Count() - 1;
		}

		private async Task ChangeForecast(ChangeEventArgs e)
		{
			index = int.Parse(e.Value as string);
			if (index <= upperIndex && index >= 0)
			{
				currentForecast = forecasts[index];
			}
		}
	}
	
Model is set to currentForecast object, which is of type WeatherForecast.  The various input elements in the EditForm are bound to properties of the currentForecast object.

Input elements
--------------
Instead of a generic <input> element in HTML, which takes different forms depending on the type attribute, EditForm has different types of input elements for different data types.

Blazor input elements: 
* InputCheckbox
* InputDate<TValue>
* InputFile
* InputNumber<TValue>
* InputRadio<TValue>
* InputRadioGroup<TValue>
* InputSelect<TValue> (eqivalent of HTML select)
* InputText
* InputTextArea (equivalent of HTML textarea)

### Attributes
Different Blazor input elements have different attributes.  Unrecognised attributes are passed to the client unchanged, allowing HTML attributes, like min, max and step, to be mixed with Blazor attributes such as DisplayName.  They can be used when the Blazor input element is rendered as an HTML input element on the client side.

@ref is used to link an input element to an ElementReference object to allow C# code to change focus to a particular element.

### Radio Group and Radio Buttons
Via InputRadio<TValue> and InputRadioGroup<TValue>.

EditForm can contain muliple InputRadioGroups.  Each group is bound to a field in the underlying Model object.

#### Example
Includes radio groups for picking shirt size and colour, which are enums.

	<EditForm Model="@shirt">
		<label>
			<h3>Size</h3>
			<InputRadioGroup Name="size" @bind-Value=shirt.Size>
				@foreach(var shirtSize in Enum.GetValues(typeof(ShirtSize)))
				{
					<label>@shirtSize:
						<InputRadio Name="size" Value="@shirtSize"></InputRadio>
					</label>
					<br />
				}
			</InputRadioGroup>
		</label>
		<p></p>
		<label>
			<h3>Color</h3>
			<InputRadioGroup Name="color" @bind-Value=shirt.Color>
				@foreach(var shirtColor in Enum.GetValues(typeof(ShirtColor)))
				{
					<label>@shirtColor:
						<InputRadio Name="color" Value="@shirtColor"></InputRadio>
					</label>
					<br />
				}
			</InputRadioGroup>
		</label>
		<p></p>
		<label>
			<h3>Price</h3>
			<InputNumber @bind-Value=shirt.Price min="0" max="100" step="0.01"></InputNumber>
		</label>
	</EditForm>

	@code {
		private Shirt shirt = new Shirt
		{
			Size = ShirtSize.Large,
			Color = ShirtColor.Blue,
			Price = 9.99M
		};
	}
	
Where the Shirt class is:

	public enum ShirtColor
	{
		Red, Blue, Yellow, Green, Black, White
	};

	public enum ShirtSize
	{
		Small, Medium, Large, ExtraLarge
	};

	public class Shirt
	{
		public ShirtColor Color { get; set; }
		public ShirtSize Size { get; set; }
		public decimal Price;
	}

EditForm submission
-------------------
Via a Submit button on the EditForm.
