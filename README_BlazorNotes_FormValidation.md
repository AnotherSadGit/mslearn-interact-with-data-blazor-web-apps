Notes about Blazor: Form Validation
===================================
Simon Elms, 3 Dec 2022

Occurs on EditForm submission.

Two types of form validation: 
* Declarative: On client side.  Fairly basic validation only.  Prevents basic user errors.
* Programmatic: On server side.  Can handle more complex validation, eg cross-checking data in form against other data.  Prevents attempts by user to circumvent validation.  Also prevents storing incomplete or corrupt data.

Validation to avoid
-------------------
Avoid validating on JavaScript events like onchange or oninput, or the Blazor equivalent @onchange or @oninput.  They would fire for each keystroke so a user might see a validation error message on each keystroke as they enter data.  Instead, use the supplied validation mechanisms which wait until user has finished entering data to validation it.

Form submission events
----------------------
Three events run on EditForm submission:
* OnSubmit: Fires when EditForm submitted.
* OnValidSubmit: Fires if all input fields pass validation rules defined by their validation attributes.
* OnInvalidSubmit: Fires if any input field fails a validation rule defined by a validation attribute

OnValidSubmit and OnInvalidSubmit are used with simple validation rules.  For more complex validation, such as cross-checking form data against another data source, use the OnSubmit event.

An EditForm can either handle the OnSubmit event or the OnValidSubmit and OnInvalidSubmit pair of events.  In cannot handle all three events.  Trying to handle all three events will result in a runtime error:
* System.InvalidOperationException: When supplying an OnSubmit parameter to EditForm, do not also supply OnValidSubmit or OnInvalidSubmit.

An EditContext object tracks changes to the Model object.  The EditContext object is passed to the three submit events as a parameter so event handler methods should include an EditContext parameter.  An event handler can retrieve user input from the EditContext object Model field.

