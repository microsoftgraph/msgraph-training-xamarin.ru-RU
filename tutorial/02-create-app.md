<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="3b6dd-101">Откройте Visual Studio и выберите **создать новый проект**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-101">Open Visual Studio, and select **Create a new project**.</span></span> <span data-ttu-id="3b6dd-102">В диалоговом окне **Создание нового проекта** выберите **мобильные приложения (Xamarin. Forms)**, а затем нажмите кнопку **Далее**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-102">In the **Create a new project** dialog, choose **Mobile App (Xamarin.Forms)**, then select **Next**.</span></span>

![Visual Studio 2019 диалоговое окно создания нового проекта](images/new-project-dialog.png)

<span data-ttu-id="3b6dd-104">В диалоговом окне **Настройка нового проекта** введите `GraphTutorial` **имя проекта** и **имя решения**, а затем нажмите кнопку **создать**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-104">In the **Configure a new project** dialog, enter `GraphTutorial` for the **Project name** and **Solution name**, then select **Create**.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3b6dd-105">Убедитесь, что вы вводите точно такое же имя для проекта Visual Studio, которое указано в этих инструкциях лаборатории.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-105">Ensure that you enter the exact same name for the Visual Studio Project that is specified in these lab instructions.</span></span> <span data-ttu-id="3b6dd-106">Имя проекта Visual Studio становится частью пространства имен в коде.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-106">The Visual Studio Project name becomes part of the namespace in the code.</span></span> <span data-ttu-id="3b6dd-107">Код в этих инструкциях зависит от пространства имен, которое соответствует имени проекта Visual Studio, указанного в данных инструкциях.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-107">The code inside these instructions depends on the namespace matching the Visual Studio Project name specified in these instructions.</span></span> <span data-ttu-id="3b6dd-108">Если вы используете другое имя проекта, код не будет компилироваться, если не настроить все пространства имен в качестве имени проекта Visual Studio, вводимого при создании проекта.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-108">If you use a different project name the code will not compile unless you adjust all the namespaces to match the Visual Studio Project name you enter when you create the project.</span></span>

![Visual Studio 2019 Настройка диалогового окна "создать проект"](images/configure-new-project-dialog.png)

<span data-ttu-id="3b6dd-110">В диалоговом окне **Создание многоплатформенного приложения** выберите **пустой** шаблон и выберите платформы, которые необходимо создать на **платформе**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-110">In the **New Cross Platform App** dialog, select the **Blank** template, and select the platforms you want to build under **Platforms**.</span></span> <span data-ttu-id="3b6dd-111">Нажмите кнопку **ОК** , чтобы создать решение.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-111">Select **OK** to create the solution.</span></span>

![Диалоговое окно Visual Studio 2019 New Cross Platform App](images/new-cross-platform-app-dialog.png)

<span data-ttu-id="3b6dd-113">Прежде чем переходить, установите некоторые дополнительные пакеты NuGet, которые будут использоваться позже.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-113">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="3b6dd-114">[Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) для обработки проверки подлинности и управления маркерами Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-114">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) to handle Azure AD authentication and token management.</span></span>
- <span data-ttu-id="3b6dd-115">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) для совершения вызовов в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-115">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to the Microsoft Graph.</span></span>

<span data-ttu-id="3b6dd-116">Выберите **инструменты > диспетчер пакетов NuGet > консоли диспетчера пакетов**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-116">Select **Tools > NuGet Package Manager > Package Manager Console**.</span></span> <span data-ttu-id="3b6dd-117">В консоли диспетчера пакетов введите указанные ниже команды.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-117">In the Package Manager Console, enter the following commands.</span></span>

```Powershell
Install-Package Microsoft.Identity.Client -Version 3.0.8 -Project GraphTutorial
Install-Package Microsoft.Identity.Client -Version 3.0.8 -Project GraphTutorial.Android
Install-Package Microsoft.Identity.Client -Version 3.0.8 -Project GraphTutorial.iOS
Install-Package Microsoft.Graph -Version 1.15.0 -Project GraphTutorial
```

