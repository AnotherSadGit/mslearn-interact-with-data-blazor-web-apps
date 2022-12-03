Notes about Blazor: Events and Event Handling
=============================================
Simon Elms, 2 Dec 2022

General Notes
-------------
Handling events involves writing a C# event handler method, then binding it to the event via a Blazor directive.

Events can be the usual DOM events available to any web app, eg button click event.  Or they can be user-defined events.  DOM event directives match the name of the HTML event, eg @onclick, @onfocus.

For Blazor Server, event handlers run on the server, not in the client.  Compare with JavaScript, where event handlers run in the client.  Frequently occurring events such as @onmousemove may be better handled in JavaScript on the client, avoiding slowness due to the latency of frequent round trips to the server.  Note that changing the DOM on the client side via JavaScript may be problematic, because Blazor keeps a copy of the DOM on the server, so changes on the client side risk it getting out of sync with the server copy of the DOM.

Blazor allows event handlers to access static data, shared between sessions.

By default Blazor event handlers are synchronous.  Can make async.

By default events propagate up the DOM tree.  For example, if a button is within a div and they both have @onclick event handlers, clicking the button will fire the button's @onclick event handler then the div's @onclick event handler.

Steps for a basic event handler:
--------------------------------

1. Add event handler method.  An EventArgs parameter is optional, only needed if data needs to be passed into the event handler method.

2. Bind the event handler method to the event via a Blazor directive, specifying the name of the event handler method.  For DOM events there is no need to explicitly pass an EventArgs argument as part of the Blazor directive - it is passed implicitly to the event handler.

### Example

	@page "/counter"

	<h1>Counter</h1>

	<p>Current count: @currentCount</p>

	<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>


	@code {
	
		private int currentCount = 0;

		private void IncrementCount(MouseEventArgs e)
		{
			if (e.CtrlKey) // Ctrl key pressed as well
			{
				currentCount += 5;
			}
			else
			{
				currentCount++;
			}
		}
	}
	
Async event handlers
--------------------
To prevent blocking by long running procedures.

* Use async keyword.

* Return a Task.

* Use await in event handler method to run long-running tasks on separate threads, freeing the current thread for other work.  When the long-running task completes the event handler continues.

### Example 

	<button @onclick="DoWork">Run time-consuming operation</button>

	@code {
		private async Task DoWork()
		{
			// Call a method that takes a long time to run and free the current thread
			var data = await timeConsumingOperation();

			// Omitted for brevity
		}
	}
	
Set focus to a DOM element via an event
---------------------------------------
Only needed in unusual circumstances, eg directing the user to fix an error.  Otherwise it can be frustrating for the user, since forcing focus to a particular element make break their expectations or disrupt their workflow.

Set focus via FocusAsync method of an ElementReference object.  The ElementReference object references the item to which you want to set the focus.

* Create an ElementReference object in C# to represent the element that will receive the focus.

* In the event handler method in C# call the FocusAsync method of the ElementReference object.

* In the Blazor component code, add a @ref attribute to the element to set the focus to, specifying the name of the C# ElementReference object.

### Example

	<button class="btn btn-primary" @onclick="ChangeFocus">Click me to change focus</button>
	<input @ref=InputField @onfocus="HandleFocus" value="@data"/>

	@code {
		private ElementReference InputField;
		private string data;

		private async Task ChangeFocus()
		{
			await InputField.FocusAsync();
		}

		private async Task HandleFocus()
		{
			data = "Received focus";
		}
	}
	
Inline event handlers using lambda expressions
----------------------------------------------

### Example

	@page "/counter"

	<h1>Counter</h1>

	<p>Current count: @currentCount</p>

	<button class="btn btn-primary" @onclick="() => currentCount++">Click me</button>

	@code {
		private int currentCount = 0;
	}
	
Lambda expressions can be used to add extra arguments to a DOM event if the event handler requires more arguments than the DOM event supplies out of the box.

