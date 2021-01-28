<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы включаете Microsoft Graph в приложение. Для этого приложения вы будете использовать клиентскую библиотеку Microsoft Graph для [.NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для вызова Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Получить события календаря из Outlook

1. Откройте **CalendarPage.xaml** в **проекте GraphTutorial** и замените его содержимое на следующее.

    ```xaml
    <?xml version="1.0" encoding="utf-8" ?>
    <ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 Title="Calendar"
                 x:Class="GraphTutorial.CalendarPage">
        <ContentPage.Content>
            <StackLayout>
                <Editor x:Name="JSONResponse"
                        IsSpellCheckEnabled="False"
                        HorizontalOptions="FillAndExpand"
                        VerticalOptions="FillAndExpand"/>
            </StackLayout>
        </ContentPage.Content>
    </ContentPage>
    ```

1. Откройте **CalendarPage.xaml.cs** и добавьте следующие утверждения в `using` верхней части файла.

    ```csharp
    using Microsoft.Graph;
    using Newtonsoft.Json;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. Добавьте в класс следующую функцию, чтобы получить начало текущей недели `CalendarPage` в часовом поясе пользователя.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml.cs" id="GetStartOfWeekSnippet":::

1. Добавьте в класс следующую функцию, чтобы получить события пользователя и отобразить `CalendarPage` ответ JSON.

    ```csharp
    protected override async void OnAppearing()
    {
        base.OnAppearing();

        // Get start and end of week in user's time zone
        var startOfWeek = GetUtcStartOfWeekInTimeZone(DateTime.Today, App.UserTimeZone);
        var endOfWeek = startOfWeek.AddDays(7);

        var queryOptions = new List<QueryOption>
        {
            new QueryOption("startDateTime", startOfWeek.ToString("o")),
            new QueryOption("endDateTime", endOfWeek.ToString("o"))
        };

        // Get the events
        var events = await App.GraphClient.Me.CalendarView.Request(queryOptions)
            .Header("Prefer", $"outlook.timezone=\"{App.UserTimeZone.DisplayName}\"")
            .Select(e => new
            {
                e.Subject,
                e.Organizer,
                e.Start,
                e.End
            })
            .OrderBy("start/DateTime")
            .Top(50)
            .GetAsync();

        // Temporary
        JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    }
    ```

    Подумайте, что делает `OnAppearing` код.

    - Будет вызван URL-адрес `/v1.0/me/calendarview` .
        - Параметры `startDateTime` `endDateTime` и параметров определяют начало и конец представления календаря.
        - Заглавная часть приводит к возвращению событий в часовом поясе `Prefer: outlook.timezone` `start` `end` пользователя.
        - Функция `Select` ограничивает поля, возвращаемые для каждого события, только теми, которые будут фактически использовать приложение.
        - Функция `OrderBy` сортировать результаты по дате и времени начала.
        - Функция `Top` запрашивает не более 50 событий.

1. Запустите приложение, войдите в  систему и щелкните элемент навигации "Календарь" в меню. Вы увидите дамп событий в JSON в календаре пользователя.

## <a name="display-the-results"></a>Отображение результатов

Теперь вы можете заменить дамп JSON на что-то, чтобы отобразить результаты в удобном для пользователя виде.

Для начала [](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) создайте конвертер значений привязки, чтобы преобразовать значения [dateTimeTimeZone,](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) возвращенные Microsoft Graph, в форматы даты и времени, которые ожидает пользователь.

1. Щелкните правой **кнопкой мыши** папку Models в проекте **GraphTutorial** и выберите **"Добавить",** затем **"Класс"...** Назовем класс `GraphDateTimeTimeZoneConverter` и выберите **"Добавить".**

1. Замените все содержимое файла на следующее:

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. Замените все содержимое **CalendarPage.xaml** следующим:

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml" id="CalendarPageXamlSnippet":::

    Он заменяется `Editor` на `ListView` .. Для отображения каждого элемента используется преобразование свойств события `DataTemplate` `GraphDateTimeTimeZoneConverter` в `Start` `End` строку.

1. Откройте **CalendarPage.xaml.cs** и удалите из функции следующие `OnAppearing` строки.

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. На их месте добавьте следующий код.

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. Запустите приложение, войдите в систему и щелкните элемент **навигации календаря.** Должен отформатирован список событий со значениями **start** и **End.**

    ![Снимок экрана с таблицей событий](./images/calendar-page.png)
