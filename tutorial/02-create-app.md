<!-- markdownlint-disable MD002 MD041 -->

1. <span data-ttu-id="2fe17-101">Откройте Visual Studio и выберите **создать новый проект**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-101">Open Visual Studio, and select **Create a new project**.</span></span>

1. <span data-ttu-id="2fe17-102">В диалоговом окне **Создание нового проекта** выберите **мобильные приложения (Xamarin. Forms)**, а затем нажмите кнопку **Далее**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-102">In the **Create a new project** dialog, choose **Mobile App (Xamarin.Forms)**, then select **Next**.</span></span>

    ![Visual Studio 2019 диалоговое окно создания нового проекта](images/new-project-dialog.png)

1. <span data-ttu-id="2fe17-104">В диалоговом окне **Настройка нового проекта** введите `GraphTutorial` **имя проекта** и **имя решения**, а затем нажмите кнопку **создать**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-104">In the **Configure a new project** dialog, enter `GraphTutorial` for the **Project name** and **Solution name**, then select **Create**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="2fe17-105">Убедитесь, что вы вводите точно такое же имя для проекта Visual Studio, которое указано в этих инструкциях лаборатории.</span><span class="sxs-lookup"><span data-stu-id="2fe17-105">Ensure that you enter the exact same name for the Visual Studio Project that is specified in these lab instructions.</span></span> <span data-ttu-id="2fe17-106">Имя проекта Visual Studio становится частью пространства имен в коде.</span><span class="sxs-lookup"><span data-stu-id="2fe17-106">The Visual Studio Project name becomes part of the namespace in the code.</span></span> <span data-ttu-id="2fe17-107">Код в этих инструкциях зависит от пространства имен, которое соответствует имени проекта Visual Studio, указанного в данных инструкциях.</span><span class="sxs-lookup"><span data-stu-id="2fe17-107">The code inside these instructions depends on the namespace matching the Visual Studio Project name specified in these instructions.</span></span> <span data-ttu-id="2fe17-108">Если вы используете другое имя проекта, код не будет компилироваться, если не настроить все пространства имен в качестве имени проекта Visual Studio, вводимого при создании проекта.</span><span class="sxs-lookup"><span data-stu-id="2fe17-108">If you use a different project name the code will not compile unless you adjust all the namespaces to match the Visual Studio Project name you enter when you create the project.</span></span>

    ![Visual Studio 2019 Настройка диалогового окна "создать проект"](images/configure-new-project-dialog.png)

1. <span data-ttu-id="2fe17-110">В диалоговом окне **Создание многоплатформенного приложения** выберите **пустой** шаблон и выберите платформы, которые необходимо создать на **платформе**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-110">In the **New Cross Platform App** dialog, select the **Blank** template, and select the platforms you want to build under **Platforms**.</span></span> <span data-ttu-id="2fe17-111">Нажмите кнопку **ОК** , чтобы создать решение.</span><span class="sxs-lookup"><span data-stu-id="2fe17-111">Select **OK** to create the solution.</span></span>

    ![Диалоговое окно Visual Studio 2019 New Cross Platform App](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a><span data-ttu-id="2fe17-113">Установка пакетов</span><span class="sxs-lookup"><span data-stu-id="2fe17-113">Install packages</span></span>

<span data-ttu-id="2fe17-114">Прежде чем переходить, установите некоторые дополнительные пакеты NuGet, которые будут использоваться позже.</span><span class="sxs-lookup"><span data-stu-id="2fe17-114">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="2fe17-115">[Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) для обработки проверки подлинности и управления маркерами Azure AD.</span><span class="sxs-lookup"><span data-stu-id="2fe17-115">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) to handle Azure AD authentication and token management.</span></span>
- <span data-ttu-id="2fe17-116">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) для совершения вызовов в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="2fe17-116">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to the Microsoft Graph.</span></span>

1. <span data-ttu-id="2fe17-117">Выберите **инструменты > диспетчер пакетов NuGet > консоли диспетчера пакетов**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-117">Select **Tools > NuGet Package Manager > Package Manager Console**.</span></span>

1. <span data-ttu-id="2fe17-118">В консоли диспетчера пакетов введите указанные ниже команды.</span><span class="sxs-lookup"><span data-stu-id="2fe17-118">In the Package Manager Console, enter the following commands.</span></span>

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 3.0.1 -Project GraphTutorial
    ```

## <a name="design-the-app"></a><span data-ttu-id="2fe17-119">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="2fe17-119">Design the app</span></span>

<span data-ttu-id="2fe17-120">Сначала обновите `App` класс, чтобы добавить переменные для отслеживания состояния проверки подлинности и вошедшего пользователя.</span><span class="sxs-lookup"><span data-stu-id="2fe17-120">Start by updating the `App` class to add variables to track the authentication state and the signed-in user.</span></span>

1. <span data-ttu-id="2fe17-121">В **обозревателе решений**разверните проект **графтуториал** , а затем разверните файл **app. XAML** .</span><span class="sxs-lookup"><span data-stu-id="2fe17-121">In **Solution Explorer**, expand the **GraphTutorial** project, then expand the **App.xaml** file.</span></span> <span data-ttu-id="2fe17-122">Откройте файл **app.XAML.CS** и добавьте приведенные ниже `using` операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="2fe17-122">Open the **App.xaml.cs** file and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using System.ComponentModel;
    using System.IO;
    using System.Reflection;
    using System.Threading.Tasks;
    ```