### Example

	@page "/counter"
	@inject IJSRuntime JS

	<h1>Counter</h1>

	<p id="currentCount">Current count: @currentCount</p>

	<button class="btn btn-primary" @onclick='mouseEvent => HandleClick(mouseEvent, "Hello")'>Click me</button>

	@code {
		private int currentCount = 0;

		private async Task HandleClick(MouseEventArgs e, string msg)
		{
			if (e.CtrlKey) // Ctrl key pressed as well
			{
				await JS.InvokeVoidAsync("alert", msg);
				currentCount += 5;
			}
			else
			{
				currentCount++;
			}
		}
	}

Overriding default actions for DOM events
-----------------------------------------
Via the preventDefault attribute of the event directive.

Some DOM events have default actions that fire when the event occurs, regardless of whether there is an explicit event handler.  eg @onkeypress for an <input> always displays the character corresponding to the key pressed.  Sometimes you may want to suppress the default action.

### Example
The following code displays an alert if an "@" key is pressed.  However, the "@" symbol is still displayed in the <input> textbox:

	<input value=@data @onkeypress="ProcessKeyPress"/>

	@code {
		private string data;

		private async Task ProcessKeyPress(KeyboardEventArgs e)
		{
			if (e.Key == "@")
			{
				await JS.InvokeVoidAsync("alert", "You pressed @");
			}
			else
			{
				data += e.Key.ToUpper();
			}
		}
	}
	
To prevent the "@" symbol from being displayed in the textbox, add the preventDefault attribute to the @onkeypress event directive:

	<input value=@data @onkeypress="ProcessKeyPress" @onkeypress:preventDefault />
	
The event handler will fire but the default action will not occur.

Preventing events propagating up the DOM tree
---------------------------------------------
Via the stopPropagation attribute of the event directive.

### Example
* Click me button has @onclick event handler IncrementCount;
* div has @onclick event handler HandleDivClick;
* Normally clicking the button will fire IncrementCount event handler followed by HandleDivClick event handler:

	<div @onclick="HandleDivClick">
		<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
		<input value=@data @onkeypress="ProcessKeyPress" @onkeypress:preventDefault />
	</div>

	@code {
		private async Task HandleDivClick()
		{
			await JS.InvokeVoidAsync("alert", "Div click");
		}

		private async Task ProcessKeyPress(KeyboardEventArgs e)
		{
			// Omitted for brevity
		}

		private int currentCount = 0;

		private void IncrementCount(MouseEventArgs e)
		{
			// Omitted for brevity
		}
	}
	
To prevent the HandleDivClick event handler firing when the button is clicked, add the stopPropagation attribute to the button @onclick directive:

	<button class="btn btn-primary" @onclick="IncrementCount" @onclick:stopPropagation>Click me</button>
	
Handling events across components
---------------------------------
For example, an event in a child component can fire an event handler in the parent component.

Via EventCallback, which works like a C# delegate.  

The child component C# code defines an EventCallback parameter representing the event.  It must be decorated with a [Parameter] attribute.

In the child component HTML code the event directive can point directly to the EventCallback parameter or it can point to a local event handler method which then calls the EventCallback. 

The parent component, when it references the child component, sets the EventCallback to the method that will be used as an event handler.  Note that the event handler method does not need to be defined in the parent component, it could be in another referenced component.

### Example: Event handlers in tutorial app
1. Add event handler methods (in this example OrderState CancelConfigurePizzaDialog and ConfirmConfigurePizzaDialog methods).

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
	
### Passing arguments to the EventCallback method
The method referenced by the EventCallback can take a single parameter, equivalent to an EventArgs parameter (although it can be any type of object, not just a subclass of EventArgs).  The type of the parameter is specified via a type parameter in the EventCallback declaration.  

#### Example
`public EventCallback<KeyTransformation> OnKeyPressCallback { get; set; }` indicates that the method referenced by the EventCallback will take a KeyTransformation object as an argument.

If the child component event directive points directly to the EventCallback then the EventCallback type parameter must specify the appropriate EventArgs that the event will supply.

#### Example

	<button @onclick="OnClickCallback">
		Click me!
	</button>

	@code {
		[Parameter]
		public EventCallback<MouseEventArgs> OnClickCallback { get; set; }
	}

