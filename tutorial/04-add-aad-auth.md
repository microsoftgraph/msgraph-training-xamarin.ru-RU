<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="26301-101">В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="26301-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="26301-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="26301-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="26301-103">На этом этапе вы интегрируете библиотеку проверки подлинности [Майкрософт для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.</span><span class="sxs-lookup"><span data-stu-id="26301-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

1. <span data-ttu-id="26301-104">В **обозревателе решений** разйдите проект **GraphTutorial** и щелкните правой кнопкой мыши **папку Models.**</span><span class="sxs-lookup"><span data-stu-id="26301-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="26301-105">Выберите **"Добавить > класс..."** Назовем класс `OAuthSettings` и выберите **"Добавить".**</span><span class="sxs-lookup"><span data-stu-id="26301-105">Select **Add > Class...**. Name the class `OAuthSettings` and select **Add**.</span></span>

1. <span data-ttu-id="26301-106">Откройте файл **OAuthSettings.cs** и замените его содержимое на следующий файл.</span><span class="sxs-lookup"><span data-stu-id="26301-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. <span data-ttu-id="26301-107">Замените `YOUR_APP_ID_HERE` его на ИД приложения из регистрации приложения.</span><span class="sxs-lookup"><span data-stu-id="26301-107">Replace `YOUR_APP_ID_HERE` with the application ID from your app registration.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="26301-108">Если вы используете управление исходным кодом, например git, пришло бы время исключить файл из системы управления исходным кодом, чтобы не допустить случайной утечки вашего `OAuthSettings.cs` ИД приложения.</span><span class="sxs-lookup"><span data-stu-id="26301-108">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="26301-109">Реализация входов</span><span class="sxs-lookup"><span data-stu-id="26301-109">Implement sign-in</span></span>