## <a name="design-the-app"></a><span data-ttu-id="3b6dd-118">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="3b6dd-118">Design the app</span></span>

<span data-ttu-id="3b6dd-119">Сначала обновите `App` класс, чтобы добавить переменные для отслеживания состояния проверки подлинности и вошедшего пользователя.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-119">Start by updating the `App` class to add variables to track the authentication state and the signed-in user.</span></span> <span data-ttu-id="3b6dd-120">В **обозревателе решений**разверните проект **графтуториал** , а затем разверните файл **app. XAML** .</span><span class="sxs-lookup"><span data-stu-id="3b6dd-120">In **Solution Explorer**, expand the **GraphTutorial** project, then expand the **App.xaml** file.</span></span> <span data-ttu-id="3b6dd-121">Откройте файл **app.XAML.CS** и добавьте приведенные ниже `using` операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-121">Open the **App.xaml.cs** file and add the following `using` statements to the top of the file.</span></span>

```cs
using System.ComponentModel;
using System.IO;
using System.Reflection;
using System.Threading.Tasks;
```

<span data-ttu-id="3b6dd-122">Затем добавьте `INotifyPropertyChanged` интерфейс к объявлению класса.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-122">Next, add the `INotifyPropertyChanged` interface to the class declaration.</span></span>

```cs
public partial class App : Application, INotifyPropertyChanged
```

<span data-ttu-id="3b6dd-123">Теперь добавьте в `App` класс указанные ниже свойства.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-123">Now add the following properties to the `App` class.</span></span>

```cs
// Is a user signed in?
private bool isSignedIn;
public bool IsSignedIn
{
    get { return isSignedIn; }
    set
    {
        isSignedIn = value;
        OnPropertyChanged("IsSignedIn");
        OnPropertyChanged("IsSignedOut");
    }
}

public bool IsSignedOut { get { return !isSignedIn; } }

// The user's display name
private string userName;
public string UserName
{
    get { return userName; }
    set
    {
        userName = value;
        OnPropertyChanged("UserName");
    }
}

// The user's email address
private string userEmail;
public string UserEmail
{
    get { return userEmail; }
    set
    {
        userEmail = value;
        OnPropertyChanged("UserEmail");
    }
}

// The user's profile photo
private ImageSource userPhoto;
public ImageSource UserPhoto
{
    get { return userPhoto; }
    set
    {
        userPhoto = value;
        OnPropertyChanged("UserPhoto");
    }
}
```

<span data-ttu-id="3b6dd-124">Теперь добавьте в `App` класс следующие функции.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-124">Now add the following functions to the `App` class.</span></span> <span data-ttu-id="3b6dd-125">Функции `SignIn`, `SignOut`и, `GetUserInfo` а также — это просто заполнители.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-125">The `SignIn`, `SignOut`, and `GetUserInfo` functions are just placeholders for now.</span></span>

```cs
public async Task SignIn()
{
    await GetUserInfo();

    IsSignedIn = true;
}

public async Task SignOut()
{
    UserPhoto = null;
    UserName = string.Empty;
    UserEmail = string.Empty;
    IsSignedIn = false;
}

private async Task GetUserInfo()
{
    UserPhoto = ImageSource.FromStream(() => GetUserPhoto());
    UserName = "Adele Vance";
    UserEmail = "adelev@contoso.com";
}

private Stream GetUserPhoto()
{
    // Return the default photo
    return Assembly.GetExecutingAssembly().GetManifestResourceStream("GraphTutorial.no-profile-pic.png");
}
```

