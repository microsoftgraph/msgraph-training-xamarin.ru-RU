<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1cc0e-101">В этом упражнении вы расширит приложение от предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="1cc0e-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="1cc0e-103">На этом этапе вы интегрируете библиотеку проверки подлинности [Майкрософт для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

1. <span data-ttu-id="1cc0e-104">В **Обозревателе** решений разойдите **проект GraphTutorial** и щелкните правой кнопкой мыши **папку Models.**</span><span class="sxs-lookup"><span data-stu-id="1cc0e-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="1cc0e-105">Выберите **Добавить > класс...**. Назови класс `OAuthSettings` и выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-105">Select **Add > Class...**. Name the class `OAuthSettings` and select **Add**.</span></span>

1. <span data-ttu-id="1cc0e-106">Откройте **файл OAuthSettings.cs** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. <span data-ttu-id="1cc0e-107">Замените `YOUR_APP_ID_HERE` с помощью ID приложения из регистрации приложения.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-107">Replace `YOUR_APP_ID_HERE` with the application ID from your app registration.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="1cc0e-108">Если вы используете источник управления, например git, сейчас самое время исключить файл из источника управления, чтобы избежать случайной утечки вашего `OAuthSettings.cs` ID приложения.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-108">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="1cc0e-109">Реализация входа в систему</span><span class="sxs-lookup"><span data-stu-id="1cc0e-109">Implement sign-in</span></span>

1. <span data-ttu-id="1cc0e-110">Откройте **файл App.xaml.cs** в **проекте GraphTutorial** и добавьте в верхней части файла следующие `using` утверждения.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-110">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    using TimeZoneConverter;
    ```

1. <span data-ttu-id="1cc0e-111">Измените **строку объявления** класса App, чтобы разрешить конфликт имен для **приложения.**</span><span class="sxs-lookup"><span data-stu-id="1cc0e-111">Change the **App** class declaration line to resolve the name conflict for **Application**.</span></span>

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="1cc0e-112">Добавьте в класс следующие `App` свойства.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-112">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. <span data-ttu-id="1cc0e-113">Далее создайте новое `PublicClientApplication` в конструкторе `App` класса.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-113">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. <span data-ttu-id="1cc0e-114">`SignIn`Обнови функцию, чтобы использовать `PublicClientApplication` маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-114">Update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="1cc0e-115">Добавьте следующий код над `await GetUserInfo();` строкой.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-115">Add the following code above the `await GetUserInfo();` line.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    <span data-ttu-id="1cc0e-116">Этот код сначала пытается получить маркер доступа молча.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-116">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="1cc0e-117">Если сведения пользователя уже есть в кэше приложения (например, если пользователь закрыл приложение ранее без подписки), это будет успешно, и нет причин подсказывать пользователю.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-117">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="1cc0e-118">Если в кэше нет сведений пользователя, функция `AcquireTokenSilent().ExecuteAsync()` бросает `MsalUiRequiredException` .</span><span class="sxs-lookup"><span data-stu-id="1cc0e-118">If there is not a user's information in the cache, the `AcquireTokenSilent().ExecuteAsync()` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="1cc0e-119">В этом случае код вызывает интерактивную функцию для получения `AcquireTokenInteractive` маркера.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-119">In this case, the code calls the interactive function to get a token, `AcquireTokenInteractive`.</span></span>

1. <span data-ttu-id="1cc0e-120">`SignOut`Обновите функцию, чтобы удалить сведения пользователя из кэша.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-120">Update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="1cc0e-121">Добавьте следующий код в начало `SignOut` функции.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-121">Add the following code to the beginning of the `SignOut` function.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="1cc0e-122">Обновление проекта Android, чтобы включить вход</span><span class="sxs-lookup"><span data-stu-id="1cc0e-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="1cc0e-123">При его реализации в android-проекте Xamarin библиотека проверки подлинности Майкрософт имеет несколько [требований, определенных для Android.](/azure/active-directory/develop/msal-net-xamarin-android-considerations)</span><span class="sxs-lookup"><span data-stu-id="1cc0e-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](/azure/active-directory/develop/msal-net-xamarin-android-considerations).</span></span>

1. <span data-ttu-id="1cc0e-124">В **проекте GraphTutorial.Android** разложите папку **Свойства** и откройте **AndroidManifest.xml.**</span><span class="sxs-lookup"><span data-stu-id="1cc0e-124">In **GraphTutorial.Android** project, expand the **Properties** folder, then open **AndroidManifest.xml**.</span></span> <span data-ttu-id="1cc0e-125">Если вы используете Visual Studio для Mac, нажмите кнопку **AndroidManifest.xml** и выберите **Open With**, а затем **редактор исходных кодов**.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-125">If you're using Visual Studio for Mac, Control click **AndroidManifest.xml** and choose **Open With**, then **Source Code Editor**.</span></span> <span data-ttu-id="1cc0e-126">Замените все содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-126">Replace the entire contents with the following.</span></span>

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. <span data-ttu-id="1cc0e-127">Откройте **MainActivity.cs** и добавьте следующие утверждения `using` в верхнюю часть файла.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-127">Open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="1cc0e-128">Переопределять функцию, чтобы передать управление библиотеке `OnActivityResult` MSAL.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-128">Override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="1cc0e-129">Добавьте следующее в `MainActivity` класс.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-129">Add the following to the `MainActivity` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. <span data-ttu-id="1cc0e-130">В `OnCreate` функции добавьте следующую строку после `LoadApplication(new App());` строки.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-130">In the `OnCreate` function, add the following line after the `LoadApplication(new App());` line.</span></span>

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="1cc0e-131">Обновление проекта iOS, чтобы включить вход</span><span class="sxs-lookup"><span data-stu-id="1cc0e-131">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="1cc0e-132">Так как MSAL требует использования файла Entitlements.plist, необходимо настроить Visual Studio учетной записью разработчика Apple, чтобы включить подготовка.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-132">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="1cc0e-133">Если вы работаете с этим учебником в симуляторе iPhone, вам необходимо добавить **Entitlements.plist** в поле Custom **Entitlements** в параметрах **проекта GraphTutorial.iOS,** **build->iOS Bundle Signing**.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-133">If you are running this tutorial in the iPhone simulator, you need to add **Entitlements.plist** in the **Custom Entitlements** field in **GraphTutorial.iOS** project's settings, **Build->iOS Bundle Signing**.</span></span> <span data-ttu-id="1cc0e-134">Дополнительные сведения см. в [пункте Подготовка устройств для Xamarin.iOS.](/xamarin/ios/get-started/installation/device-provisioning)</span><span class="sxs-lookup"><span data-stu-id="1cc0e-134">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="1cc0e-135">При его реализации в проекте Xamarin iOS библиотека проверки подлинности Майкрософт имеет несколько требований, [определенных для iOS.](/azure/active-directory/develop/msal-net-xamarin-ios-considerations)</span><span class="sxs-lookup"><span data-stu-id="1cc0e-135">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](/azure/active-directory/develop/msal-net-xamarin-ios-considerations).</span></span>

1. <span data-ttu-id="1cc0e-136">В Обозревателе решений раз откройте **проект GraphTutorial.iOS,** а затем откройте файл **Entitlements.plist.**</span><span class="sxs-lookup"><span data-stu-id="1cc0e-136">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span>

1. <span data-ttu-id="1cc0e-137">Найдите **право Keychain** и выберите **Включить KeyChain.**</span><span class="sxs-lookup"><span data-stu-id="1cc0e-137">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span>

1. <span data-ttu-id="1cc0e-138">В **Keychain Groups** добавьте запись с форматом `com.companyname.GraphTutorial` .</span><span class="sxs-lookup"><span data-stu-id="1cc0e-138">In **Keychain Groups**, add an entry with the format `com.companyname.GraphTutorial`.</span></span>

    ![Снимок экрана конфигурации прав keychain](./images/enable-keychain-access.png)

1. <span data-ttu-id="1cc0e-140">Обновление кода в **проекте GraphTutorial.iOS** для обработки перенаправления во время проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-140">Update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="1cc0e-141">Откройте **файл AppDelegate.cs** и добавьте следующее утверждение `using` в верхней части файла.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-141">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="1cc0e-142">Добавьте следующую строку `FinishedLaunching` для работы перед `LoadApplication(new App());` строкой.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-142">Add the following line to `FinishedLaunching` function just before the `LoadApplication(new App());` line.</span></span>

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. <span data-ttu-id="1cc0e-143">Переопределять `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-143">Override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="1cc0e-144">Добавьте следующее в `AppDelegate` класс.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-144">Add the following to the `AppDelegate` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a><span data-ttu-id="1cc0e-145">Хранение маркеров</span><span class="sxs-lookup"><span data-stu-id="1cc0e-145">Storing the tokens</span></span>

<span data-ttu-id="1cc0e-146">Когда библиотека проверки подлинности Майкрософт используется в проекте Xamarin, она по умолчанию используется для кэширования маркеров на родном безопасном хранилище.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-146">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="1cc0e-147">Дополнительные сведения см. в дополнительных [сведениях.](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization)</span><span class="sxs-lookup"><span data-stu-id="1cc0e-147">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="1cc0e-148">Тестовый вход</span><span class="sxs-lookup"><span data-stu-id="1cc0e-148">Test sign-in</span></span>

<span data-ttu-id="1cc0e-149">В этот момент при запуске приложения и нажатии кнопки **Вход** в кнопку, вам будет предложено войти.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-149">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="1cc0e-150">При успешном входе вы должны увидеть маркер доступа, напечатанный на выходе отладки.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-150">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Снимок экрана окна Выход в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="1cc0e-152">Получение сведений о пользователе</span><span class="sxs-lookup"><span data-stu-id="1cc0e-152">Get user details</span></span>

1. <span data-ttu-id="1cc0e-153">Добавьте новую функцию в **класс App,** чтобы инициализировать `GraphServiceClient` .</span><span class="sxs-lookup"><span data-stu-id="1cc0e-153">Add a new function to the **App** class to initialize the `GraphServiceClient`.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. <span data-ttu-id="1cc0e-154">`SignIn`Обнови функцию **в App.xaml.cs,** чтобы вызвать эту функцию вместо `GetUserInfo` .</span><span class="sxs-lookup"><span data-stu-id="1cc0e-154">Update the `SignIn` function in **App.xaml.cs** to call this function instead of `GetUserInfo`.</span></span> <span data-ttu-id="1cc0e-155">Удалите следующее из `SignIn` функции.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-155">Remove the following from the `SignIn` function.</span></span>

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. <span data-ttu-id="1cc0e-156">Добавьте следующее в конец `SignIn` функции.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-156">Add the following to the end of the `SignIn` function.</span></span>

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. <span data-ttu-id="1cc0e-157">Обнови функцию, чтобы получить сведения пользователя `GetUserInfo` из microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-157">Update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="1cc0e-158">Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-158">Replace the existing `GetUserInfo` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. <span data-ttu-id="1cc0e-159">Сохраните изменения и запустите приложение.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-159">Save your changes and run the app.</span></span> <span data-ttu-id="1cc0e-160">После регистрации пользовательский интерфейс обновляется с именем и адресом электронной почты пользователя.</span><span class="sxs-lookup"><span data-stu-id="1cc0e-160">After sign-in the UI is updated with the user's display name and email address.</span></span>
