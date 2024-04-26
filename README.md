# "Saving the grid's sort state as part of GridSetting" feature

The feature "Saving the grid's sorting state as part of GridSetting" related to [Blazor Bootstrap](https://docs.blazorbootstrap.com/getting-started/blazor-webassembly-net-8). It hasn't been developed yet at the end of April, 2024.

It is expected that this feature will allow using data selected in the Grid, to export them to other applications (i.e. clipboard or Microsoft Excel).

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
