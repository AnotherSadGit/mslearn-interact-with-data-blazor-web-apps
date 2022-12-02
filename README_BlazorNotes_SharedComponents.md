Notes about Blazor: Shared Components
=====================================
Simon Elms, 2 Dec 2022

Shared components
-----------------
Razor pages which can be called from other Razor pages.

The parent page incorporates a child component via a tag matching the child component page name.

#### Example
Shared component: ConfigurePizzaDialog.razor
  
Parent page calling shared component: Index.razor
Calls the shared component via a tag matching the child component page name:

	@if (showingConfigureDialog)
	{
		<ConfigurePizzaDialog Pizza="configuringPizza" />
	}
	
Layout components (AKA Layouts)
-------------------------------
Component that shares its rendered markup with components that reference it.  

For common UI elements like banners, branding, nav menus, footers.

Conventions: 

* Stored in Shared folder

* For projects created from a Blazor project template, Shared/MainLayout.razor is the default layout.

Works the opposite way to shared components:

* Shared components: Shared component content is inserted into the referencing component's content.

* Layout components: Referencing component's content is inserted into layout component content where the @body directive is.

Layout component must:

* Inherit LayoutComponentBase

* Include a @body directive

* Not include a @page directive since they're shared components.  The referencing componment must have the @page directive.

The referencing component references a layout component via a @layout directive at the top of the page. 

### Example

#### Layout component:

	@inherits LayoutComponentBase

	<header>
		<h1>Blazing Pizza</h1>
	</header>

	<nav>
		<a href="Pizzas">Browse Pizzas</a>
		<a href="Toppings">Browse Extra Toppings</a>
		<a href="FavoritePizzas">Tell us your favorite</a>
		<a href="Orders">Track Your Order</a>
	</nav>

	@Body

	<footer>
		@new MarkdownString(TrademarkMessage)
	</footer>

	@code {
		public string TrademarkMessage { get; set; } = "All content is &copy; Blazing Pizzas 2021";
	}
	
#### Referencing component:

	@page "/FavoritePizzas/{favorite}"
	@layout BlazingPizzasMainLayout

	<h1>Choose a Pizza</h1>

	<p>Your favorite pizza is: @Favorite</p>

	@code {
		[Parameter]
		public string Favorite { get; set; }
	}
	
Default Layouts
----------------

### _Imports.razor
Layout component that is applied to all Blazor components automatically, without needing to add a @layout directive to each referencing component.

* Only applies to components in the same folder as the _Imports.razor, or to sub-folders of that folder.

* DANGER: Do NOT add a @layout directive to the _Imports.razor file in the project root, as that results in an infinite loop.

### Via Router component RouteView > DefaultLayout attribute
Will apply the default layout to every component in every folder of the web app.

In App.razor component.

Can also apply the layout to the 404 page specified under <NotFound>, except this uses the LayoutView > Layout attribute, not RouteView > DefaultLayout.

It appears the leading "@" for @typeof and @routeData are optional.  The example below comes from the tutorial web app and some @typeof have the leading "@" but others don't.

Overridden by:
	* Layout in _Imports.razor, where it applies
	* @layout directive in a component
	
#### Example

	<Router AppAssembly="@typeof(Program).Assembly" PreferExactMatches="@true">
		<Found Context="routeData">
			<RouteView RouteData="@routeData" DefaultLayout="typeof(MainLayout)"/>
		</Found>
		<NotFound>
			<LayoutView Layout="typeof(MainLayout)">
				<p>Sorry, there's nothing at this address.</p>
			</LayoutView>
		</NotFound>
	</Router>