<span data-ttu-id="3b6dd-126">`GetUserPhoto` Функция возвращает фотографию, используемую по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-126">The `GetUserPhoto` function returns a default photo for now.</span></span> <span data-ttu-id="3b6dd-127">Вы можете указать собственный файл здесь или скачать его, использованный в примере, на сайте [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span><span class="sxs-lookup"><span data-stu-id="3b6dd-127">You can either supply your own file here, or you can download the one used in the sample from [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span></span> <span data-ttu-id="3b6dd-128">Скопируйте файл в `./GraphTutorial/GraphTutorial` каталог.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-128">Copy the file to the `./GraphTutorial/GraphTutorial` directory.</span></span> <span data-ttu-id="3b6dd-129">Щелкните правой кнопкой мыши проект **графтуториал** в **обозревателе решений** и выберите **добавить**, а затем **существующий элемент...**. Выберите `no-profile-pic.png` файл и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-129">Right-click the **GraphTutorial** project in **Solution Explorer** and select **Add**, then **Existing Item...**. Select the `no-profile-pic.png` file and select **Add**.</span></span> <span data-ttu-id="3b6dd-130">Теперь щелкните правой кнопкой мыши файл в **обозревателе решений** и выберите пункт **свойства**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-130">Now right-click the file in **Solution Explorer** and select **Properties**.</span></span> <span data-ttu-id="3b6dd-131">В окне **Свойства** измените значение **действия** при построении на внедренный **ресурс**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-131">In the **Properties** window, change the value of **Build Action** to **Embedded resource**.</span></span>

![Снимок экрана: окно "Свойства" для PNG-файла](./images/png-file-properties.png)

### <a name="app-navigation"></a><span data-ttu-id="3b6dd-133">Навигация в приложении</span><span class="sxs-lookup"><span data-stu-id="3b6dd-133">App navigation</span></span>

