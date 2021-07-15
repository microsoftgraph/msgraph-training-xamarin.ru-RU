<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы будете включать Graph Microsoft в приложение. Для этого приложения вы будете использовать клиентскую библиотеку [Microsoft Graph для .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для звонков в Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Получение событий календаря из Outlook

1. Откройте **CalendarPage.xaml** в **проекте GraphTutorial** и замените его содержимое следующим.

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

    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. Добавьте в класс следующую функцию, чтобы получить начало текущей недели `CalendarPage` в часовом поясе пользователя.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml.cs" id="GetStartOfWeekSnippet":::

1. Добавьте в класс следующую функцию, чтобы получить события пользователя и `CalendarPage` отобразить ответ JSON.

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

        var timeZoneString =
            Xamarin.Forms.Device.RuntimePlatform == Xamarin.Forms.Device.UWP ?
                App.UserTimeZone.StandardName : App.UserTimeZone.DisplayName;

        // Get the events
        var events = await App.GraphClient.Me.CalendarView.Request(queryOptions)
            .Header("Prefer", $"outlook.timezone=\"{timeZoneString}\"")
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
        JSONResponse.Text = App.GraphClient.HttpProvider.Serializer.SerializeObject(events.CurrentPage);
    }
    ```

    Рассмотрим, что делает `OnAppearing` код.

    - Вызывается URL-адрес `/v1.0/me/calendarview`.
        - Параметры `startDateTime` `endDateTime` и параметров определяют начало и конец представления календаря.
        - Заготвка приводит к возвращению событий в часовом поясе `Prefer: outlook.timezone` `start` `end` пользователя.
        - Функция `Select` ограничивает поля, возвращаемые для каждого события, только теми, которые приложение будет фактически использовать.
        - Функция `OrderBy` сортировать результаты к дате начала и времени.
        - Функция `Top` запрашивает не более 50 событий.

1. Запустите приложение, войдите и нажмите элемент **навигации Календарь** в меню. Вы должны увидеть сброс JSON событий в календаре пользователя.

## <a name="display-the-results"></a>Отображение результатов

Теперь вы можете заменить свалку JSON чем-то, чтобы отобразить результаты в удобной для пользователя манере.

Начните с [](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) создания конвертера связывающего значения для преобразования значений [dateTimeTimeZone,](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) возвращенных Корпорацией Майкрософт Graph в форматы даты и времени, которые ожидает пользователь.

1. Щелкните правой кнопкой мыши папку **Модели** в **проекте GraphTutorial** и выберите **Добавить**, а затем **класс ...**. Назови класс `GraphDateTimeTimeZoneConverter` и выберите **Добавить**.

1. Замените все содержимое файла следующим.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. Замените все содержимое **CalendarPage.xaml** следующим.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml":::

    Это заменяет на `Editor` `ListView` . Используемый для отображения каждого элемента используется для преобразования и `DataTemplate` `GraphDateTimeTimeZoneConverter` свойств события в `Start` `End` строку.

1. Откройте **CalendarPage.xaml.cs** и удалите следующие строки из `OnAppearing` функции.

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. На их месте добавьте следующий код.

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. Запустите приложение, войдите и нажмите элемент **навигации Календарь.** Список событий с форматными значениями **"Начните"** и **"Конец".**

    ![Снимок экрана с таблицей событий](./images/calendar-page.png)
