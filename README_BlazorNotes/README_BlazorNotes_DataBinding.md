Notes about Blazor: Data Binding
================================
Simon Elms, 2 Dec 2022

In general via a @bind directive.

@bind directive
---------------
### Example

	@page "/"

	<h1>My favorite pizza is: @favPizza</h1>

	<p>
		Enter your favorite pizza:
		<input @bind="favPizza" />
	</p>

	@code {
		private string favPizza { get; set; } = "Margherita"
	}

It specifies the member that will be bound to the element it's added to.

Its effect varies depending on the control it's added to:
* When added to an <input> textbox it binds to the `value` attribute.
* When added to an <input> checkbox it binds to the `checked` attribute.

@bind-value:event directive and @bind-value directive
-----------------------------------------------------
@bind is just an overload of @bind-value, with the DOM event set to `onchange`.  So the following two commands are equivalent:

	@bind="userName"
	@bind-value="userName" @bind-value:event="onchange"

The DOM `onchange` event fires when you tab out of a textbox or hit the Enter key.

The event that causes the update can be changed via the @bind-value:event directive.  In that case the @bind-value directive is also needed.  

### Example

	@page "/"

	<h1>My favorite pizza is: @favPizza</h1>

	<p>
		Enter your favorite pizza:
		<input @bind-value="favPizza" @bind-value:event="oninput" />
	</p>

	@code {
		private string favPizza { get; set; } = "Margherita"
	}
	
In this case the update occurs on the `oninput` event, as soon as anything is entered into the textbox.

@bind:format directive
----------------------
Specifies the format for entering data.  

### Example
When entering a date:

	@page "/ukbirthdaypizza"

	<h1>Order a pizza for your birthday!</h1>

	<p>
		Enter your birth date:
		<input @bind="birthdate" @bind:format="dd-MM-yyyy" />
	</p>

	@code {
		private DateTime birthdate { get; set; } = new(2000, 1, 1);
	}

As at November 2022 @bind:format only works with date values.