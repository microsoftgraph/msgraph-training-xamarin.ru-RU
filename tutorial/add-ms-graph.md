<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="fc14f-101">В этом упражнении вы добавите Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="fc14f-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="fc14f-102">Для этого приложения вы будете использовать клиентскую [библиотеку Microsoft Graph для .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) , чтобы совершать вызовы в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="fc14f-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="fc14f-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="fc14f-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="fc14f-104">Для начала обновите файл **календарпаже. XAML** в проекте **графтуториал** .</span><span class="sxs-lookup"><span data-stu-id="fc14f-104">Start by updating the **CalendarPage.xaml** file in the **GraphTutorial** project.</span></span> <span data-ttu-id="fc14f-105">Откройте файл и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="fc14f-105">Open the file and replace its contents with the following.</span></span>

```xml
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

<span data-ttu-id="fc14f-106">Откройте `CalendarPage.xaml.cs` и добавьте приведенные `using` ниже операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="fc14f-106">Open `CalendarPage.xaml.cs` and add the following `using` statements at the top of the file.</span></span>

```cs
using Newtonsoft.Json;
using Microsoft.Graph;
using System.ComponentModel;
using System.Collections.ObjectModel;
```

<span data-ttu-id="fc14f-107">Затем добавьте приведенный ниже код, чтобы получить события пользователя и отобразить ответ JSON.</span><span class="sxs-lookup"><span data-stu-id="fc14f-107">Then add the following code to get the user's events and display the JSON response.</span></span>

```cs
protected override async void OnAppearing()
{
    base.OnAppearing();

    // Get the events
    var events = await App.GraphClient.Me.Events.Request()
        .Select("subject,organizer,start,end")
        .OrderBy("createdDateTime DESC")
        .GetAsync();

    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
}
```

<span data-ttu-id="fc14f-108">РасСмотрите, какие действия `OnAppearing` выполняет код.</span><span class="sxs-lookup"><span data-stu-id="fc14f-108">Consider what the code in `OnAppearing` is doing.</span></span>

- <span data-ttu-id="fc14f-109">URL-адрес, который будет вызываться — это `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="fc14f-109">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="fc14f-110">`Select` Функция ограничит поля, возвращаемые для каждого события, только теми, которые будут реально использоваться в представлении.</span><span class="sxs-lookup"><span data-stu-id="fc14f-110">The `Select` function limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="fc14f-111">`OrderBy` Функция сортирует результаты по дате и времени создания, начиная с самого последнего элемента.</span><span class="sxs-lookup"><span data-stu-id="fc14f-111">The `OrderBy` function sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="fc14f-112">Теперь вы можете запустить приложение, войти и щелкнуть элемент навигации в **календаре** в меню.</span><span class="sxs-lookup"><span data-stu-id="fc14f-112">You can now run the app, sign in, and click the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="fc14f-113">Вы должны увидеть дамп событий в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="fc14f-113">You should see a JSON dump of the events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="fc14f-114">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="fc14f-114">Display the results</span></span>

<span data-ttu-id="fc14f-115">Теперь вы можете заменить дамп JSON на какой-то способ отобразить результаты в удобном для пользователя виде.</span><span class="sxs-lookup"><span data-stu-id="fc14f-115">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="fc14f-116">Сначала создайте преобразователь [значений привязки](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) , чтобы преобразовать значения [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) , возвращенные Microsoft Graph, в форматы даты и времени, которые ожидает пользователь.</span><span class="sxs-lookup"><span data-stu-id="fc14f-116">Start by creating a [binding value converter](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) to convert the [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) values returned by Microsoft Graph into the date and time formats the user expects.</span></span> <span data-ttu-id="fc14f-117">Щелкните правой кнопкой мыши \*\*\*\* папку Models в проекте **Графтуториал** и выберите **создать**, затем **класс...**. НаЗовите класс `GraphDateTimeTimeZoneConverter` и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="fc14f-117">Right-click the **Models** folder in the **GraphTutorial** project and choose **New**, then **Class...**. Name the class `GraphDateTimeTimeZoneConverter` and choose **Add**.</span></span> <span data-ttu-id="fc14f-118">Замените все содержимое файла на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="fc14f-118">Replace the entire contents of the file with the following.</span></span>

