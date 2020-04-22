<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы добавите Microsoft Graph в приложение. Для этого приложения вы будете использовать [клиентскую библиотеку Microsoft Graph для .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) , чтобы совершать вызовы в Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Получение событий календаря из Outlook

1. Откройте **календарпаже. XAML** в проекте **графтуториал** и замените его содержимое приведенным ниже кодом.

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

1. Откройте **CalendarPage.XAML.CS** и добавьте приведенные `using` ниже операторы в начало файла.

    ```csharp
    using Microsoft.Graph;
    using Newtonsoft.Json;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. Добавьте указанную ниже функцию в `CalendarPage` класс, чтобы получить события пользователя и ОТОБРАЗИТЬ ответ JSON.

    ```csharp
    protected override async void OnAppearing()
    {
        base.OnAppearing();

        // Get the events
        var events = await App.GraphClient.Me.Events.Request()
            .Select(e => new
            {
                e.Subject,
                e.Organizer,
                e.Start,
                e.End
            })
            .OrderBy("createdDateTime DESC")
            .GetAsync();

        // Temporary
        JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    }
    ```

    Рассмотрите, какие действия `OnAppearing` выполняет код.

    - URL-адрес, который будет вызываться — это `/v1.0/me/events`.
    - `Select` Функция ограничит поля, возвращаемые для каждого события, только теми, которые будут реально использоваться в представлении.
    - `OrderBy` Функция сортирует результаты по дате и времени создания, начиная с самого последнего элемента.

1. Запустите приложение и войдите в него, а затем щелкните элемент Навигация в **календаре** в меню. Вы должны увидеть дамп событий в календаре пользователя.

## <a name="display-the-results"></a>Отображение результатов

Теперь вы можете заменить дамп JSON на какой-то способ отобразить результаты в удобном для пользователя виде.

Сначала создайте [преобразователь значений привязки](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) , чтобы преобразовать значения [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) , возвращенные Microsoft Graph, в форматы даты и времени, которые ожидает пользователь.

1. Щелкните правой кнопкой мыши папку **Models** в проекте **графтуториал** и выберите команду **добавить**, а затем — **класс...**. Назовите класс `GraphDateTimeTimeZoneConverter` и нажмите кнопку **Добавить**.

1. Замените все содержимое файла на приведенный ниже код.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. Замените все содержимое файла **календарпаже. XAML** следующим кодом:

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml":::

    Вместо него `Editor` заменяется `ListView`. Объект `DataTemplate` , используемый для отображения каждого элемента, `GraphDateTimeTimeZoneConverter` использует для преобразования `Start` свойства `End` и свойства события в строку.

1. Откройте **CalendarPage.XAML.CS** и удалите из `OnAppearing` функции следующие строки.

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. Вместо этого добавьте следующий код.

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. Запустите приложение и войдите в систему, а затем щелкните элемент навигации по **календарю** . Вы должны увидеть список событий с отформатированными **начальным** и **конечным** значениями.

    ![Снимок экрана с таблицей событий](./images/calendar-page.png)