1. <span data-ttu-id="26301-110">Откройте файл **App.xaml.cs** в **проекте GraphTutorial** и добавьте следующие утверждения в `using` верхнюю часть файла.</span><span class="sxs-lookup"><span data-stu-id="26301-110">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    using TimeZoneConverter;
    ```

1. <span data-ttu-id="26301-111">Измените **строку объявления** класса App, чтобы устранить конфликт имен **для Приложения.**</span><span class="sxs-lookup"><span data-stu-id="26301-111">Change the **App** class declaration line to resolve the name conflict for **Application**.</span></span>

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="26301-112">Добавьте в класс следующие `App` свойства.</span><span class="sxs-lookup"><span data-stu-id="26301-112">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. <span data-ttu-id="26301-113">Затем создайте новый `PublicClientApplication` в конструкторе `App` класса.</span><span class="sxs-lookup"><span data-stu-id="26301-113">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. <span data-ttu-id="26301-114">`SignIn`Обновим функцию, чтобы использовать `PublicClientApplication` ее для получения маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="26301-114">Update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="26301-115">Добавьте следующий код над `await GetUserInfo();` строкой.</span><span class="sxs-lookup"><span data-stu-id="26301-115">Add the following code above the `await GetUserInfo();` line.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    <span data-ttu-id="26301-116">Этот код сначала пытается получить маркер доступа в тихом режиме.</span><span class="sxs-lookup"><span data-stu-id="26301-116">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="26301-117">Если сведения о пользователе уже есть в кэше приложения (например, если пользователь ранее закрыл приложение без выходных данных), это будет успешно, и нет причин для запроса пользователя.</span><span class="sxs-lookup"><span data-stu-id="26301-117">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="26301-118">Если в кэше нет сведений о пользователе, функция `AcquireTokenSilent().ExecuteAsync()` вырлает `MsalUiRequiredException` исключение.</span><span class="sxs-lookup"><span data-stu-id="26301-118">If there is not a user's information in the cache, the `AcquireTokenSilent().ExecuteAsync()` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="26301-119">В этом случае код вызывает интерактивную функцию для получения `AcquireTokenInteractive` маркера.</span><span class="sxs-lookup"><span data-stu-id="26301-119">In this case, the code calls the interactive function to get a token, `AcquireTokenInteractive`.</span></span>

1. <span data-ttu-id="26301-120">`SignOut`Обновите функцию, чтобы удалить данные пользователя из кэша.</span><span class="sxs-lookup"><span data-stu-id="26301-120">Update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="26301-121">Добавьте следующий код в начало `SignOut` функции.</span><span class="sxs-lookup"><span data-stu-id="26301-121">Add the following code to the beginning of the `SignOut` function.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="26301-122">Обновление проекта Android, чтобы включить вход</span><span class="sxs-lookup"><span data-stu-id="26301-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="26301-123">При его использования в проекте Xamarin для Android библиотека проверки подлинности Майкрософт предъявляет несколько [требований, характерных для Android.](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics)</span><span class="sxs-lookup"><span data-stu-id="26301-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

1. <span data-ttu-id="26301-124">В **проекте GraphTutorial.Android** разкроте папку **"Свойства",** а затем откройте **AndroidManifest.xml.**</span><span class="sxs-lookup"><span data-stu-id="26301-124">In **GraphTutorial.Android** project, expand the **Properties** folder, then open **AndroidManifest.xml**.</span></span> <span data-ttu-id="26301-125">Если вы используете Visual Studio Mac, выберите "AndroidManifest.xml" и "Открыть с помощью" и "Редактор **исходных кодов".** </span><span class="sxs-lookup"><span data-stu-id="26301-125">If you're using Visual Studio for Mac, Control click **AndroidManifest.xml** and choose **Open With**, then **Source Code Editor**.</span></span> <span data-ttu-id="26301-126">Замените все содержимое на следующее.</span><span class="sxs-lookup"><span data-stu-id="26301-126">Replace the entire contents with the following.</span></span>

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. <span data-ttu-id="26301-127">Откройте **MainActivity.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.</span><span class="sxs-lookup"><span data-stu-id="26301-127">Open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="26301-128">Переопределять `OnActivityResult` функцию, чтобы передать управление библиотеке MSAL.</span><span class="sxs-lookup"><span data-stu-id="26301-128">Override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="26301-129">Добавьте в класс `MainActivity` следующее:</span><span class="sxs-lookup"><span data-stu-id="26301-129">Add the following to the `MainActivity` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. <span data-ttu-id="26301-130">Добавьте `OnCreate` в функцию следующую строку после `LoadApplication(new App());` строки.</span><span class="sxs-lookup"><span data-stu-id="26301-130">In the `OnCreate` function, add the following line after the `LoadApplication(new App());` line.</span></span>

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="26301-131">Обновление проекта iOS для реализации входов</span><span class="sxs-lookup"><span data-stu-id="26301-131">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="26301-132">Так как MSAL требует использования файла Entitlements.plist, необходимо настроить Visual Studio с помощью учетной записи разработчика Apple, чтобы включить ее.</span><span class="sxs-lookup"><span data-stu-id="26301-132">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="26301-133">Если вы работаете с этим учебником в имитаторе iPhone, вам потребуется добавить **Entitlements.plist** в поле "Пользовательские права" в параметрах проекта **GraphTutorial.iOS,** подписывая пакеты **>iOS.** </span><span class="sxs-lookup"><span data-stu-id="26301-133">If you are running this tutorial in the iPhone simulator, you need to add **Entitlements.plist** in the **Custom Entitlements** field in **GraphTutorial.iOS** project's settings, **Build->iOS Bundle Signing**.</span></span> <span data-ttu-id="26301-134">Дополнительные сведения [см. в подгонке "Подготовка устройств для Xamarin.iOS".](/xamarin/ios/get-started/installation/device-provisioning)</span><span class="sxs-lookup"><span data-stu-id="26301-134">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="26301-135">При этом в проекте Xamarin для iOS библиотека проверки подлинности Майкрософт предъявляет несколько [требований, характерных для iOS.](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics)</span><span class="sxs-lookup"><span data-stu-id="26301-135">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

1. <span data-ttu-id="26301-136">В обозревателе решений раз откройте проект **GraphTutorial.iOS** и откройте файл **Entitlements.plist.**</span><span class="sxs-lookup"><span data-stu-id="26301-136">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span>

1. <span data-ttu-id="26301-137">Найдите **право на получение ключей** и выберите **"Включить keyChain".**</span><span class="sxs-lookup"><span data-stu-id="26301-137">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span>

1. <span data-ttu-id="26301-138">В **группах ключей** добавьте запись в формате. `com.companyname.GraphTutorial`</span><span class="sxs-lookup"><span data-stu-id="26301-138">In **Keychain Groups**, add an entry with the format `com.companyname.GraphTutorial`.</span></span>

    ![Снимок экрана: конфигурация прав на ключ](./images/enable-keychain-access.png)

1. <span data-ttu-id="26301-140">Обновите код в **проекте GraphTutorial.iOS,** чтобы обработать перенаправление во время проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="26301-140">Update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="26301-141">Откройте **AppDelegate.cs** файла и добавьте в верхнюю часть файла следующий `using` выписку.</span><span class="sxs-lookup"><span data-stu-id="26301-141">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="26301-142">Добавьте следующую строку для `FinishedLaunching` работы прямо перед `LoadApplication(new App());` строкой.</span><span class="sxs-lookup"><span data-stu-id="26301-142">Add the following line to `FinishedLaunching` function just before the `LoadApplication(new App());` line.</span></span>

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. <span data-ttu-id="26301-143">Переопределять `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="26301-143">Override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="26301-144">Добавьте в класс `AppDelegate` следующее:</span><span class="sxs-lookup"><span data-stu-id="26301-144">Add the following to the `AppDelegate` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a><span data-ttu-id="26301-145">Хранение маркеров</span><span class="sxs-lookup"><span data-stu-id="26301-145">Storing the tokens</span></span>

