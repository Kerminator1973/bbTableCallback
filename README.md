# "Saving the grid's sort state as part of GridSetting" feature

The **sorting state of the grid** is the representation of the data after the filtering and sorting rules have been accepted.

We need to save the sort state of the grid in several cases, such as when we want to keep user input filters persistent. Another case - we want access to the filtered data in order to export it.

The feature "Saving the grid's sorting state as part of GridSetting" related to [Blazor Bootstrap](https://docs.blazorbootstrap.com/getting-started/blazor-webassembly-net-8). It hasn't been developed yet at the end of April, 2024.

It is expected that this feature will allow the use of data that is filtered and displayed in the grid component for export to other applications (i.e. clipboard or Microsoft Excel).

This repository contains a minimal functionality application to illustrate the expected implementation of the **GridSettingsChanged** event. The event is called when Grid's sorting state is changed.

## How the demo application was developed

Following [the detailed instructions](https://github.com/vikramlearning/blazorbootstrap), the project templates were installed first:

```shell
dotnet new install Blazor.Bootstrap.Templates::1.10.0
```

Next, the template "Blazor Bootstrap - WebAssembly App (Vikram Reddy)" was used to generate boilerplate code.

The redundant page "Counter" has been removed from the app. The Razor code has been copied from [the example](https://docs.blazorbootstrap.com/components/grid#save-and-load-grid-settings) to the FetchData.razor page.

Unfortunately, the example doesn't have an implementation of the Employee1 class. So it was reconstructed:

```csharp
@code {
    public class Employee1
    {
        public int Id { get; set; }
        public string? Name { get; set; }
        public string? Designation { get; set; }
        public DateOnly DOJ { get; set; }
        public bool IsActive { get; set; }
    }
}
```

The implementation of the OnGridSettingsChanged() and GridSettingsProvider() methods has been simplified a bit:


```csharp
@code {
    public GridSettings Settings { get; set; }

    private async Task OnGridSettingsChanged(GridSettings settings)
    {
        Settings = settings;
    }

    private async Task<GridSettings> GridSettingsProvider()
    {
        return Settings;
    }
}
```

This made it possible to not store data on local storage and eliminate dependency injection.

```csharp
[Inject] public IJSRuntime JS { get; set; }
```

The main idea is to keep a reference to GridSettings in the **Settings** class member. When a user presses a certain button, for instance - "Export to Excel", we can use the Settings to access the last selected data.

## Detailed description of the problem

There is a **GridSettingsChanged** event that can be triggered when the Grid state has changed. The event has a parameter named **GridSettings** that contains [a number of useful fields](https://docs.blazorbootstrap.com/components/grid#gridsettings-properties), including filters, but no reference to the current selection data.

In accordance with the requirements for the application we are developing, it is necessary to implement export to Excel based on the current selection. Since the event does not contain a reference to the current selected data, this requirement can only be implemented using “crutches” - for instance:

- extract filters
- execute an SQL query on the server
- send the result to the Blazor client application to generate an Excel file

Regarding the long-term support of the solution, I would like to exclude any "crutches".

The official product documentation contains the following information: "_IMPORTANT! Saving the Grid's sorting state as part of GridSettings is not yet supported. This functionality will be included in future releases_".

## How to debug the Blazor.Bootstrap source code

Download the source files from the GitHub repository at [vikramlearning/blazorbootstrap](https://github.com/vikramlearning/blazorbootstrap)

There are multiple projects in the solution. The target project is blazorbootstrap. Before building the library you should choose the target platform. It is .NET 6 by default. It's important to note, that the default platform for Blazor.Bootstrap.Templates::1.10.0 is .NET 7.

You will have no problems building the library and successfully compiling it, you can find the library in the folder: \blazorbootstrap\bin\Debug\net6.0\BlazorBootstrap.dll

The next step is to add the library to your application project as follows:

- Select your application project in the Solution Tree
- In the context menu select "Add -> Project Reference..."
- In the modal dialog find the "Browse..." button
- Locate the "BlazorBootstrap.dll" library on your local disk

It's not enough to add the library as a dependency. You have to copy files from the "wwwroot" folder of the Blazor.Bootstrap to the corresponding folder of your project: "\[yourprojectname]\wwwroot\_content\Blazor.Bootstrap".

Build your project. Now you can set breakpoints in your code and drill down to the BlazorBootstrap sources. At this point you can easily debug BlazorBootstrap with your application code.

However, there are some glitches in the layout. In particular, the part of the layout related to screen resolution is broken. To get to the different application screens, you have to click on the "expand available pages" button, which should only be displayed at the minimum screen width (on a mobile device with portrait orientation).

Unfortunately, changing the platform from .NET 6 to .NET 8 (including all dependencies on ASP.NET Core) doesn't solve the problem.
