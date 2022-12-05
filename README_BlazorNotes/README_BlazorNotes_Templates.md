Notes about Blazor: Templates
=============================
Simon Elms, 5 Dec 2022

Template components can be shared between multiple applications.

They can be applied to all pages in an application.

They are useful for corporate branding, to apply corporation colours and styles across all web applications.

Child page content is injected into RenderFragement object
----------------------------------------------------------
A template wraps the content of the child page.  Page content is injected into a RenderFragement object in the template.

ChildContent is the default name of the RenderFragement object.  If a different name is used then the name must be specified in the child component that uses the template.  

### Example
The following is a simple template component wich adds the company name as a page header and a copyright message as a page footer:

	<h1>Contoso Ltd</h1>
	<h3>Heading</h3>

	@ChildContent

	<p></p>
	<p style="font-style:italic; color:red;">(c) 2021</p>
	@code {
	  [Parameter]
	  public RenderFragment ChildContent { get; set; }
	}
		
Using the template component
----------------------------
Assuming the template component in the example above is called HeadingComponent.razor, in the component that will use the template, wrap the page markup in a <HeadingComponent> element.  The markup within that element will be passed to the template in the ChildContent parameter. 

### Example 

	@page "/testpage"

	<HeadingComponent>
		<p>This text will be rendered by the template.</p>
		<p>So will this list:
		<ol>
			<li>Item 1</li>
			<li>Item 2</li>
			<li>Item 3</li>
		</ol>
		</p> 
	</HeadingComponent>
	
### Example of a different name for the RenderFragement object
If the parameter in the template were called BodyContent instead of ChildContent, a BodyContent element would need to be included in the child component:

	@page "/testpage"

	<HeadingComponent>
		<BodyContent>
			<p>This text will be rendered by the template.</p>
			<p>So will this list:
			<ol>
				<li>Item 1</li>
				<li>Item 2</li>
				<li>Item 3</li>
			</ol>
			</p> 
		</BodyContent>
	</HeadingComponent>
	
Multiple RenderFragment objects in a template
----------------------------------------------
A template can support multiple RenderFragment objects.  Each will need a different name.

### Example
Template component:

	<h1>Contoso Ltd</h1>
	<h3>@HeaderContent</h3>

	@BodyContent
	<p></p>
	<p style="font-style:italic; color:red;">(c) 2021</p>

	@code {
	  [Parameter]
	  public RenderFragment HeaderContent { get; set; }

	  [Parameter]
	  public RenderFragment BodyContent { get; set; }
	}
	
Using the different RenderFragment objects in a child component (assuming the template is still called HeadingComponent):

	@page "/testpage"

	<HeadingComponent>
		<HeaderContent>
			This is the test page heading
		</HeaderContent>
		<BodyContent>
			<p>This text will be rendered by the template.</p>
			<p>So will this list:
			<ol>
				<li>Item 1</li>
				<li>Item 2</li>
				<li>Item 3</li>
			</ol>
			</p> 
		</BodyContent>
	</HeadingComponent>

Generic RenderFragment with type parameter
------------------------------------------
The non-generic RenderFragment renders HTML.  Providing a type parameter allows RenderFragment to render other content.  

The type parameter must be specified via a @typeparam directive.  Multiple @typeparam directives are allowed.

The template automatically creates a @context object, of the type specified via the type parameter.  This @context object can be accessed in the child component.  The properties of the @context object that can be accessed in the child component will depend on the type specified via the type parameter.

### Example
Template component:

	@typeparam TItem
	
	@foreach(TItem item in Data ?? Array.Empty<TItem>())
	{
		<p>@ChildContent(item)</p>
	}


	@code {
	  [Parameter]
	  public IEnumerable<TItem> Data { get; set; }

	  [Parameter]
	  public RenderFragment<TItem> ChildContent { get; set; }
	}
	
