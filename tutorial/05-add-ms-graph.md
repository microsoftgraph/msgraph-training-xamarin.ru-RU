<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="79aa7-101">В этом упражнении вы будете включать Graph Microsoft в приложение.</span><span class="sxs-lookup"><span data-stu-id="79aa7-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="79aa7-102">Для этого приложения вы будете использовать клиентскую библиотеку [Microsoft Graph для .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для звонков в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="79aa7-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="79aa7-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="79aa7-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="79aa7-104">Откройте **CalendarPage.xaml** в **проекте GraphTutorial** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="79aa7-104">Open **CalendarPage.xaml** in the **GraphTutorial** project and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="79aa7-105">Откройте **CalendarPage.xaml.cs** и добавьте следующие утверждения в `using` верхней части файла.</span><span class="sxs-lookup"><span data-stu-id="79aa7-105">Open **CalendarPage.xaml.cs** and add the following `using` statements at the top of the file.</span></span>

    ```csharp
    using Microsoft.Graph;

    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. <span data-ttu-id="79aa7-106">Добавьте в класс следующую функцию, чтобы получить начало текущей недели `CalendarPage` в часовом поясе пользователя.</span><span class="sxs-lookup"><span data-stu-id="79aa7-106">Add the following function to the `CalendarPage` class to get the start of the current week in the user's time zone.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml.cs" id="GetStartOfWeekSnippet":::

1. <span data-ttu-id="79aa7-107">Добавьте в класс следующую функцию, чтобы получить события пользователя и `CalendarPage` отобразить ответ JSON.</span><span class="sxs-lookup"><span data-stu-id="79aa7-107">Add the following function to the `CalendarPage` class to get the user's events and display the JSON response.</span></span>

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

    <span data-ttu-id="79aa7-108">Рассмотрим, что делает `OnAppearing` код.</span><span class="sxs-lookup"><span data-stu-id="79aa7-108">Consider what the code in `OnAppearing` is doing.</span></span>

    - <span data-ttu-id="79aa7-109">Вызывается URL-адрес `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="79aa7-109">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="79aa7-110">Параметры `startDateTime` `endDateTime` и параметров определяют начало и конец представления календаря.</span><span class="sxs-lookup"><span data-stu-id="79aa7-110">The `startDateTime` and `endDateTime` parameters define the start and end of the calendar view.</span></span>
        - <span data-ttu-id="79aa7-111">Заготвка приводит к возвращению событий в часовом поясе `Prefer: outlook.timezone` `start` `end` пользователя.</span><span class="sxs-lookup"><span data-stu-id="79aa7-111">The `Prefer: outlook.timezone` header causes the `start` and `end` of the events to be returned in the user's time zone.</span></span>
        - <span data-ttu-id="79aa7-112">Функция `Select` ограничивает поля, возвращаемые для каждого события, только теми, которые приложение будет фактически использовать.</span><span class="sxs-lookup"><span data-stu-id="79aa7-112">The `Select` function limits the fields returned for each event to just those the app will actually use.</span></span>
        - <span data-ttu-id="79aa7-113">Функция `OrderBy` сортировать результаты к дате начала и времени.</span><span class="sxs-lookup"><span data-stu-id="79aa7-113">The `OrderBy` function sorts the results by the start date and time.</span></span>
        - <span data-ttu-id="79aa7-114">Функция `Top` запрашивает не более 50 событий.</span><span class="sxs-lookup"><span data-stu-id="79aa7-114">The `Top` function requests at most 50 events.</span></span>

1. <span data-ttu-id="79aa7-115">Запустите приложение, войдите и нажмите элемент **навигации Календарь** в меню.</span><span class="sxs-lookup"><span data-stu-id="79aa7-115">Run the app, sign in, and click the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="79aa7-116">Вы должны увидеть сброс JSON событий в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="79aa7-116">You should see a JSON dump of the events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="79aa7-117">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="79aa7-117">Display the results</span></span>

<span data-ttu-id="79aa7-118">Теперь вы можете заменить свалку JSON чем-то, чтобы отобразить результаты в удобной для пользователя манере.</span><span class="sxs-lookup"><span data-stu-id="79aa7-118">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span>

<span data-ttu-id="79aa7-119">Начните с [](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) создания конвертера связывающего значения для преобразования значений [dateTimeTimeZone,](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) возвращенных Корпорацией Майкрософт Graph в форматы даты и времени, которые ожидает пользователь.</span><span class="sxs-lookup"><span data-stu-id="79aa7-119">Start by creating a [binding value converter](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) to convert the [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) values returned by Microsoft Graph into the date and time formats the user expects.</span></span>

1. <span data-ttu-id="79aa7-120">Щелкните правой кнопкой мыши папку **Модели** в **проекте GraphTutorial** и выберите **Добавить**, а затем **класс ...**. Назови класс `GraphDateTimeTimeZoneConverter` и выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="79aa7-120">Right-click the **Models** folder in the **GraphTutorial** project and select **Add**, then **Class...**. Name the class `GraphDateTimeTimeZoneConverter` and select **Add**.</span></span>

1. <span data-ttu-id="79aa7-121">Замените все содержимое файла следующим.</span><span class="sxs-lookup"><span data-stu-id="79aa7-121">Replace the entire contents of the file with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. <span data-ttu-id="79aa7-122">Замените все содержимое **CalendarPage.xaml** следующим.</span><span class="sxs-lookup"><span data-stu-id="79aa7-122">Replace the entire contents of **CalendarPage.xaml** with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml":::

    <span data-ttu-id="79aa7-123">Это заменяет на `Editor` `ListView` .</span><span class="sxs-lookup"><span data-stu-id="79aa7-123">This replaces the `Editor` with a `ListView`.</span></span> <span data-ttu-id="79aa7-124">Используемый для отображения каждого элемента используется для преобразования и `DataTemplate` `GraphDateTimeTimeZoneConverter` свойств события в `Start` `End` строку.</span><span class="sxs-lookup"><span data-stu-id="79aa7-124">The `DataTemplate` used to render each item uses the `GraphDateTimeTimeZoneConverter` to convert the `Start` and `End` properties of the event to a string.</span></span>

1. <span data-ttu-id="79aa7-125">Откройте **CalendarPage.xaml.cs** и удалите следующие строки из `OnAppearing` функции.</span><span class="sxs-lookup"><span data-stu-id="79aa7-125">Open **CalendarPage.xaml.cs** and remove the following lines from the `OnAppearing` function.</span></span>

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. <span data-ttu-id="79aa7-126">На их месте добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="79aa7-126">In their place, add the following code.</span></span>

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. <span data-ttu-id="79aa7-127">Запустите приложение, войдите и нажмите элемент **навигации Календарь.**</span><span class="sxs-lookup"><span data-stu-id="79aa7-127">Run the app, sign in, and click the **Calendar** navigation item.</span></span> <span data-ttu-id="79aa7-128">Список событий с форматными значениями **"Начните"** и **"Конец".**</span><span class="sxs-lookup"><span data-stu-id="79aa7-128">You should see the list of events with the **Start** and **End** values formatted.</span></span>

    ![Снимок экрана с таблицей событий](./images/calendar-page.png)
