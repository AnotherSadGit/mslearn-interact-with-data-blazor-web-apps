Notes about Blazor: Events and Event Handling
=============================================
Simon Elms, 2 Dec 2022

Steps:
------

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
