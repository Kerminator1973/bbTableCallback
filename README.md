# "Saving the grid's sort state as part of GridSetting" feature

Проблема касается package Blazor Bootstrap, официальный портал которого находится [на странице](https://docs.blazorbootstrap.com/getting-started/blazor-webassembly-net-8).

Данное приложение иллюстрирует проблему выгрузки фактически отобранных данных из Grid-а по событию **GridSettingsChanged**. Это событие возникает, когда состояние Grid-а изменяется.

## Как было создано приложение

Основываясь на [подробной инструкции](https://github.com/vikramlearning/blazorbootstrap) по созданию проекта с использованием Blazor Bootstrap, был загружен шаблон проекта:

```shell
dotnet new install Blazor.Bootstrap.Templates::1.10.0
```

Далее, из среды Visual Studio было сгенерировано новое приложение на основании шаблона "Blazor Bootstrap - WebAssembly App (Vikram Reddy)".

Из приложения была удалена страница "Counter", а на странице "FetchData.razor" была добавлена верстка из [примера реализации страницы](https://docs.blazorbootstrap.com/components/grid#save-and-load-grid-settings).

Поскольку в примере не была определена реализация класс Employee1, он был разработан, основываясь на его использовании в коде:

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

Также была заменена default-ная реализация методов OnGridSettingsChanged() и GridSettingsProvider():

```csharp
@code {
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

Это позволяет не сохранять данные в локальное хранилище и исключить внедрения зависимости:

```csharp
[Inject] public IJSRuntime JS { get; set; }
```

## Подробное описание проблемы

В Grid существует событие **GridSettingsChanged**, параметром которого является объект **GridSettings**. В этом событии есть [ряд полезных полей](https://docs.blazorbootstrap.com/components/grid#gridsettings-properties), включая фильтры но нет ссылки на данные текущей выборки.

В соответствии с требованиями к разрабатываемому нами приложению, необходимо реализовать экспорт в Excel, на основании текущей выборки. Поскольку в событии нет ссылки на данные, реализовать это требование можно только используя "костыли" - извлечь фильтры, выполнить SQL-запрос на сервере и переслать результат в клиентское Blazor-приложение для генерации Excel-файла. С точки зрения долговременого сопровождения решения, хотелось бы исключить разработку "костылей".

В официальной документации по продукту есть следующая информация: "_IMPORTANT! Saving the Grid's sorting state as part of GridSettings is not yet supported. This functionality will be included in future releases_".