Child component that uses the template component, assuming the template is called ObjectDataList.razor:

	@page "/testpage"
	@using WebApplication.Data
	@inject WeatherForecastService ForecastService

	<ObjectDataList Data=@forecasts>
		<h4>Date: @context.Date</h4>
		<p>Temperature (C): @context.TemperatureC</p>
		<p>Temperature (F): @context.TemperatureF</p>
		<p>Summary: @context.Summary</p>
	</ObjectDataList>

	@code {
		private WeatherForecast[] forecasts;

		protected override async Task OnInitializedAsync()
		{
			await base.OnInitializedAsync();
			forecasts ??= await ForecastService.GetForecastAsync(DateTime.Now);
		}
	}
	
In this case the @context is a WeatherForecast object, so the child component can access WeatherForecast properties via @context.

Nested templates
----------------
Are possible.

Use `<CascadingValue Value="this">` in the parent template to allow the child templates to get data from the parent template.

### Example
Creating tab pages with vertical tabs on the left.  Using two templates:
* VerticalTab: Container for the tabs and the current page being displayed;
* TabPage: Renders the current page in the VerticalTab template.

VerticalTab.razor:

	<CascadingValue Value="this">
		<div class="row align-items-start">
			<div class="btn-group-vertical">
				@foreach (TabPage tabPage in Items)
				{
					<button type="button" class="btn @GetButtonClass(tabPage) text-left" style="margin-bottom: 10px;" @onclick=@(() => ActivateItem(tabPage) )>    
						@tabPage.Title
					</button>
				 }
			</div>
			<div class="card-body" style="background-color:#EBEBEB;">
				@ChildContent
			</div>
		</div>
	</CascadingValue>

	@code {
		[Parameter]
		public RenderFragment ChildContent { get; set; }
		
		public TabPage CurrentItem { get; set; }
		private List<TabPage> Items = new List<TabPage>();

		internal void AddItem(TabPage tabPage)
		{
			Items.Add(tabPage);
			if (Items.Count == 1)
			{
				CurrentItem = tabPage;
			}
			StateHasChanged();
		}

		string GetButtonClass(TabPage tabPage) => tabPage == CurrentItem ? "btn-primary" : "btn-secondary";

		void ActivateItem(TabPage tabPage) => CurrentItem = tabPage;
	}
	
TabPage.razor:

	@if (Container.CurrentItem == this)
	{
		@ChildContent
	}

	@code {
		[CascadingParameter]
		private VerticalTab Container { get; set; }

		[Parameter]
		public RenderFragment ChildContent { get; set; }

		[Parameter]
		public string Title { get; set; }
	  
		protected override void OnInitialized()
		{
			if (Container is null)
			{
				throw new ArgumentNullException(nameof(Container), "Cannot create a tab page without a vertical tab container");
			}
			base.OnInitialized();
			Container.AddItem(this);
		}
	}
	
The cascading parameter defined in VerticalTab represents the VerticalTab object.  It is injected into TabPage as the Container parameter.  As TabPage is initialized it checks to see if it has a parent VerticalTab and, if not, throws an exception.  If it does have a parent VerticalTab then it adds itself to the VerticalTab Items collection.  If the currently selected item in VerticalTab matches this TabPage then it will render its child contents.

Using the templates in a child component:

	@page "/testpage"

	<VerticalTab>
		<TabPage Title="Tab 1">
			<h1>Content for tab 1</h1>
			<p>Text</p>
			<p>Text</p>
			<p>Text</p>
		</TabPage>
		<TabPage Title="Tab 2">
			<h1>Content for tab 2</h1>
			<p>Rhubarb</p>
			<p>Custard</p>
		</TabPage>
			<TabPage Title="Tab 3">
			<h1>Content for tab 1</h1>
			<p>Cat</p>
			<p>Dog</p>
		</TabPage>
	</VerticalTab>
	
It includes a <VerticalTab> element which contains multiple <TabPage> elements.