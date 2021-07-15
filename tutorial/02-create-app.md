<!-- markdownlint-disable MD002 MD041 -->

1. <span data-ttu-id="9c2b2-101">Откройте Visual Studio и нажмите **Создать новый проект**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-101">Open Visual Studio, and select **Create a new project**.</span></span>

1. <span data-ttu-id="9c2b2-102">В **диалоговом окте "Создание нового** проекта" выберите мобильное **приложение (Xamarin.Forms)** и выберите **Далее**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-102">In the **Create a new project** dialog, choose **Mobile App (Xamarin.Forms)**, then select **Next**.</span></span>

    ![Visual Studio 2019 г. создайте новый диалоговое окно проекта](images/new-project-dialog.png)

1. <span data-ttu-id="9c2b2-104">В **диалоговом** октаге Настройка нового проекта введите имя Project и имя решения, а затем `GraphTutorial` выберите **Создать**.  </span><span class="sxs-lookup"><span data-stu-id="9c2b2-104">In the **Configure a new project** dialog, enter `GraphTutorial` for the **Project name** and **Solution name**, then select **Create**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="9c2b2-105">Убедитесь, что вы вводите точно такое же имя для Visual Studio Project, указанного в этих лабораторных инструкциях.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-105">Ensure that you enter the exact same name for the Visual Studio Project that is specified in these lab instructions.</span></span> <span data-ttu-id="9c2b2-106">Имя проекта Visual Studio становится частью пространства имен в коде.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-106">The Visual Studio Project name becomes part of the namespace in the code.</span></span> <span data-ttu-id="9c2b2-107">Для кода в этих инструкциях важно, чтобы пространство имен совпадало с именем проекта Visual Studio, указанным в этих инструкциях.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-107">The code inside these instructions depends on the namespace matching the Visual Studio Project name specified in these instructions.</span></span> <span data-ttu-id="9c2b2-108">Если ввести другое имя проекта, код не будет компилироваться, пока вы не настроите все пространства имен в соответствии с именем, которое вы указали для проекта Visual Studio при его создании.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-108">If you use a different project name the code will not compile unless you adjust all the namespaces to match the Visual Studio Project name you enter when you create the project.</span></span>

    ![Visual Studio 2019 г. настройте новый диалоговое окно проекта](images/configure-new-project-dialog.png)