<span data-ttu-id="3b6dd-134">Затем замените основную страницу приложения на [страницу "основной-подробности](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page)".</span><span class="sxs-lookup"><span data-stu-id="3b6dd-134">Next, change the application's main page to a [Master-Detail page](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span></span> <span data-ttu-id="3b6dd-135">Откроется меню навигации, чтобы переключиться между представлениями в приложении.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-135">This will provide a navigation menu to switch between view in the app.</span></span>

<span data-ttu-id="3b6dd-136">Откройте файл **MainPage. XAML** в проекте **графтуториал** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-136">Open the **MainPage.xaml** file in the **GraphTutorial** project and replace its contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<MasterDetailPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:GraphTutorial"
             x:Class="GraphTutorial.MainPage">

    <MasterDetailPage.Master>
        <local:MenuPage/>
    </MasterDetailPage.Master>

    <MasterDetailPage.Detail>
        <NavigationPage>
            <x:Arguments>
                <local:WelcomePage/>
            </x:Arguments>
        </NavigationPage>
    </MasterDetailPage.Detail>

</MasterDetailPage>
```

#### <a name="implement-the-menu"></a><span data-ttu-id="3b6dd-137">Реализация меню</span><span class="sxs-lookup"><span data-stu-id="3b6dd-137">Implement the menu</span></span>

<span data-ttu-id="3b6dd-138">Начните с создания модели для представления элементов меню.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-138">Start by creating a model to represent the menu items.</span></span> <span data-ttu-id="3b6dd-139">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **Новая папка**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-139">Right-click the **GraphTutorial** project and select **Add**, then **New Folder**.</span></span> <span data-ttu-id="3b6dd-140">Назовите папку `Models`.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-140">Name the folder `Models`.</span></span>

<span data-ttu-id="3b6dd-141">Щелкните правой кнопкой мыши \*\*\*\* папку Models и выберите команду **Добавить**, а затем — **класс...**. Назовите класс `NavMenuItem` и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-141">Right-click the **Models** folder and select **Add**, then **Class...**. Name the class `NavMenuItem` and select **Add**.</span></span> <span data-ttu-id="3b6dd-142">Откройте файл **NavMenuItem.CS** и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-142">Open the **NavMenuItem.cs** file and replace its contents with the following.</span></span>

```cs
namespace GraphTutorial.Models
{
    public enum MenuItemType
    {
        Welcome,
        Calendar
    }

    public class NavMenuItem
    {
        public MenuItemType Id { get; set; }

        public string Title { get; set; }
    }
}
```

<span data-ttu-id="3b6dd-143">Теперь добавьте страницу меню.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-143">Now add the menu page.</span></span> <span data-ttu-id="3b6dd-144">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **новый элемент...**. Выберите **страницу содержимого** и присвойте странице `MenuPage`имя.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-144">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `MenuPage`.</span></span> <span data-ttu-id="3b6dd-145">Нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-145">Select **Add**.</span></span> <span data-ttu-id="3b6dd-146">Откройте файл **менупаже. XAML** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-146">Open the **MenuPage.xaml** file and replace its contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:ios="clr-namespace:Xamarin.Forms.PlatformConfiguration.iOSSpecific;assembly=Xamarin.Forms.Core"
             ios:Page.UseSafeArea="true"
             Title="Menu"
             x:Class="GraphTutorial.MenuPage">
    <ContentPage.Padding>
        <OnPlatform x:TypeArguments="Thickness">
            <On Platform="UWP" Value="10, 10, 10, 10" />
        </OnPlatform>
    </ContentPage.Padding>
    <ContentPage.Content>
        <StackLayout VerticalOptions="Start" HorizontalOptions="Center">
            <StackLayout x:Name="UserArea" />

            <!-- Signed out UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedOut, Source={x:Static Application.Current}}">
                <Label Text="Sign in to get started"
                       HorizontalOptions="Center"
                       FontAttributes="Bold"
                       FontSize="Medium"
                       Margin="10,20,10,20" />
                <Button Text="Sign in"
                        Clicked="OnSignIn"
                        HorizontalOptions="Center" />
            </StackLayout>

            <!-- Signed in UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedIn, Source={x:Static Application.Current}}">
                <Image Source="{Binding Path=UserPhoto, Source={x:Static Application.Current}}"
                       HorizontalOptions="Center"
                       Margin="0,20,0,10" />
                <Label Text="{Binding Path=UserName, Source={x:Static Application.Current}}"
                       HorizontalOptions="Center"
                       FontAttributes="Bold"
                       FontSize="Small" />
                <Label Text="{Binding Path=UserEmail, Source={x:Static Application.Current}}"
                       HorizontalOptions="Center"
                       FontAttributes="Italic" />
                <Button Text="Sign out"
                        Margin="0,20,0,20"
                        Clicked="OnSignOut"
                        HorizontalOptions="Center" />
                <ListView x:Name="ListViewMenu"
                          HasUnevenRows="True"
                          HorizontalOptions="Start">
                    <ListView.ItemTemplate>
                        <DataTemplate>
                            <ViewCell>
                                <Grid Padding="10">
                                    <Label Text="{Binding Title}" FontSize="20"/>
                                </Grid>
                            </ViewCell>
                        </DataTemplate>
                    </ListView.ItemTemplate>
                </ListView>
            </StackLayout>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```

<span data-ttu-id="3b6dd-147">Теперь разверните **менупаже. XAML** в **обозревателе решений** и откройте файл **MenuPage.XAML.CS** .</span><span class="sxs-lookup"><span data-stu-id="3b6dd-147">Now, expand **MenuPage.xaml** in **Solution Explorer** and open the **MenuPage.xaml.cs** file.</span></span> <span data-ttu-id="3b6dd-148">Замените его содержимое приведенным ниже.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-148">Replace its contents with the following.</span></span>

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using Xamarin.Forms;
using Xamarin.Forms.Xaml;

using GraphTutorial.Models;

namespace GraphTutorial
{
    [XamlCompilation(XamlCompilationOptions.Compile)]
    public partial class MenuPage : ContentPage
    {
        MainPage RootPage => Application.Current.MainPage as MainPage;
        List<NavMenuItem> menuItems;

        public MenuPage ()
        {
            InitializeComponent ();

            // Add items to the menu
            menuItems = new List<NavMenuItem>
            {
                new NavMenuItem {Id = MenuItemType.Welcome, Title="Home" },
                new NavMenuItem {Id = MenuItemType.Calendar, Title="Calendar" }
            };
            ListViewMenu.ItemsSource = menuItems;

            // Initialize the selected item
            ListViewMenu.SelectedItem = menuItems[0];

            // Handle the ItemSelected event to navigate to the
            // selected page
            ListViewMenu.ItemSelected += async (sender, e) =>
            {
                if (e.SelectedItem == null)
                    return;

                var id = (int)((NavMenuItem)e.SelectedItem).Id;
                await RootPage.NavigateFromMenu(id);
            };
        }

        private async void OnSignOut(object sender, EventArgs e)
        {
            var signout = await DisplayAlert("Sign out?", "Do you want to sign out?", "Yes", "No");
            if (signout)
            {
                await (Application.Current as App).SignOut();
            }
        }

        private async void OnSignIn(object sender, EventArgs e)
        {
            await (Application.Current as App).SignIn();
        }
    }
}
```

> [!NOTE]
> <span data-ttu-id="3b6dd-149">Visual Studio сообщит об ошибках в **MenuPage.XAML.CS**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-149">Visual Studio will report errors in **MenuPage.xaml.cs**.</span></span> <span data-ttu-id="3b6dd-150">Эти ошибки будут устранены на более позднем этапе.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-150">These errors will be resolved in a later step.</span></span>

#### <a name="implement-the-welcome-page"></a><span data-ttu-id="3b6dd-151">Реализация страницы приветствия</span><span class="sxs-lookup"><span data-stu-id="3b6dd-151">Implement the welcome page</span></span>

<span data-ttu-id="3b6dd-152">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **новый элемент...**. Выберите **страницу содержимого** и присвойте странице `WelcomePage`имя.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-152">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `WelcomePage`.</span></span> <span data-ttu-id="3b6dd-153">Нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-153">Select **Add**.</span></span> <span data-ttu-id="3b6dd-154">Откройте файл **велкомепаже. XAML** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-154">Open the **WelcomePage.xaml** file and replace its contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             Title="Home"
             x:Class="GraphTutorial.WelcomePage">
    <ContentPage.Padding>
        <OnPlatform x:TypeArguments="Thickness">
            <On Platform="UWP" Value="10, 10, 10, 10" />
        </OnPlatform>
    </ContentPage.Padding>
    <ContentPage.Content>
        <StackLayout>
            <Label Text="Graph Xamarin Tutorial App"
                       HorizontalOptions="Center"
                       FontAttributes="Bold"
                       FontSize="Large"
                       Margin="10,20,10,20" />

            <!-- Signed out UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedOut, Source={x:Static Application.Current}}">
                <Label Text="Please sign in to get started"
                       HorizontalOptions="Center"
                       FontSize="Medium"
                       Margin="10,0,10,20"/>
                <Button Text="Sign in"
                        HorizontalOptions="Center"
                        Clicked="OnSignIn" />
            </StackLayout>

            <!-- Signed in UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedIn, Source={x:Static Application.Current}}">
                <Label Text="{Binding Path=UserName, Source={x:Static Application.Current}, StringFormat='Welcome \{0\}!'}"
                       HorizontalOptions="Center"
                       FontSize="Medium"/>
            </StackLayout>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```

<span data-ttu-id="3b6dd-155">Теперь разверните **велкомепаже. XAML** в **обозревателе решений** и откройте файл **WelcomePage.XAML.CS** .</span><span class="sxs-lookup"><span data-stu-id="3b6dd-155">Now, expand **WelcomePage.xaml** in **Solution Explorer** and open the **WelcomePage.xaml.cs** file.</span></span> <span data-ttu-id="3b6dd-156">Добавьте к классу `WelcomePage` следующую функцию:</span><span class="sxs-lookup"><span data-stu-id="3b6dd-156">Add the following function to the `WelcomePage` class.</span></span>

```cs
private void OnSignIn(object sender, EventArgs e)
{
    (App.Current.MainPage as MasterDetailPage).IsPresented = true;
}
```

#### <a name="add-calendar-page"></a><span data-ttu-id="3b6dd-157">Страница "Добавление календаря"</span><span class="sxs-lookup"><span data-stu-id="3b6dd-157">Add calendar page</span></span>

<span data-ttu-id="3b6dd-158">Теперь добавьте страницу "Календарь".</span><span class="sxs-lookup"><span data-stu-id="3b6dd-158">Now add a calendar page.</span></span> <span data-ttu-id="3b6dd-159">Это будет просто заполнитель.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-159">This will just be a placeholder for now.</span></span> <span data-ttu-id="3b6dd-160">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **новый элемент...**. Выберите **страницу содержимого** и присвойте странице `CalendarPage`имя.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-160">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `CalendarPage`.</span></span> <span data-ttu-id="3b6dd-161">Нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-161">Select **Add**.</span></span>

<span data-ttu-id="3b6dd-162">Оставьте добавленную страницу как есть.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-162">Leave the added page as-is for now.</span></span>

#### <a name="update-mainpage-code-behind"></a><span data-ttu-id="3b6dd-163">Обновление кода MainPage</span><span class="sxs-lookup"><span data-stu-id="3b6dd-163">Update MainPage code-behind</span></span>

<span data-ttu-id="3b6dd-164">Теперь, когда все страницы находятся на месте, обновите код программной части для **MainPage. XAML**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-164">Now that all of the pages are in place, update the code-behind for **MainPage.xaml**.</span></span> <span data-ttu-id="3b6dd-165">Разверните **MainPage. XAML** в **обозревателе решений** и откройте файл **MainPage.XAML.CS** и замените все содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-165">Expand **MainPage.xaml** in **Solution Explorer** and open the **MainPage.xaml.cs** file and replace its entire contents with the following.</span></span>

```cs
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Reflection;
using System.Text;
using System.Threading.Tasks;
using Xamarin.Forms;
using GraphTutorial.Models;

namespace GraphTutorial
{
    public partial class MainPage : MasterDetailPage
    {
        Dictionary<int, NavigationPage> MenuPages = new Dictionary<int, NavigationPage>();

        public MainPage()
        {
            InitializeComponent();

            MasterBehavior = MasterBehavior.Popover;

            // Load the welcome page at start
            MenuPages.Add((int)MenuItemType.Welcome, (NavigationPage)Detail);
        }

        // Navigate to the selected page
        public async Task NavigateFromMenu(int id)
        {
            if (!MenuPages.ContainsKey(id))
            {
                switch (id)
                {
                    case (int)MenuItemType.Welcome:
                        MenuPages.Add(id, new NavigationPage(new WelcomePage()));
                        break;
                    case (int)MenuItemType.Calendar:
                        MenuPages.Add(id, new NavigationPage(new CalendarPage()));
                        break;
                }
            }

            var newPage = MenuPages[id];

            if (newPage != null && Detail != newPage)
            {
                Detail = newPage;

                if (Device.RuntimePlatform == Device.Android)
                    await Task.Delay(100);

                IsPresented = false;
            }
        }
    }
}
```

<span data-ttu-id="3b6dd-166">Сохраните все изменения.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-166">Save all of your changes.</span></span> <span data-ttu-id="3b6dd-167">Щелкните правой кнопкой мыши проект, который требуется запустить (Android, iOS или UWP), и выберите пункт **Назначить запускаемым проектом**.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-167">Right-click the project that you want to run (Android, iOS, or UWP) and select **Set as StartUp Project**.</span></span> <span data-ttu-id="3b6dd-168">Нажмите клавишу **F5** или выберите **Отладка > начать отладку** в Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="3b6dd-168">Press **F5** or select **Debug > Start Debugging** in Visual Studio.</span></span>

![Снимки экрана с версиями приложения для Android, iOS и UWP](./images/welcome-page.png)
