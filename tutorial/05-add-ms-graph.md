<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c59fc-101">В этом упражнении вы включаете Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="c59fc-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="c59fc-102">Для этого приложения вы будете использовать клиентскую библиотеку Microsoft Graph для [.NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="c59fc-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="c59fc-103">Получить события календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="c59fc-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="c59fc-104">Откройте **CalendarPage.xaml** в **проекте GraphTutorial** и замените его содержимое на следующее.</span><span class="sxs-lookup"><span data-stu-id="c59fc-104">Open **CalendarPage.xaml** in the **GraphTutorial** project and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="c59fc-105">Откройте **CalendarPage.xaml.cs** и добавьте следующие утверждения в `using` верхней части файла.</span><span class="sxs-lookup"><span data-stu-id="c59fc-105">Open **CalendarPage.xaml.cs** and add the following `using` statements at the top of the file.</span></span>

    ```csharp
    using Microsoft.Graph;
    using Newtonsoft.Json;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. <span data-ttu-id="c59fc-106">Добавьте в класс следующую функцию, чтобы получить начало текущей недели `CalendarPage` в часовом поясе пользователя.</span><span class="sxs-lookup"><span data-stu-id="c59fc-106">Add the following function to the `CalendarPage` class to get the start of the current week in the user's time zone.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml.cs" id="GetStartOfWeekSnippet":::

1. <span data-ttu-id="c59fc-107">Добавьте в класс следующую функцию, чтобы получить события пользователя и отобразить `CalendarPage` ответ JSON.</span><span class="sxs-lookup"><span data-stu-id="c59fc-107">Add the following function to the `CalendarPage` class to get the user's events and display the JSON response.</span></span>

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

    <span data-ttu-id="c59fc-108">Подумайте, что делает `OnAppearing` код.</span><span class="sxs-lookup"><span data-stu-id="c59fc-108">Consider what the code in `OnAppearing` is doing.</span></span>

    - <span data-ttu-id="c59fc-109">Будет вызван URL-адрес `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="c59fc-109">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="c59fc-110">Параметры `startDateTime` `endDateTime` и параметров определяют начало и конец представления календаря.</span><span class="sxs-lookup"><span data-stu-id="c59fc-110">The `startDateTime` and `endDateTime` parameters define the start and end of the calendar view.</span></span>
        - <span data-ttu-id="c59fc-111">Заглавная часть приводит к возвращению событий в часовом поясе `Prefer: outlook.timezone` `start` `end` пользователя.</span><span class="sxs-lookup"><span data-stu-id="c59fc-111">The `Prefer: outlook.timezone` header causes the `start` and `end` of the events to be returned in the user's time zone.</span></span>
        - <span data-ttu-id="c59fc-112">Функция `Select` ограничивает поля, возвращаемые для каждого события, только теми, которые будут фактически использовать приложение.</span><span class="sxs-lookup"><span data-stu-id="c59fc-112">The `Select` function limits the fields returned for each event to just those the app will actually use.</span></span>
        - <span data-ttu-id="c59fc-113">Функция `OrderBy` сортировать результаты по дате и времени начала.</span><span class="sxs-lookup"><span data-stu-id="c59fc-113">The `OrderBy` function sorts the results by the start date and time.</span></span>
        - <span data-ttu-id="c59fc-114">Функция `Top` запрашивает не более 50 событий.</span><span class="sxs-lookup"><span data-stu-id="c59fc-114">The `Top` function requests at most 50 events.</span></span>

1. <span data-ttu-id="c59fc-115">Запустите приложение, войдите в  систему и щелкните элемент навигации "Календарь" в меню.</span><span class="sxs-lookup"><span data-stu-id="c59fc-115">Run the app, sign in, and click the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="c59fc-116">Вы увидите дамп событий в JSON в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="c59fc-116">You should see a JSON dump of the events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="c59fc-117">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="c59fc-117">Display the results</span></span>

<span data-ttu-id="c59fc-118">Теперь вы можете заменить дамп JSON на что-то, чтобы отобразить результаты в удобном для пользователя виде.</span><span class="sxs-lookup"><span data-stu-id="c59fc-118">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span>

<span data-ttu-id="c59fc-119">Для начала [](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) создайте конвертер значений привязки, чтобы преобразовать значения [dateTimeTimeZone,](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) возвращенные Microsoft Graph, в форматы даты и времени, которые ожидает пользователь.</span><span class="sxs-lookup"><span data-stu-id="c59fc-119">Start by creating a [binding value converter](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) to convert the [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) values returned by Microsoft Graph into the date and time formats the user expects.</span></span>

1. <span data-ttu-id="c59fc-120">Щелкните правой **кнопкой мыши** папку Models в проекте **GraphTutorial** и выберите **"Добавить",** затем **"Класс"...** Назовем класс `GraphDateTimeTimeZoneConverter` и выберите **"Добавить".**</span><span class="sxs-lookup"><span data-stu-id="c59fc-120">Right-click the **Models** folder in the **GraphTutorial** project and select **Add**, then **Class...**. Name the class `GraphDateTimeTimeZoneConverter` and select **Add**.</span></span>

1. <span data-ttu-id="c59fc-121">Замените все содержимое файла на следующее:</span><span class="sxs-lookup"><span data-stu-id="c59fc-121">Replace the entire contents of the file with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. <span data-ttu-id="c59fc-122">Замените все содержимое **CalendarPage.xaml** следующим:</span><span class="sxs-lookup"><span data-stu-id="c59fc-122">Replace the entire contents of **CalendarPage.xaml** with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml" id="CalendarPageXamlSnippet":::

    <span data-ttu-id="c59fc-123">Он заменяется `Editor` на `ListView` ..</span><span class="sxs-lookup"><span data-stu-id="c59fc-123">This replaces the `Editor` with a `ListView`.</span></span> <span data-ttu-id="c59fc-124">Для отображения каждого элемента используется преобразование свойств события `DataTemplate` `GraphDateTimeTimeZoneConverter` в `Start` `End` строку.</span><span class="sxs-lookup"><span data-stu-id="c59fc-124">The `DataTemplate` used to render each item uses the `GraphDateTimeTimeZoneConverter` to convert the `Start` and `End` properties of the event to a string.</span></span>

1. <span data-ttu-id="c59fc-125">Откройте **CalendarPage.xaml.cs** и удалите из функции следующие `OnAppearing` строки.</span><span class="sxs-lookup"><span data-stu-id="c59fc-125">Open **CalendarPage.xaml.cs** and remove the following lines from the `OnAppearing` function.</span></span>

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. <span data-ttu-id="c59fc-126">На их месте добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="c59fc-126">In their place, add the following code.</span></span>

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. <span data-ttu-id="c59fc-127">Запустите приложение, войдите в систему и щелкните элемент **навигации календаря.**</span><span class="sxs-lookup"><span data-stu-id="c59fc-127">Run the app, sign in, and click the **Calendar** navigation item.</span></span> <span data-ttu-id="c59fc-128">Должен отформатирован список событий со значениями **start** и **End.**</span><span class="sxs-lookup"><span data-stu-id="c59fc-128">You should see the list of events with the **Start** and **End** values formatted.</span></span>

    ![Снимок экрана с таблицей событий](./images/calendar-page.png)