1. <span data-ttu-id="2fe17-123">Добавьте `INotifyPropertyChanged` интерфейс к объявлению класса.</span><span class="sxs-lookup"><span data-stu-id="2fe17-123">Add the `INotifyPropertyChanged` interface to the class declaration.</span></span>

    ```csharp
    public partial class App : Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="2fe17-124">Добавьте в `App` класс указанные ниже свойства.</span><span class="sxs-lookup"><span data-stu-id="2fe17-124">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GlobalPropertiesSnippet":::

1. <span data-ttu-id="2fe17-125">Добавьте в `App` класс следующие функции.</span><span class="sxs-lookup"><span data-stu-id="2fe17-125">Add the following functions to the `App` class.</span></span> <span data-ttu-id="2fe17-126">Функции `SignIn`, `SignOut`и, `GetUserInfo` а также — это просто заполнители.</span><span class="sxs-lookup"><span data-stu-id="2fe17-126">The `SignIn`, `SignOut`, and `GetUserInfo` functions are just placeholders for now.</span></span>

    ```csharp
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

1. <span data-ttu-id="2fe17-127">`GetUserPhoto` Функция возвращает фотографию, используемую по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="2fe17-127">The `GetUserPhoto` function returns a default photo for now.</span></span> <span data-ttu-id="2fe17-128">Вы можете указать собственный файл или скачать его, который использовался в примере, на сайте [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span><span class="sxs-lookup"><span data-stu-id="2fe17-128">You can either supply your own file, or you can download the one used in the sample from [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span></span> <span data-ttu-id="2fe17-129">Если вы используете свой файл, переименуйте его в **но-профиле-пик. png**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-129">If you use your own file, rename it to **no-profile-pic.png**.</span></span>

1. <span data-ttu-id="2fe17-130">Скопируйте файл в каталог **./графтуториал/графтуториал** .</span><span class="sxs-lookup"><span data-stu-id="2fe17-130">Copy the file to the **./GraphTutorial/GraphTutorial** directory.</span></span>

1. <span data-ttu-id="2fe17-131">Щелкните правой кнопкой мыши файл в **обозревателе решений** и выберите пункт **свойства**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-131">Right-click the file in **Solution Explorer** and select **Properties**.</span></span> <span data-ttu-id="2fe17-132">В окне **Свойства** измените значение **действия при построении** на **внедренный ресурс**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-132">In the **Properties** window, change the value of **Build Action** to **Embedded resource**.</span></span>

    ![Снимок экрана: окно "Свойства" для PNG-файла](./images/png-file-properties.png)

### <a name="app-navigation"></a><span data-ttu-id="2fe17-134">Навигация в приложении</span><span class="sxs-lookup"><span data-stu-id="2fe17-134">App navigation</span></span>

<span data-ttu-id="2fe17-135">В этом разделе показано, как изменить основную страницу приложения на [страницу с основными и подробными данными](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span><span class="sxs-lookup"><span data-stu-id="2fe17-135">In this section, you'll change the application's main page to a [Master-Detail page](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span></span> <span data-ttu-id="2fe17-136">Откроется меню навигации, чтобы переключиться между представлениями в приложении.</span><span class="sxs-lookup"><span data-stu-id="2fe17-136">This will provide a navigation menu to switch between view in the app.</span></span>

1. <span data-ttu-id="2fe17-137">Откройте файл **MainPage. XAML** в проекте **графтуториал** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="2fe17-137">Open the **MainPage.xaml** file in the **GraphTutorial** project and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml":::

#### <a name="implement-the-menu"></a><span data-ttu-id="2fe17-138">Реализация меню</span><span class="sxs-lookup"><span data-stu-id="2fe17-138">Implement the menu</span></span>

1. <span data-ttu-id="2fe17-139">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **Новая папка**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-139">Right-click the **GraphTutorial** project and select **Add**, then **New Folder**.</span></span> <span data-ttu-id="2fe17-140">Назовите папку `Models`.</span><span class="sxs-lookup"><span data-stu-id="2fe17-140">Name the folder `Models`.</span></span>

1. <span data-ttu-id="2fe17-141">Щелкните правой кнопкой мыши папку **Models** и выберите команду **Добавить**, а затем — **класс...**. Назовите класс `NavMenuItem` и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-141">Right-click the **Models** folder and select **Add**, then **Class...**. Name the class `NavMenuItem` and select **Add**.</span></span>

1. <span data-ttu-id="2fe17-142">Откройте файл **NavMenuItem.CS** и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="2fe17-142">Open the **NavMenuItem.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. <span data-ttu-id="2fe17-143">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **новый элемент...**. Выберите **страницу содержимого** и присвойте странице `MenuPage`имя.</span><span class="sxs-lookup"><span data-stu-id="2fe17-143">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `MenuPage`.</span></span> <span data-ttu-id="2fe17-144">Нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-144">Select **Add**.</span></span>

1. <span data-ttu-id="2fe17-145">Откройте файл **менупаже. XAML** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="2fe17-145">Open the **MenuPage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml":::

1. <span data-ttu-id="2fe17-146">Разверните **менупаже. XAML** в **обозревателе решений** и откройте файл **MenuPage.XAML.CS** .</span><span class="sxs-lookup"><span data-stu-id="2fe17-146">Expand **MenuPage.xaml** in **Solution Explorer** and open the **MenuPage.xaml.cs** file.</span></span> <span data-ttu-id="2fe17-147">Замените его содержимое приведенным ниже.</span><span class="sxs-lookup"><span data-stu-id="2fe17-147">Replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > <span data-ttu-id="2fe17-148">Visual Studio сообщит об ошибках в **MenuPage.XAML.CS**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-148">Visual Studio will report errors in **MenuPage.xaml.cs**.</span></span> <span data-ttu-id="2fe17-149">Эти ошибки будут устранены на более позднем этапе.</span><span class="sxs-lookup"><span data-stu-id="2fe17-149">These errors will be resolved in a later step.</span></span>

#### <a name="implement-the-welcome-page"></a><span data-ttu-id="2fe17-150">Реализация страницы приветствия</span><span class="sxs-lookup"><span data-stu-id="2fe17-150">Implement the welcome page</span></span>

1. <span data-ttu-id="2fe17-151">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **новый элемент...**. Выберите **страницу содержимого** и присвойте странице `WelcomePage`имя.</span><span class="sxs-lookup"><span data-stu-id="2fe17-151">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `WelcomePage`.</span></span> <span data-ttu-id="2fe17-152">Нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-152">Select **Add**.</span></span> <span data-ttu-id="2fe17-153">Откройте файл **велкомепаже. XAML** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="2fe17-153">Open the **WelcomePage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml":::

1. <span data-ttu-id="2fe17-154">Разверните **велкомепаже. XAML** в **обозревателе решений** и откройте файл **WelcomePage.XAML.CS** .</span><span class="sxs-lookup"><span data-stu-id="2fe17-154">Expand **WelcomePage.xaml** in **Solution Explorer** and open the **WelcomePage.xaml.cs** file.</span></span> <span data-ttu-id="2fe17-155">Добавьте к классу `WelcomePage` следующую функцию:</span><span class="sxs-lookup"><span data-stu-id="2fe17-155">Add the following function to the `WelcomePage` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-page"></a><span data-ttu-id="2fe17-156">Страница "Добавление календаря"</span><span class="sxs-lookup"><span data-stu-id="2fe17-156">Add calendar page</span></span>

<span data-ttu-id="2fe17-157">Теперь добавьте страницу "Календарь".</span><span class="sxs-lookup"><span data-stu-id="2fe17-157">Now add a calendar page.</span></span> <span data-ttu-id="2fe17-158">Это будет просто заполнитель.</span><span class="sxs-lookup"><span data-stu-id="2fe17-158">This will just be a placeholder for now.</span></span>

1. <span data-ttu-id="2fe17-159">Щелкните правой кнопкой мыши проект **графтуториал** и выберите команду **Добавить**, а затем — **новый элемент...**. Выберите **страницу содержимого** и присвойте странице `CalendarPage`имя.</span><span class="sxs-lookup"><span data-stu-id="2fe17-159">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `CalendarPage`.</span></span> <span data-ttu-id="2fe17-160">Нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-160">Select **Add**.</span></span>

#### <a name="update-mainpage-code-behind"></a><span data-ttu-id="2fe17-161">Обновление кода MainPage</span><span class="sxs-lookup"><span data-stu-id="2fe17-161">Update MainPage code-behind</span></span>

<span data-ttu-id="2fe17-162">Теперь, когда все страницы находятся на месте, обновите код программной части для **MainPage. XAML**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-162">Now that all of the pages are in place, update the code-behind for **MainPage.xaml**.</span></span>

1. <span data-ttu-id="2fe17-163">Разверните **MainPage. XAML** в **обозревателе решений** и откройте файл **MainPage.XAML.CS** и замените все содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="2fe17-163">Expand **MainPage.xaml** in **Solution Explorer** and open the **MainPage.xaml.cs** file and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. <span data-ttu-id="2fe17-164">Сохраните все изменения.</span><span class="sxs-lookup"><span data-stu-id="2fe17-164">Save all of your changes.</span></span> <span data-ttu-id="2fe17-165">Щелкните правой кнопкой мыши проект, который требуется запустить (Android, iOS или UWP), и выберите пункт **Назначить запускаемым проектом**.</span><span class="sxs-lookup"><span data-stu-id="2fe17-165">Right-click the project that you want to run (Android, iOS, or UWP) and select **Set as StartUp Project**.</span></span> <span data-ttu-id="2fe17-166">Нажмите клавишу **F5** или выберите **Отладка > начать отладку** в Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="2fe17-166">Press **F5** or select **Debug > Start Debugging** in Visual Studio.</span></span>

    ![Снимки экрана с версиями приложения для Android, iOS и UWP](./images/welcome-page.png)
