<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="270ab-101">В этом упражнении вы добавите Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="270ab-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="270ab-102">Для этого приложения вы будете использовать [клиентскую библиотеку Microsoft Graph для .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) , чтобы совершать вызовы в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="270ab-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="270ab-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="270ab-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="270ab-104">Откройте **календарпаже. XAML** в проекте **графтуториал** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="270ab-104">Open **CalendarPage.xaml** in the **GraphTutorial** project and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="270ab-105">Откройте **CalendarPage.XAML.CS** и добавьте приведенные `using` ниже операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="270ab-105">Open **CalendarPage.xaml.cs** and add the following `using` statements at the top of the file.</span></span>

    ```csharp
    using Microsoft.Graph;
    using Newtonsoft.Json;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. <span data-ttu-id="270ab-106">Добавьте указанную ниже функцию в `CalendarPage` класс, чтобы получить события пользователя и ОТОБРАЗИТЬ ответ JSON.</span><span class="sxs-lookup"><span data-stu-id="270ab-106">Add the following function to the `CalendarPage` class to get the user's events and display the JSON response.</span></span>

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

    <span data-ttu-id="270ab-107">Рассмотрите, какие действия `OnAppearing` выполняет код.</span><span class="sxs-lookup"><span data-stu-id="270ab-107">Consider what the code in `OnAppearing` is doing.</span></span>

    - <span data-ttu-id="270ab-108">URL-адрес, который будет вызываться — это `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="270ab-108">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="270ab-109">`Select` Функция ограничит поля, возвращаемые для каждого события, только теми, которые будут реально использоваться в представлении.</span><span class="sxs-lookup"><span data-stu-id="270ab-109">The `Select` function limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="270ab-110">`OrderBy` Функция сортирует результаты по дате и времени создания, начиная с самого последнего элемента.</span><span class="sxs-lookup"><span data-stu-id="270ab-110">The `OrderBy` function sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="270ab-111">Запустите приложение и войдите в него, а затем щелкните элемент Навигация в **календаре** в меню.</span><span class="sxs-lookup"><span data-stu-id="270ab-111">Run the app, sign in, and click the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="270ab-112">Вы должны увидеть дамп событий в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="270ab-112">You should see a JSON dump of the events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="270ab-113">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="270ab-113">Display the results</span></span>

<span data-ttu-id="270ab-114">Теперь вы можете заменить дамп JSON на какой-то способ отобразить результаты в удобном для пользователя виде.</span><span class="sxs-lookup"><span data-stu-id="270ab-114">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span>

<span data-ttu-id="270ab-115">Сначала создайте [преобразователь значений привязки](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) , чтобы преобразовать значения [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) , возвращенные Microsoft Graph, в форматы даты и времени, которые ожидает пользователь.</span><span class="sxs-lookup"><span data-stu-id="270ab-115">Start by creating a [binding value converter](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) to convert the [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) values returned by Microsoft Graph into the date and time formats the user expects.</span></span>

1. <span data-ttu-id="270ab-116">Щелкните правой кнопкой мыши папку **Models** в проекте **графтуториал** и выберите команду **добавить**, а затем — **класс...**. Назовите класс `GraphDateTimeTimeZoneConverter` и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="270ab-116">Right-click the **Models** folder in the **GraphTutorial** project and select **Add**, then **Class...**. Name the class `GraphDateTimeTimeZoneConverter` and select **Add**.</span></span>

1. <span data-ttu-id="270ab-117">Замените все содержимое файла на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="270ab-117">Replace the entire contents of the file with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. <span data-ttu-id="270ab-118">Замените все содержимое файла **календарпаже. XAML** следующим кодом:</span><span class="sxs-lookup"><span data-stu-id="270ab-118">Replace the entire contents of **CalendarPage.xaml** with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml":::

    <span data-ttu-id="270ab-119">Вместо него `Editor` заменяется `ListView`.</span><span class="sxs-lookup"><span data-stu-id="270ab-119">This replaces the `Editor` with a `ListView`.</span></span> <span data-ttu-id="270ab-120">Объект `DataTemplate` , используемый для отображения каждого элемента, `GraphDateTimeTimeZoneConverter` использует для преобразования `Start` свойства `End` и свойства события в строку.</span><span class="sxs-lookup"><span data-stu-id="270ab-120">The `DataTemplate` used to render each item uses the `GraphDateTimeTimeZoneConverter` to convert the `Start` and `End` properties of the event to a string.</span></span>

1. <span data-ttu-id="270ab-121">Откройте **CalendarPage.XAML.CS** и удалите из `OnAppearing` функции следующие строки.</span><span class="sxs-lookup"><span data-stu-id="270ab-121">Open **CalendarPage.xaml.cs** and remove the following lines from the `OnAppearing` function.</span></span>

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. <span data-ttu-id="270ab-122">Вместо этого добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="270ab-122">In their place, add the following code.</span></span>

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. <span data-ttu-id="270ab-123">Запустите приложение и войдите в систему, а затем щелкните элемент навигации по **календарю** .</span><span class="sxs-lookup"><span data-stu-id="270ab-123">Run the app, sign in, and click the **Calendar** navigation item.</span></span> <span data-ttu-id="270ab-124">Вы должны увидеть список событий с отформатированными **начальным** и **конечным** значениями.</span><span class="sxs-lookup"><span data-stu-id="270ab-124">You should see the list of events with the **Start** and **End** values formatted.</span></span>

    ![Снимок экрана с таблицей событий](./images/calendar-page.png)