<span data-ttu-id="26301-146">Когда библиотека проверки подлинности Майкрософт используется в проекте Xamarin, она по умолчанию кэширование маркеров обеспечивается с помощью нативного безопасного хранилища.</span><span class="sxs-lookup"><span data-stu-id="26301-146">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="26301-147">Дополнительные [сведения см. в](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) сведениях о сериализации кэша маркеров.</span><span class="sxs-lookup"><span data-stu-id="26301-147">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="26301-148">Тестовый вход</span><span class="sxs-lookup"><span data-stu-id="26301-148">Test sign-in</span></span>

<span data-ttu-id="26301-149">На этом этапе, если запустить  приложение и нажать кнопку "Войти", вам будет предложено выполнить вход.</span><span class="sxs-lookup"><span data-stu-id="26301-149">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="26301-150">При успешном входе маркер доступа должен отпечатываться в выходных данных отладка.</span><span class="sxs-lookup"><span data-stu-id="26301-150">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Снимок экрана с окном вывода в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="26301-152">Получить сведения о пользователе</span><span class="sxs-lookup"><span data-stu-id="26301-152">Get user details</span></span>

1. <span data-ttu-id="26301-153">Добавьте новую функцию в **класс App** для инициализации `GraphServiceClient` .</span><span class="sxs-lookup"><span data-stu-id="26301-153">Add a new function to the **App** class to initialize the `GraphServiceClient`.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. <span data-ttu-id="26301-154">`SignIn`Обновим функцию **в App.xaml.cs,** чтобы она вызывала эту функцию вместо `GetUserInfo` .</span><span class="sxs-lookup"><span data-stu-id="26301-154">Update the `SignIn` function in **App.xaml.cs** to call this function instead of `GetUserInfo`.</span></span> <span data-ttu-id="26301-155">Удалите из функции `SignIn` следующее:</span><span class="sxs-lookup"><span data-stu-id="26301-155">Remove the following from the `SignIn` function.</span></span>

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. <span data-ttu-id="26301-156">Добавьте следующее в конец `SignIn` функции.</span><span class="sxs-lookup"><span data-stu-id="26301-156">Add the following to the end of the `SignIn` function.</span></span>

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. <span data-ttu-id="26301-157">`GetUserInfo`Обновим функцию, чтобы получить сведения о пользователе из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="26301-157">Update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="26301-158">Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="26301-158">Replace the existing `GetUserInfo` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. <span data-ttu-id="26301-159">Сохраните изменения и запустите приложение.</span><span class="sxs-lookup"><span data-stu-id="26301-159">Save your changes and run the app.</span></span> <span data-ttu-id="26301-160">После того как пользовательский интерфейс будет обновлен с помощью отображаемого имени пользователя и адреса электронной почты.</span><span class="sxs-lookup"><span data-stu-id="26301-160">After sign-in the UI is updated with the user's display name and email address.</span></span>