### Example
Using the OnSubmit event.

	<EditForm Model="@shirt" OnSubmit="ValidateData">
		<!-- Omitted for brevity -->
		<input type="submit" class="btn btn-primary" value="Save"/>
		<p></p>
		<div>@Message</div>
	</EditForm>

	@code {
		private string Message = String.Empty;

		// Omitted for brevity

		private async Task ValidateData(EditContext editContext)
		{
			if (editContext.Model is not Shirt shirt)
			{
				Message = "T-Shirt object is invalid";
				return;
			}

			if (shirt is { Color: ShirtColor.Red, Size: ShirtSize.ExtraLarge })
			{
				Message = "Red T-Shirts not available in Extra Large size";
				return;
			}

			if (shirt is { Color: ShirtColor.Blue, Size: <= ShirtSize.Medium)
			{
				Message = "Blue T-Shirts not available in Small or Medium sizes";
				return;
			}

			if (shirt is { Color: ShirtColor.White, Price: > 50 })
			{
				Message = "White T-Shirts must be priced at 50 or lower";
				return;
			}

			// Data is valid
			// Save the data
			Message = "Changes saved";
		}
	}

Declarative validation
----------------------
Via:
* Data annotations: Against each Model property.  For in dicating if a property is required, what the valid data range is and the data format.
* DataAnnotationsValidator component within the EditForm component: Checks entered values against Model data annotations.
* ValidationSummary component: Displays summary of all validation messages for submitted form.
* ValidationMessage component: Displays validation message for specific Model property.

### Data annotations for Model class
Possible data annotations:
* [Required]
* [Range]
* [EmailAddress]: Checks that entered value is valid email address.
* [ValidationNever]: Skip validation for this property.
* [CreditCard]
* [Compare]: For ensuring two properties match.
* [Phone]
* [RegularExpression]: Checks value via a regex.
* [StringLength]
* [Url]: Checks the entered value is valid URL.

Out of the box annotations are found in the System.ComponentModel.DataAnnotations namespace.  The Model class will need a using directive:

    using System.ComponentModel.DataAnnotations;

Can add custom error messages to data annotations in Model class.  These will be displayed in either ValidationSummary or ValidationMessage.

#### Example

	public class Pizza
	{
		public int Id { get; set; }
		
		[Required(ErrorMessage = "You must set a name for your pizza.")]
		public string Name { get; set; }
		
		public string Description { get; set; }
		
		[EmailAddress(ErrorMessage = "You must set a valid email address for the chef responsible for the pizza recipe.")]
		public string ChefEmail { get; set;}
		
		[Required]
		[Range(10.00, 25.00, ErrorMessage = "You must set a price between $10 and $25.")]
		public decimal Price { get; set; }
	}

### Custom data annotation
Via class that inherits ValidationAttribute.  Override the IsValid method.

#### Example
Custom ValidationAttribute class:

	public class PizzaBase : ValidationAttribute
	{
		public string GetErrorMessage() => $"Sorry, that's not a valid pizza base.";

		protected override ValidationResult IsValid(
			object value, ValidationContext validationContext)
		{
			if (value != "Tomato" || value != "Pesto")
			{
				return new ValidationResult(GetErrorMessage());
			}

			return ValidationResult.Success;
		}
	}
	
Using it in the Model class:

	public class Pizza
	{
		public int Id { get; set; }
		
		[Required(ErrorMessage = "You must set a name for your pizza.")]
		public string Name { get; set; }
		
		public string Description { get; set; }
		
		[EmailAddress(
			ErrorMessage = "You must set a valid email address for the chef responsible for the pizza recipe.")]
		public string ChefEmail { get; set;}
		
		[Required]
		[Range(10.00, 25.00, ErrorMessage = "You must set a price between $10 and $25.")]
		public decimal Price { get; set; }
		
		[PizzaBase]
		public string Base { get; set; }
	}

### Declarative validation within EditForm component
* Add DataAnnotationsValidator component
* Add either single ValidationSummary component or ValidationMessage components for each control on the form.
* ValidationMessage is linked to a specific Input component via the For attribute.

#### Example

	@page "/admin/createpizza"

	<h1>Add a new pizza</h1>

	<EditForm Model="@pizza">
		<DataAnnotationsValidator />
		<ValidationSummary />
		
		<InputText id="name" @bind-Value="pizza.Name" />
		<ValidationMessage For="@(() => pizza.Name)" />
		
		<InputText id="description" @bind-Value="pizza.Description" />
		
		<InputText id="chefemail" @bind-Value="pizza.ChefEmail" />
		<ValidationMessage For="@(() => pizza.ChefEmail)" />
		
		<InputText id="price" @bind-Value="pizza.Price" />
		<ValidationMessage For="@(() => pizza.Price)" />
	</EditForm>

	@code {
		private Pizza pizza = new();
	}
	
### When validation occurs
Valdiation is performed twice:

1. Field validation: When user tabs out of a field.  This lets the user know the input they've just entered is invalid before they move to the next field.
2. Model validation: When user submits form.  This prevents the saving of invalid data.

Server-side validation
----------------------
Via OnSubmit or OnValidSubmit & OnInvalidSubmit event handlers.

Can be programmatic, via OnSubmit event handler with custom validation code, or declarative via OnValidSubmit & OnInvalidSubmit event handlers.

Using OnSubmit prevents the other two events from firing.

### OnSubmit event handling
Must check the EditContext argument to see if input is valid or not.

Use OnSubmit for custom validation logic.

#### Example

	@page "/admin/createpizza"

	<h1>Add a new pizza</a>

	<EditForm Model="@pizza" OnSubmit=@HandleSubmission>
		<DataAnnotationsValidator />
		<ValidationSummary />
		
		<InputText id="name" @bind-Value="pizza.Name" />
		<ValidationMessage For="@(() => pizza.Name)" />
		
		<InputText id="description" @bind-Value="pizza.Description" />
		
		<InputText id="chefemail" @bind-Value="pizza.ChefEmail" />
		<ValidationMessage For="@(() => pizza.ChefEMail)" />
		
		<InputText id="price" @bind-Value="pizza.Price" />
		<ValidationMessage For="@(() => pizza.Price" />
	</EditForm>

	@code {
		private Pizza pizza = new();
		
		void HandleSubmission(EditContext context)
		{
			bool dataIsValid = context.Validate();
			if (dataIsValid)
			{
				// Store valid data here
			}
		}
	}

### OnValidSubmit & OnInvalidSubmit event handling
Don't require checking the EditContext argument to see if input is valid or not.

#### Example

	@page "/admin/createpizza"

	<h1>Add a new pizza</a>

	<EditForm Model="@pizza" OnValidSubmit=@ProcessInputData OnInvalidSubmit=@ShowFeedback>
		<DataAnnotationsValidator />
		<ValidationSummary />
		
		<InputText id="name" @bind-Value="pizza.Name" />
		<ValidationMessage For="@(() => pizza.Name)" />
		
		<InputText id="description" @bind-Value="pizza.Description" />
		
		<InputText id="chefemail" @bind-Value="pizza.ChefEmail" />
		<ValidationMessage For="@(() => pizza.ChefEMail)" />
		
		<InputText id="price" @bind-Value="pizza.Price" />
		<ValidationMessage For="@(() => pizza.Price" />
	</EditForm>

	@code {
		private Pizza pizza = new();
		
		void ProcessInputData(EditContext context)
		{
			// Store valid data here
		}
		
		void ShowFeedback(EditContext context)
		{
			// Take action here to help the user correct the issues
		}
	}
	