```cs
using Microsoft.Graph;
using System;
using System.Globalization;
using Xamarin.Forms;

namespace GraphTutorial.Models
{
    class GraphDateTimeTimeZoneConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value is DateTimeTimeZone date)
            {
                // Resolve the time zone
                var timezone = TimeZoneInfo.FindSystemTimeZoneById(date.TimeZone);
                // Parse method assumes local time, which may not be the case
                var parsedDateAsLocal = DateTimeOffset.Parse(date.DateTime);
                // Determine the offset from UTC time for the specific date
                // Making this call adjusts for DST as appropriate
                var tzOffset = timezone.GetUtcOffset(parsedDateAsLocal.DateTime);
                // Create a new DateTimeOffset with the specific offset from UTC
                var correctedDate = new DateTimeOffset(parsedDateAsLocal.DateTime, tzOffset);
                // Return the local date time string
                return $"{correctedDate.LocalDateTime.ToShortDateString()} {correctedDate.LocalDateTime.ToShortTimeString()}";
            }

            return string.Empty;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
}
```

<span data-ttu-id="fc14f-119">Затем замените все содержимое `CalendarPage.xaml` следующим.</span><span class="sxs-lookup"><span data-stu-id="fc14f-119">Next, replace the entire contents of `CalendarPage.xaml` with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:GraphTutorial.Models"
             Title="Calendar"
             x:Class="GraphTutorial.CalendarPage">
    <ContentPage.Resources>
        <local:GraphDateTimeTimeZoneConverter x:Key="DateConverter" />
    </ContentPage.Resources>
    <ContentPage.Content>
        <StackLayout>
            <ListView x:Name="CalendarList"
                      VerticalOptions="StartAndExpand"
                      Margin="10,10,10,10">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <ViewCell>
                            <StackLayout Margin="10,10,10,10">
                                <Label Text="{Binding Path=Subject}"
                                       FontAttributes="Bold"
                                       FontSize="Medium" />
                                <Label Text="{Binding Path=Organizer.EmailAddress.Name}"
                                       FontSize="Small" />
                                <StackLayout Orientation="Horizontal">
                                    <Label Text="{Binding Path=Start, Converter={StaticResource DateConverter}}"
                                       FontSize="Micro" />
                                    <Label Text="to"
                                           FontSize="Micro" />
                                    <Label Text="{Binding Path=End, Converter={StaticResource DateConverter}}"
                                       FontSize="Micro" />
                                </StackLayout>
                            </StackLayout>
                        </ViewCell>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```

<span data-ttu-id="fc14f-120">Вместо него `Editor` заменяется `ListView`.</span><span class="sxs-lookup"><span data-stu-id="fc14f-120">This replaces the `Editor` with a `ListView`.</span></span> <span data-ttu-id="fc14f-121">Объект `DataTemplate` , используемый для отображения каждого элемента, `GraphDateTimeTimeZoneConverter` использует для преобразования `Start` свойства `End` и свойства события в строку.</span><span class="sxs-lookup"><span data-stu-id="fc14f-121">The `DataTemplate` used to render each item uses the `GraphDateTimeTimeZoneConverter` to convert the `Start` and `End` properties of the event to a string.</span></span> <span data-ttu-id="fc14f-122">Теперь откройте `CalendarPage.xaml.cs` и удалите из `OnAppearing` функции следующие строки.</span><span class="sxs-lookup"><span data-stu-id="fc14f-122">Now open `CalendarPage.xaml.cs` and remove the following lines from the `OnAppearing` function.</span></span>

```cs
// Temporary
JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
```

<span data-ttu-id="fc14f-123">Вместо этого добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="fc14f-123">In their place, add the following code.</span></span>

```cs
// Add the events to the list view
CalendarList.ItemsSource = events.CurrentPage.ToList();
```

<span data-ttu-id="fc14f-124">Запустите приложение и войдите в систему, а затем щелкните \*\*\*\* элемент навигации по календарю.</span><span class="sxs-lookup"><span data-stu-id="fc14f-124">Run the app, sign in, and click the **Calendar** navigation item.</span></span> <span data-ttu-id="fc14f-125">Вы должны увидеть список событий с отформатированными **начальным** и **конечным** значениями.</span><span class="sxs-lookup"><span data-stu-id="fc14f-125">You should see the list of events with the **Start** and **End** values formatted.</span></span>

![Снимок экрана С таблицей событий](./images/calendar-page.png)
