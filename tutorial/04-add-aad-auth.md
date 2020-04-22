<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4aae0-101">В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="4aae0-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="4aae0-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="4aae0-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="4aae0-103">На этом этапе выполняется интеграция [библиотеки проверки подлинности Microsoft для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.</span><span class="sxs-lookup"><span data-stu-id="4aae0-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

1. <span data-ttu-id="4aae0-104">В **обозревателе решений**разверните проект **графтуториал** и щелкните правой кнопкой мыши папку **Models** .</span><span class="sxs-lookup"><span data-stu-id="4aae0-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="4aae0-105">Выберите **Добавить класс >..**.. Назовите класс `OAuthSettings` и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="4aae0-105">Select **Add > Class...**. Name the class `OAuthSettings` and select **Add**.</span></span>

1. <span data-ttu-id="4aae0-106">Откройте файл **OAuthSettings.CS** и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="4aae0-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. <span data-ttu-id="4aae0-107">Замените `YOUR_APP_ID_HERE` идентификатором приложения из регистрации приложения.</span><span class="sxs-lookup"><span data-stu-id="4aae0-107">Replace `YOUR_APP_ID_HERE` with the application ID from your app registration.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="4aae0-108">Если вы используете систему управления версиями (например, Git), то теперь мы бы не могли исключить `OAuthSettings.cs` файл из системы управления версиями, чтобы избежать случайной утечки идентификатора приложения.</span><span class="sxs-lookup"><span data-stu-id="4aae0-108">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="4aae0-109">Реализация входа</span><span class="sxs-lookup"><span data-stu-id="4aae0-109">Implement sign-in</span></span>

1. <span data-ttu-id="4aae0-110">Откройте файл **app.XAML.CS** в проекте **графтуториал** и добавьте приведенные ниже `using` операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="4aae0-110">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    ```

1. <span data-ttu-id="4aae0-111">Измените строку объявления класса **приложения** , чтобы разрешить конфликт имен для **приложения**.</span><span class="sxs-lookup"><span data-stu-id="4aae0-111">Change the **App** class declaration line to resolve the name conflict for **Application**.</span></span>

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="4aae0-112">Добавьте в `App` класс указанные ниже свойства.</span><span class="sxs-lookup"><span data-stu-id="4aae0-112">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. <span data-ttu-id="4aae0-113">Затем создайте новый `PublicClientApplication` элемент в конструкторе `App` класса.</span><span class="sxs-lookup"><span data-stu-id="4aae0-113">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. <span data-ttu-id="4aae0-114">Обновите `SignIn` функцию, чтобы она `PublicClientApplication` использовалась для получения маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="4aae0-114">Update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="4aae0-115">Добавьте приведенный ниже код перед `await GetUserInfo();` строкой.</span><span class="sxs-lookup"><span data-stu-id="4aae0-115">Add the following code above the `await GetUserInfo();` line.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    <span data-ttu-id="4aae0-116">Этот код сначала пытается получить маркер доступа в автоматическом режиме.</span><span class="sxs-lookup"><span data-stu-id="4aae0-116">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="4aae0-117">Если сведения о пользователе уже находятся в кэше приложения (например, если пользователь закрыл приложение раньше, без выхода), это будет успешным, и нет причин запрашивать пользователя.</span><span class="sxs-lookup"><span data-stu-id="4aae0-117">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="4aae0-118">Если в кэше нет сведений о пользователе, `AcquireTokenSilent().ExecuteAsync()` функция создает исключение. `MsalUiRequiredException`</span><span class="sxs-lookup"><span data-stu-id="4aae0-118">If there is not a user's information in the cache, the `AcquireTokenSilent().ExecuteAsync()` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="4aae0-119">В этом случае код вызывает интерактивную функцию для получения маркера `AcquireTokenInteractive`.</span><span class="sxs-lookup"><span data-stu-id="4aae0-119">In this case, the code calls the interactive function to get a token, `AcquireTokenInteractive`.</span></span>

1. <span data-ttu-id="4aae0-120">Обновите `SignOut` функцию, чтобы удалить сведения о пользователе из кэша.</span><span class="sxs-lookup"><span data-stu-id="4aae0-120">Update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="4aae0-121">Добавьте следующий код в начало `SignOut` функции.</span><span class="sxs-lookup"><span data-stu-id="4aae0-121">Add the following code to the beginning of the `SignOut` function.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="4aae0-122">Обновление проекта Android для включения входа</span><span class="sxs-lookup"><span data-stu-id="4aae0-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="4aae0-123">При использовании в проекте Xamarin Android Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span><span class="sxs-lookup"><span data-stu-id="4aae0-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

1. <span data-ttu-id="4aae0-124">В проекте **графтуториал. Android** разверните папку **свойства** , а затем откройте **файл AndroidManifest. XML**.</span><span class="sxs-lookup"><span data-stu-id="4aae0-124">In **GraphTutorial.Android** project, expand the **Properties** folder, then open **AndroidManifest.xml**.</span></span> <span data-ttu-id="4aae0-125">Если вы используете Visual Studio для Mac, нажмите кнопку **AndroidManifest. XML** и выберите **Открыть с помощью** **редактора исходного кода**.</span><span class="sxs-lookup"><span data-stu-id="4aae0-125">If you're using Visual Studio for Mac, Control click **AndroidManifest.xml** and choose **Open With**, then **Source Code Editor**.</span></span> <span data-ttu-id="4aae0-126">Замените все содержимое приведенным ниже содержимым.</span><span class="sxs-lookup"><span data-stu-id="4aae0-126">Replace the entire contents with the following.</span></span>

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. <span data-ttu-id="4aae0-127">Откройте **MainActivity.CS** и добавьте приведенные `using` ниже операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="4aae0-127">Open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="4aae0-128">Переопределите `OnActivityResult` функцию, чтобы передать управление в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="4aae0-128">Override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="4aae0-129">Добавьте в `MainActivity` класс следующее:</span><span class="sxs-lookup"><span data-stu-id="4aae0-129">Add the following to the `MainActivity` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. <span data-ttu-id="4aae0-130">В `OnCreate` функции добавьте следующую строку после `LoadApplication(new App());` строки.</span><span class="sxs-lookup"><span data-stu-id="4aae0-130">In the `OnCreate` function, add the following line after the `LoadApplication(new App());` line.</span></span>

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="4aae0-131">Обновление проекта iOS для включения входа</span><span class="sxs-lookup"><span data-stu-id="4aae0-131">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="4aae0-132">Так как MSAL требует использования файла. plist, необходимо настроить Visual Studio с помощью учетной записи разработчика Apple, чтобы включить подготовку.</span><span class="sxs-lookup"><span data-stu-id="4aae0-132">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="4aae0-133">Если вы используете это руководство в имитаторе iPhone, вам нужно добавить **plists.** в поле **Настраиваемые** права в параметрах проекта **графтуториал. iOS** , **Сборка >Подписывание пакетов iOS**.</span><span class="sxs-lookup"><span data-stu-id="4aae0-133">If you are running this tutorial in the iPhone simulator, you need to add **Entitlements.plist** in the **Custom Entitlements** field in **GraphTutorial.iOS** project's settings, **Build->iOS Bundle Signing**.</span></span> <span data-ttu-id="4aae0-134">Для получения дополнительных сведений см [deviceing Device Подготовка для Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).</span><span class="sxs-lookup"><span data-stu-id="4aae0-134">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="4aae0-135">При использовании в проекте Xamarin для iOS Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span><span class="sxs-lookup"><span data-stu-id="4aae0-135">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

1. <span data-ttu-id="4aae0-136">В обозревателе решений разверните проект **графтуториал. iOS** , а затем откройте файл **plist** .</span><span class="sxs-lookup"><span data-stu-id="4aae0-136">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span>

1. <span data-ttu-id="4aae0-137">Укажите право на предоставление **цепочки ключей** и выберите **включить цепочку ключей**.</span><span class="sxs-lookup"><span data-stu-id="4aae0-137">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span>

1. <span data-ttu-id="4aae0-138">В **группы цепочки ключей**добавьте запись в формате `com.companyname.GraphTutorial`.</span><span class="sxs-lookup"><span data-stu-id="4aae0-138">In **Keychain Groups**, add an entry with the format `com.companyname.GraphTutorial`.</span></span>

    ![Снимок экрана: Настройка обслуживания цепочки ключей](./images/enable-keychain-access.png)

1. <span data-ttu-id="4aae0-140">Обновление кода в проекте **графтуториал. iOS** для обработки перенаправления во время проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="4aae0-140">Update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="4aae0-141">Откройте файл **AppDelegate.CS** и добавьте приведенный ниже `using` оператор в начало файла.</span><span class="sxs-lookup"><span data-stu-id="4aae0-141">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="4aae0-142">Добавьте указанную ниже строку `FinishedLaunching` , чтобы она функционировала непосредственно перед `LoadApplication(new App());` строкой.</span><span class="sxs-lookup"><span data-stu-id="4aae0-142">Add the following line to `FinishedLaunching` function just before the `LoadApplication(new App());` line.</span></span>

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. <span data-ttu-id="4aae0-143">Переопределите `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="4aae0-143">Override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="4aae0-144">Добавьте в `AppDelegate` класс следующее:</span><span class="sxs-lookup"><span data-stu-id="4aae0-144">Add the following to the `AppDelegate` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a><span data-ttu-id="4aae0-145">Сохранение маркеров</span><span class="sxs-lookup"><span data-stu-id="4aae0-145">Storing the tokens</span></span>

<span data-ttu-id="4aae0-146">Когда библиотека проверки подлинности (Майкрософт) используется в проекте Xamarin, для кэширования маркеров по умолчанию используется встроенное безопасное хранение.</span><span class="sxs-lookup"><span data-stu-id="4aae0-146">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="4aae0-147">Дополнительные сведения см. в статье [сериализация кэша маркеров](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .</span><span class="sxs-lookup"><span data-stu-id="4aae0-147">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="4aae0-148">Проверка входа</span><span class="sxs-lookup"><span data-stu-id="4aae0-148">Test sign-in</span></span>

<span data-ttu-id="4aae0-149">На этом шаге при запуске приложения и нажатии кнопки **входа** вам будет предложено выполнить вход.</span><span class="sxs-lookup"><span data-stu-id="4aae0-149">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="4aae0-150">При успешном входе в систему отображается маркер доступа, напечатанный в выводе отладчика.</span><span class="sxs-lookup"><span data-stu-id="4aae0-150">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Снимок экрана с окном вывода в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="4aae0-152">Получение сведений о пользователе</span><span class="sxs-lookup"><span data-stu-id="4aae0-152">Get user details</span></span>

1. <span data-ttu-id="4aae0-153">Добавьте новую функцию в класс **app** , чтобы инициализировать `GraphServiceClient`.</span><span class="sxs-lookup"><span data-stu-id="4aae0-153">Add a new function to the **App** class to initialize the `GraphServiceClient`.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. <span data-ttu-id="4aae0-154">Обновите `SignIn` функцию в **app.XAML.CS** , чтобы вызвать эту функцию, `GetUserInfo`а не.</span><span class="sxs-lookup"><span data-stu-id="4aae0-154">Update the `SignIn` function in **App.xaml.cs** to call this function instead of `GetUserInfo`.</span></span> <span data-ttu-id="4aae0-155">Удалите из `SignIn` функции следующее:</span><span class="sxs-lookup"><span data-stu-id="4aae0-155">Remove the following from the `SignIn` function.</span></span>

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. <span data-ttu-id="4aae0-156">Добавьте следующий элемент в конец `SignIn` функции.</span><span class="sxs-lookup"><span data-stu-id="4aae0-156">Add the following to the end of the `SignIn` function.</span></span>

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. <span data-ttu-id="4aae0-157">Обновите `GetUserInfo` функцию, чтобы получить сведения о пользователе из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="4aae0-157">Update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="4aae0-158">Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="4aae0-158">Replace the existing `GetUserInfo` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. <span data-ttu-id="4aae0-159">Сохраните изменения и запустите приложение.</span><span class="sxs-lookup"><span data-stu-id="4aae0-159">Save your changes and run the app.</span></span> <span data-ttu-id="4aae0-160">После входа в пользовательский интерфейс будет выполнено обновление отображаемого имени пользователя и адреса электронной почты.</span><span class="sxs-lookup"><span data-stu-id="4aae0-160">After sign-in the UI is updated with the user's display name and email address.</span></span>
