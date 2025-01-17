Notes about Blazor: Routing and Navigation
==========================================
Simon Elms, 2 Dec 2022

@page directive
----------------
Specifies the route to the page.  Multiple @page directives are possible on the same page.  Example:

	@page "/Pizzas"
	@page "/CustomPizzas"

Router component
----------------
Sets up client-side routing.  It intercepts browser navigation and renders the page matching the requested address.

Example:

	<Router AppAssembly="@typeof(Program).Assembly">
		<Found Context="routeData">
			<RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
		</Found>
		<NotFound>
			<p>Sorry, we haven't found any pizzas here.</p>
		</NotFound>
	</Router>
	
* Included in auto-generated App.razor (generated via project template "blazorserver").

* The @page directive in each component sets a RouteAttribute.
	
* On startup:
	* Blazor checks Router AppAssembly attribute to determine which assembly to scan.
	* Blazor scans assembly, looking for components with RouteAttribute (ie @page directive), and assembles a RouteData object which specifies how requests are routed.

* <Found> tag: Passes the RouteData object to the RouteView component, along with parameters in URI or query string.  RouteView renders the requested component.
* DefaultLayout attribute: Used when requested component has no @layout directive.
* <NotFound> tag: When nothing matches requested URI.

NavigationManager
-----------------
Object, injected into components, that can access different parts of the URI, including:

* Full URI, eg http://www.contoso.com/pizzas/margherita?extratopping=pineapple
* Base URI, eg http://www.contoso.com/
* Base relative path, eg pizzas/margherita
* Query string, eg ?extratopping=pineapple.  Requires use of QueryHelpers class from Microsoft.AspNetCore.WebUtilities to parse the full URI.  

### Example

	@page "/pizzas"
	@using Microsoft.AspNetCore.WebUtilities
	@inject NavigationManager NavManager

	<h1>Buy a Pizza</h1>

	<p>I want to order a: @PizzaName</p>

	<p>I want to add this topping: @ToppingName</p>

	@code {
		[Parameter]
		public string PizzaName { get; set; }
		
		private string ToppingName { get; set; }
		
		protected override void OnInitialized()
		{
			StringValues extraTopping;
			var uri = NavManager.ToAbsoluteUri(NavManager.Uri);
			if (QueryHelpers.ParseQuery(uri.Query).TryGetValue("extratopping", out extraTopping))
			{
				ToppingName = System.Convert.ToString(extraTopping);
			}
		}
	}

Redirection via NavigationManager
---------------------------------	
NavigationManager can also redirect to another component via NavigationManager.NavigateTo(uri), where uri can be relative or absolute URI.  

### Example

	@page "/pizzas/{pizzaname}"
	@inject NavigationManager NavManager

	<h1>Buy a Pizza</h1>

	<p>I want to order a: @PizzaName</p>

	<button class="btn" @onclick="NavigateToPaymentPage">
		Buy this pizza!
	</button>

	@code {
		[Parameter]
		public string PizzaName { get; set; }
		
		private void NavigateToPaymentPage()
		{
			NavManager.NavigateTo("buypizza");
		}
	}
	
NavLink component
-----------------
Drop-in replacement for <a> tag.  Useful because it toggles an active CSS class when the link href matches the current URL, highlighting to the user which link is for the current page.

### Example

	@page "/pizzas"
	@inject NavigationManager NavManager

	<h1>Buy a Pizza</h1>

	<p>I want to order a: @PizzaName</p>

	<NavLink href=@HomePageUri Match="NavLinkMatch.All">Home Page</NavLink>

	@code {
		[Parameter]
		public string PizzaName { get; set; }
		
		public string HomePageURI { get; set; }
		
		protected override void OnInitialized()
		{
			HomePageURI = NavManager.BaseUri
		}
	}
	
* NavLink Match attribute: Determines the type of match required to highlight the link.  Two options:
	* NavLinkMatch.All: href must match entire current URL;
	* NavLinkMatch.Prefix: href must match start of current URL.  eg <NavLink href="pizzas" Match="NavLinkMatch.Prefix"> would match both http://www.contoso.com/pizzas and http://www.contoso.com/pizzas/formaggio
	
Route Parameters
----------------
For accessing different parts of the URI in a component.

* Route parameters are defined in @page directive inside curly braces.

* Case insensitive.

* Value is forwarded to a component parameter of the same name \(requires \[Parameter\] atttribute in the @code block\).

### Example

	@page "/FavoritePizzas/{favorite}"

	<h1>Choose a Pizza</h1>

	<p>Your favorite pizza is: @Favorite</p>

	@code {
		[Parameter]
		public string Favorite { get; set; }
	}
	
* Make a route parameter optional via trailing "?".  Set a default value in OnInitialized method or in OnParametersSet method.

### Example

	@page "/FavoritePizzas/{favorite?}"

	<h1>Choose a Pizza</h1>

	<p>Your favorite pizza is: @Favorite</p>

	@code {
		[Parameter]
		public string Favorite { get; set; }
		
		protected override void OnInitialized()
		{
			Favorite ??= "Fiorentina";
		}
	}
	
Catch-all route parameter
-------------------------
If the route parameter may include a forward slash then prefix it with a "*" to make it a catch-all route parameter.

### Example

	@page "/FavoritePizza/{*favorites}"

	<h1>Choose a Pizza</h1>

	<p>Your favorites pizza are: @Favorites</p>

	@code {
		[Parameter]
		public string Favorites { get; set; }
	}

Supplying URL http://www.contoso.com/favoritepizza/margherita/hawaiian results in the page displaying: 

"Your favorite pizzas are: margherita/hawaiian"
	
Route Constraints
-----------------
Can specify a type for the route parameter with trailing ":<type>".  This is known as a route constraint.  

### Example
	@page "/FavoritePizza/{preferredsize:int}"
	
* Types for route constraints: bool, datetime, decimal, double, float, guid, int, long

	