1. <span data-ttu-id="9c2b2-110">В **диалоговом** окланте  New Cross Platform App выберите пустой шаблон и выберите платформы, которые необходимо создать в **рамках Платформы.**</span><span class="sxs-lookup"><span data-stu-id="9c2b2-110">In the **New Cross Platform App** dialog, select the **Blank** template, and select the platforms you want to build under **Platforms**.</span></span> <span data-ttu-id="9c2b2-111">Выберите **ОК** для создания решения.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-111">Select **OK** to create the solution.</span></span>

    ![Visual Studio 2019 новый диалоговое окно приложения для кросс-платформы](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a><span data-ttu-id="9c2b2-113">Установка пакетов</span><span class="sxs-lookup"><span data-stu-id="9c2b2-113">Install packages</span></span>

<span data-ttu-id="9c2b2-114">Прежде чем двигаться дальше, установите дополнительные NuGet пакеты, которые вы будете использовать позже.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-114">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="9c2b2-115">[Microsoft.Identity.Client для](https://www.nuget.org/packages/Microsoft.Identity.Client/) обработки проверки подлинности Azure AD и управления маркерами.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-115">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) to handle Azure AD authentication and token management.</span></span>
- <span data-ttu-id="9c2b2-116">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) для звонков в microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-116">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to the Microsoft Graph.</span></span>
- <span data-ttu-id="9c2b2-117">[TimeZoneConverter для](https://www.nuget.org/packages/TimeZoneConverter/) обработки часовых поясов поперек платформы.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-117">[TimeZoneConverter](https://www.nuget.org/packages/TimeZoneConverter/) for handling time zones cross-platform.</span></span>

1. <span data-ttu-id="9c2b2-118">Выберите **Сервис > Диспетчер пакетов NuGet > Консоль диспетчера пакетов**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-118">Select **Tools > NuGet Package Manager > Package Manager Console**.</span></span>

1. <span data-ttu-id="9c2b2-119">В консоли диспетчера пакетов введите следующие команды.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-119">In the Package Manager Console, enter the following commands.</span></span>

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.34.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.34.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.34.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 4.0.0 -Project GraphTutorial
    Install-Package TimeZoneConverter -Project GraphTutorial
    ```

## <a name="design-the-app"></a><span data-ttu-id="9c2b2-120">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="9c2b2-120">Design the app</span></span>

<span data-ttu-id="9c2b2-121">Начните с обновления класса, чтобы добавить переменные для отслеживания состояния проверки подлинности и `App` подписанного пользователя.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-121">Start by updating the `App` class to add variables to track the authentication state and the signed-in user.</span></span>

1. <span data-ttu-id="9c2b2-122">В **Обозревателе** решений расширь **проект GraphTutorial,** а затем расширь **файл App.xaml.**</span><span class="sxs-lookup"><span data-stu-id="9c2b2-122">In **Solution Explorer**, expand the **GraphTutorial** project, then expand the **App.xaml** file.</span></span> <span data-ttu-id="9c2b2-123">Откройте **файл App.xaml.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-123">Open the **App.xaml.cs** file and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using System.ComponentModel;
    using System.IO;
    using System.Reflection;
    using System.Threading.Tasks;
    ```

1. <span data-ttu-id="9c2b2-124">Добавьте интерфейс `INotifyPropertyChanged` в объявление класса.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-124">Add the `INotifyPropertyChanged` interface to the class declaration.</span></span>

    ```csharp
    public partial class App : Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="9c2b2-125">Добавьте в класс следующие `App` свойства.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-125">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GlobalPropertiesSnippet":::

1. <span data-ttu-id="9c2b2-126">Добавьте в класс следующие `App` функции.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-126">Add the following functions to the `App` class.</span></span> <span data-ttu-id="9c2b2-127">, `SignIn` `SignOut` и `GetUserInfo` функции являются только placeholders на данный момент.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-127">The `SignIn`, `SignOut`, and `GetUserInfo` functions are just placeholders for now.</span></span>

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

1. <span data-ttu-id="9c2b2-128">Функция `GetUserPhoto` возвращает фотографию по умолчанию на данный момент.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-128">The `GetUserPhoto` function returns a default photo for now.</span></span> <span data-ttu-id="9c2b2-129">Вы можете либо предоставить собственный файл, либо скачать тот, который используется в примере [из GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png)</span><span class="sxs-lookup"><span data-stu-id="9c2b2-129">You can either supply your own file, or you can download the one used in the sample from [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span></span> <span data-ttu-id="9c2b2-130">Если вы используете собственный файл, переименуйте его в **no-profile-pic.png**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-130">If you use your own file, rename it to **no-profile-pic.png**.</span></span>

1. <span data-ttu-id="9c2b2-131">Скопируйте файл в **каталог ./GraphTutorial/GraphTutorial.**</span><span class="sxs-lookup"><span data-stu-id="9c2b2-131">Copy the file to the **./GraphTutorial/GraphTutorial** directory.</span></span>

1. <span data-ttu-id="9c2b2-132">Щелкните правой кнопкой мыши файл в **обозревателе решений** и выберите **Свойства**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-132">Right-click the file in **Solution Explorer** and select **Properties**.</span></span> <span data-ttu-id="9c2b2-133">В **окне Свойства** измените значение **Действия сборки** на **встроенный ресурс.**</span><span class="sxs-lookup"><span data-stu-id="9c2b2-133">In the **Properties** window, change the value of **Build Action** to **Embedded resource**.</span></span>

    ![Снимок экрана окна Свойства для PNG-файла](./images/png-file-properties.png)

### <a name="app-navigation"></a><span data-ttu-id="9c2b2-135">Навигация по приложениям</span><span class="sxs-lookup"><span data-stu-id="9c2b2-135">App navigation</span></span>

<span data-ttu-id="9c2b2-136">В этом разделе вы измените главную страницу приложения на [FlyoutPage.](/xamarin/xamarin-forms/app-fundamentals/navigation/flyoutpage)</span><span class="sxs-lookup"><span data-stu-id="9c2b2-136">In this section, you'll change the application's main page to a [FlyoutPage](/xamarin/xamarin-forms/app-fundamentals/navigation/flyoutpage).</span></span> <span data-ttu-id="9c2b2-137">Это обеспечит меню навигации для переключения между представлением в приложении.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-137">This will provide a navigation menu to switch between view in the app.</span></span>

1. <span data-ttu-id="9c2b2-138">Откройте **файл MainPage.xaml** в **проекте GraphTutorial** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-138">Open the **MainPage.xaml** file in the **GraphTutorial** project and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml":::

#### <a name="implement-the-menu"></a><span data-ttu-id="9c2b2-139">Реализация меню</span><span class="sxs-lookup"><span data-stu-id="9c2b2-139">Implement the menu</span></span>

1. <span data-ttu-id="9c2b2-140">Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем **новую папку**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-140">Right-click the **GraphTutorial** project and select **Add**, then **New Folder**.</span></span> <span data-ttu-id="9c2b2-141">Назови папку `Models` .</span><span class="sxs-lookup"><span data-stu-id="9c2b2-141">Name the folder `Models`.</span></span>

1. <span data-ttu-id="9c2b2-142">Щелкните правой кнопкой мыши **папку Модели** и выберите **Добавить**, а затем **класс ...**. Назови класс `NavMenuItem` и выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-142">Right-click the **Models** folder and select **Add**, then **Class...**. Name the class `NavMenuItem` and select **Add**.</span></span>

1. <span data-ttu-id="9c2b2-143">Откройте **файл NavMenuItem.cs** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-143">Open the **NavMenuItem.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. <span data-ttu-id="9c2b2-144">Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `MenuPage` .</span><span class="sxs-lookup"><span data-stu-id="9c2b2-144">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `MenuPage`.</span></span> <span data-ttu-id="9c2b2-145">Нажмите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-145">Select **Add**.</span></span>

1. <span data-ttu-id="9c2b2-146">Откройте файл **MenuPage.xaml** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-146">Open the **MenuPage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml":::

1. <span data-ttu-id="9c2b2-147">**Расширь menuPage.xaml** в **обозревателе решений** и откройте **файл MenuPage.xaml.cs.**</span><span class="sxs-lookup"><span data-stu-id="9c2b2-147">Expand **MenuPage.xaml** in **Solution Explorer** and open the **MenuPage.xaml.cs** file.</span></span> <span data-ttu-id="9c2b2-148">Замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-148">Replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > <span data-ttu-id="9c2b2-149">Visual Studio будет сообщать об ошибках **в MenuPage.xaml.cs**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-149">Visual Studio will report errors in **MenuPage.xaml.cs**.</span></span> <span data-ttu-id="9c2b2-150">Эти ошибки будут устранены на более позднем этапе.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-150">These errors will be resolved in a later step.</span></span>

#### <a name="implement-the-welcome-page"></a><span data-ttu-id="9c2b2-151">Реализация страницы приветствия</span><span class="sxs-lookup"><span data-stu-id="9c2b2-151">Implement the welcome page</span></span>

1. <span data-ttu-id="9c2b2-152">Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `WelcomePage` .</span><span class="sxs-lookup"><span data-stu-id="9c2b2-152">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `WelcomePage`.</span></span> <span data-ttu-id="9c2b2-153">Нажмите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-153">Select **Add**.</span></span> <span data-ttu-id="9c2b2-154">Откройте файл **WelcomePage.xaml** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-154">Open the **WelcomePage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml":::

1. <span data-ttu-id="9c2b2-155">**Расширь welcomePage.xaml** в **обозревателе** решений и откройте **файл WelcomePage.xaml.cs.**</span><span class="sxs-lookup"><span data-stu-id="9c2b2-155">Expand **WelcomePage.xaml** in **Solution Explorer** and open the **WelcomePage.xaml.cs** file.</span></span> <span data-ttu-id="9c2b2-156">Добавьте к классу `WelcomePage` следующую функцию:</span><span class="sxs-lookup"><span data-stu-id="9c2b2-156">Add the following function to the `WelcomePage` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-and-new-event-pages"></a><span data-ttu-id="9c2b2-157">Добавление страниц календаря и новых событий</span><span class="sxs-lookup"><span data-stu-id="9c2b2-157">Add calendar and new event pages</span></span>

<span data-ttu-id="9c2b2-158">Теперь добавьте страницу календаря и новую страницу событий.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-158">Now add a calendar page and a new event page.</span></span> <span data-ttu-id="9c2b2-159">Они будут просто местообнажителей на данный момент.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-159">These will just be placeholders for now.</span></span>

1. <span data-ttu-id="9c2b2-160">Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `CalendarPage` .</span><span class="sxs-lookup"><span data-stu-id="9c2b2-160">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `CalendarPage`.</span></span> <span data-ttu-id="9c2b2-161">Нажмите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-161">Select **Add**.</span></span>

1. <span data-ttu-id="9c2b2-162">Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `NewEventPage` .</span><span class="sxs-lookup"><span data-stu-id="9c2b2-162">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `NewEventPage`.</span></span> <span data-ttu-id="9c2b2-163">Нажмите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-163">Select **Add**.</span></span>

#### <a name="update-mainpage-code-behind"></a><span data-ttu-id="9c2b2-164">Обновление кода MainPage</span><span class="sxs-lookup"><span data-stu-id="9c2b2-164">Update MainPage code-behind</span></span>

<span data-ttu-id="9c2b2-165">Теперь, когда все страницы на месте, обнови код-за для **MainPage.xaml**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-165">Now that all of the pages are in place, update the code-behind for **MainPage.xaml**.</span></span>

1. <span data-ttu-id="9c2b2-166">**Разместим MainPage.xaml** в обозревателе решений и откройте **файл MainPage.xaml.cs** и замените все содержимое следующим образом. </span><span class="sxs-lookup"><span data-stu-id="9c2b2-166">Expand **MainPage.xaml** in **Solution Explorer** and open the **MainPage.xaml.cs** file and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. <span data-ttu-id="9c2b2-167">Сохраните все изменения.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-167">Save all of your changes.</span></span> <span data-ttu-id="9c2b2-168">Щелкните правой кнопкой мыши проект, который вы хотите запустить (Android, iOS или UWP) и выберите **Set как startUp Project**.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-168">Right-click the project that you want to run (Android, iOS, or UWP) and select **Set as StartUp Project**.</span></span> <span data-ttu-id="9c2b2-169">Нажмите **кнопку F5** или **выберите отладку > начать отладку** в Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="9c2b2-169">Press **F5** or select **Debug > Start Debugging** in Visual Studio.</span></span>

    ![Скриншоты версий приложения для Android, iOS и UWP](./images/welcome-page.png)
