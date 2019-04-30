<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e6c35-101">В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="e6c35-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="e6c35-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="e6c35-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="e6c35-103">На этом этапе выполняется интеграция [библиотеки проверки ПодлиннОсти Microsoft для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.</span><span class="sxs-lookup"><span data-stu-id="e6c35-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

<span data-ttu-id="e6c35-104">В **обозревателе решений**разверните проект **графтуториал** и щелкните правой кнопкой мыши папку **Models** .</span><span class="sxs-lookup"><span data-stu-id="e6c35-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="e6c35-105">Нажмите кнопку **Добавить класс _гт_...**. НаЗовите класс `OAuthSettings` и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="e6c35-105">Choose **Add > Class...**. Name the class `OAuthSettings` and choose **Add**.</span></span> <span data-ttu-id="e6c35-106">Откройте файл **OAuthSettings.CS** и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="e6c35-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

```cs
namespace GraphTutorial.Models
{
    public static class OAuthSettings
    {
        public const string ApplicationId = "YOUR_APP_ID_HERE";
        public const string RedirectUri = "YOUR_REDIRECT_URI_HERE";
        public const string Scopes = "User.Read Calendars.Read";
    }
}
```

> [!IMPORTANT]
> <span data-ttu-id="e6c35-107">Если вы используете систему управления версиями (например, Git), то теперь мы бы не могли исключить `OAuthSettings.cs` файл из системы управления версиями, чтобы избежать случайной утечки идентификатора приложения.</span><span class="sxs-lookup"><span data-stu-id="e6c35-107">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="e6c35-108">Реализация входа</span><span class="sxs-lookup"><span data-stu-id="e6c35-108">Implement sign-in</span></span>

<span data-ttu-id="e6c35-109">Откройте файл **app.XAML.CS** в проекте **графтуториал** и добавьте приведенные ниже `using` операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="e6c35-109">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

```cs
using GraphTutorial.Models;
using Microsoft.Identity.Client;
using Microsoft.Graph;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
```

<span data-ttu-id="e6c35-110">Добавьте в `App` класс указанные ниже свойства.</span><span class="sxs-lookup"><span data-stu-id="e6c35-110">Add the following properties to the `App` class.</span></span>

```cs
// UIParent used by Android version of the app
public static UIParent AuthUIParent = null;

// Microsoft Authentication client for native/mobile apps
public static PublicClientApplication PCA;

// Microsoft Graph client
public static GraphServiceClient GraphClient;
```

<span data-ttu-id="e6c35-111">Затем создайте новый `PublicClientApplication` элемент в конструкторе `App` класса.</span><span class="sxs-lookup"><span data-stu-id="e6c35-111">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

```cs
public App()
{
    InitializeComponent();

    PCA = new PublicClientApplication(OAuthSettings.ApplicationId);

    MainPage = new MainPage();
}
```

<span data-ttu-id="e6c35-112">Теперь обновите `SignIn` функцию, чтобы она `PublicClientApplication` использовалась для получения маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="e6c35-112">Now update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="e6c35-113">Добавьте приведенный ниже код перед `await GetUserInfo();` строкой.</span><span class="sxs-lookup"><span data-stu-id="e6c35-113">Add the following code above the `await GetUserInfo();` line.</span></span>

```cs
var scopes = OAuthSettings.Scopes.Split(' ');

// First, attempt silent sign in
// If the user's information is already in the app's cache,
// they won't have to sign in again.
try
{
    var accounts = await PCA.GetAccountsAsync();
    var silentAuthResult = await PCA.AcquireTokenSilentAsync(
        scopes, accounts.FirstOrDefault());

    Debug.WriteLine("User already signed in.");
    Debug.WriteLine($"Access token: {silentAuthResult.AccessToken}");
}
catch (MsalUiRequiredException)
{
    // This exception is thrown when an interactive sign-in is required.
    // Prompt the user to sign-in
    var authResult = await PCA.AcquireTokenAsync(scopes, AuthUIParent);
    Debug.WriteLine($"Access Token: {authResult.AccessToken}");
}
```

<span data-ttu-id="e6c35-114">Этот код сначала пытается получить маркер доступа в автоматическом режиме.</span><span class="sxs-lookup"><span data-stu-id="e6c35-114">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="e6c35-115">Если сведения о пользователе уже находятся в кэше приложения (например, если пользователь закрыл приложение раньше, без выхода), это будет успешным, и нет причин запрашивать пользователя.</span><span class="sxs-lookup"><span data-stu-id="e6c35-115">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="e6c35-116">Если в кэше нет сведений о пользователе, `AcquireTokenSilentAsync` функция создает исключение. `MsalUiRequiredException`</span><span class="sxs-lookup"><span data-stu-id="e6c35-116">If there is not a user's information in the cache, the `AcquireTokenSilentAsync` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="e6c35-117">В этом случае код вызывает интерактивную функцию для получения маркера `AcquireTokenAsync`.</span><span class="sxs-lookup"><span data-stu-id="e6c35-117">In this case, the code calls the interactive function to get a token, `AcquireTokenAsync`.</span></span>

<span data-ttu-id="e6c35-118">Теперь обновите `SignOut` функцию, чтобы удалить сведения о пользователе из кэша.</span><span class="sxs-lookup"><span data-stu-id="e6c35-118">Now update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="e6c35-119">Добавьте следующий код в начало `SignOut` функции.</span><span class="sxs-lookup"><span data-stu-id="e6c35-119">Add the following code to the beginning of the `SignOut` function.</span></span>

```cs
// Get all cached accounts for the app
// (Should only be one)
var accounts = await PCA.GetAccountsAsync();
while (accounts.Any())
{
    // Remove the account info from the cache
    await PCA.RemoveAsync(accounts.First());
    accounts = await PCA.GetAccountsAsync();
}
```

<span data-ttu-id="e6c35-120">Наконец, обновите `MainPage` класс, чтобы войти в систему при загрузке.</span><span class="sxs-lookup"><span data-stu-id="e6c35-120">Finally, update the `MainPage` class to sign-in when it loads.</span></span> <span data-ttu-id="e6c35-121">Добавьте указанную ниже функцию в `MainPage` класс в **MainPage.XAML.CS**.</span><span class="sxs-lookup"><span data-stu-id="e6c35-121">Add the following function to the `MainPage` class in **MainPage.xaml.cs**.</span></span>

```cs
protected override async void OnAppearing()
{
    base.OnAppearing();
    await (Application.Current as App).SignIn();
}
```

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="e6c35-122">Обновление проекта Android для включения входа</span><span class="sxs-lookup"><span data-stu-id="e6c35-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="e6c35-123">При использовании в проекте Xamarin Android Библиотека проверки поДлинности (Майкрософт) имеет несколько [требований, характерных для Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span><span class="sxs-lookup"><span data-stu-id="e6c35-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

<span data-ttu-id="e6c35-124">Сначала необходимо добавить URI перенаправления из регистрации приложения в манифест Android.</span><span class="sxs-lookup"><span data-stu-id="e6c35-124">First, you need to add the redirect URI from your app registration to the Android manifest.</span></span> <span data-ttu-id="e6c35-125">Для этого добавьте новое действие в проект **графтуториал. Android** .</span><span class="sxs-lookup"><span data-stu-id="e6c35-125">To do this, add a new activity to the **GraphTutorial.Android** project.</span></span> <span data-ttu-id="e6c35-126">Щелкните правой кнопкой мыши **графтуториал. Android** и выберите команду **Добавить**, а затем — **новый элемент...**. Выберите **действие**, введите имя действия `MsalActivity`и нажмите кнопку **добавить**.</span><span class="sxs-lookup"><span data-stu-id="e6c35-126">Right-click the **GraphTutorial.Android** and choose **Add**, then **New Item...**. Choose **Activity**, name the activity `MsalActivity`, and choose **Add**.</span></span>

<span data-ttu-id="e6c35-127">Откройте файл **MsalActivity.CS** и удалите `[Activity(Label = "MsalActivity")]` строку, а затем добавьте следующие атрибуты над объявлением класса.</span><span class="sxs-lookup"><span data-stu-id="e6c35-127">Open the **MsalActivity.cs** file and delete the `[Activity(Label = "MsalActivity")]` line, then add the following attributes above the class declaration.</span></span>

```cs
// This class only exists to create the necessary activity in the Android
// manifest. Doing it this way allows the value of the RedirectUri constant
// to be inserted at build.
[Activity(Name = "microsoft.identity.client.BrowserTabActivity")]
[IntentFilter(new[] { Intent.ActionView },
    Categories = new[] { Intent.CategoryDefault, Intent.CategoryBrowsable },
    DataScheme = Models.OAuthSettings.RedirectUri, DataHost = "auth")]
```

<span data-ttu-id="e6c35-128">Затем откройте **MainActivity.CS** и добавьте приведенные ниже `using` операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="e6c35-128">Next, open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
using Android.Content;
```

<span data-ttu-id="e6c35-129">Затем переопределите `OnActivityResult` функцию, чтобы передать управление в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="e6c35-129">Then, override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="e6c35-130">Добавьте в `MainActivity` класс следующее:</span><span class="sxs-lookup"><span data-stu-id="e6c35-130">Add the following to the `MainActivity` class.</span></span>

```cs
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationContinuationHelper
        .SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="e6c35-131">Обновление проекта iOS для включения входа</span><span class="sxs-lookup"><span data-stu-id="e6c35-131">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="e6c35-132">Так как MSAL требует использования файла. plist, необходимо настроить Visual Studio с помощью учетной записи разработчика Apple, чтобы включить подготовку.</span><span class="sxs-lookup"><span data-stu-id="e6c35-132">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="e6c35-133">Для получения дополнительных сведений см [deviceIng Device Подготовка для Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).</span><span class="sxs-lookup"><span data-stu-id="e6c35-133">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="e6c35-134">При использовании в проекте Xamarin для iOS Библиотека проверки поДлинности (Майкрософт) имеет несколько [требований, характерных для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span><span class="sxs-lookup"><span data-stu-id="e6c35-134">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

<span data-ttu-id="e6c35-135">Для начала необходимо включить доступ к цепочке ключей.</span><span class="sxs-lookup"><span data-stu-id="e6c35-135">First, you need to enable Keychain access.</span></span> <span data-ttu-id="e6c35-136">В обозревателе решений разверните проект **графтуториал. iOS** , а затем откройте файл **plist** .</span><span class="sxs-lookup"><span data-stu-id="e6c35-136">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span> <span data-ttu-id="e6c35-137">Укажите право на предоставление **цепочки ключей** и выберите **включить цепочку ключей**.</span><span class="sxs-lookup"><span data-stu-id="e6c35-137">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span> <span data-ttu-id="e6c35-138">В **группы цепочки ключей**добавьте запись в формате `com.YOUR_DOMAIN.GraphTutorial`.</span><span class="sxs-lookup"><span data-stu-id="e6c35-138">In **Keychain Groups**, add an entry with the format `com.YOUR_DOMAIN.GraphTutorial`.</span></span>

![Снимок экрана: Настройка обслуживания цепочки ключей](./images/enable-keychain-access.png)

<span data-ttu-id="e6c35-140">Далее необходимо зарегистрировать URI перенаправления, настроенный в шаге регистрации приложения, в виде URL-адреса, который обрабатывает ваше приложение.</span><span class="sxs-lookup"><span data-stu-id="e6c35-140">Next, you need to register the redirect URI you configured in the app registration step as a URL type that your app handles.</span></span> <span data-ttu-id="e6c35-141">Откройте файл **info. plist** и внесите следующие изменения.</span><span class="sxs-lookup"><span data-stu-id="e6c35-141">Open the **Info.plist** file and make the following changes.</span></span>

- <span data-ttu-id="e6c35-142">На вкладке **приложение** убедитесь, что значение **идентификатора пакета** соответствует значению, заданному для **групп ключей** в целях **обслуживания. plist**.</span><span class="sxs-lookup"><span data-stu-id="e6c35-142">On the **Application** tab, check that the value of **Bundle identifier** matches the value you set for **Keychain Groups** in **Entitlements.plist**.</span></span> <span data-ttu-id="e6c35-143">Если это не так, обновите его сейчас.</span><span class="sxs-lookup"><span data-stu-id="e6c35-143">If it doesn't, update it now.</span></span>
- <span data-ttu-id="e6c35-144">На вкладке **Дополнительно** перейдите в раздел **типы URL-адресов** .</span><span class="sxs-lookup"><span data-stu-id="e6c35-144">On the **Advanced** tab, locate the **URL Types** section.</span></span> <span data-ttu-id="e6c35-145">Добавьте URL-адрес типа со следующими значениями:</span><span class="sxs-lookup"><span data-stu-id="e6c35-145">Add a URL type here with the following values:</span></span>
    - <span data-ttu-id="e6c35-146">**Identifier**: значение **идентификатора набора**</span><span class="sxs-lookup"><span data-stu-id="e6c35-146">**Identifier**: set to the value of your **Bundle identifier**</span></span>
    - <span data-ttu-id="e6c35-147">**Схемы URL-адресов**: укажите URI перенаправления из регистрации приложения, начинающиеся с`msal`</span><span class="sxs-lookup"><span data-stu-id="e6c35-147">**URL Schemes**: set to the redirect URI from your app registration that begins with `msal`</span></span>
    - <span data-ttu-id="e6c35-148">**Роль**:`Editor`</span><span class="sxs-lookup"><span data-stu-id="e6c35-148">**Role**: `Editor`</span></span>
    - <span data-ttu-id="e6c35-149">**Значок**: оставьте пустым</span><span class="sxs-lookup"><span data-stu-id="e6c35-149">**Icon**: Leave empty</span></span>

![Снимок экрана с разделом "типы URL-адресов" в разделе info. plist](./images/add-url-type.png)

<span data-ttu-id="e6c35-151">Наконец, обновите код в проекте **графтуториал. iOS** для обработки перенаправления во время проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="e6c35-151">Finally, update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="e6c35-152">Откройте файл **AppDelegate.CS** и добавьте приведенный ниже `using` оператор в начало файла.</span><span class="sxs-lookup"><span data-stu-id="e6c35-152">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
```

<span data-ttu-id="e6c35-153">Добавьте указанную ниже строку `FinishedLaunching` , чтобы работать непосредственно `return` перед оператором.</span><span class="sxs-lookup"><span data-stu-id="e6c35-153">Add the following line to `FinishedLaunching` function just before the `return` statement.</span></span>

```cs
// Specify the Keychain access group
App.PCA.iOSKeychainSecurityGroup = "com.graphdevx.GraphTutorial";
```

<span data-ttu-id="e6c35-154">Наконец, переопределите `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="e6c35-154">Finally, override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="e6c35-155">Добавьте в `AppDelegate` класс следующее:</span><span class="sxs-lookup"><span data-stu-id="e6c35-155">Add the following to the `AppDelegate` class.</span></span>

```cs
// Handling redirect URL
// See: https://github.com/azuread/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

## <a name="storing-the-tokens"></a><span data-ttu-id="e6c35-156">Сохранение маркеров</span><span class="sxs-lookup"><span data-stu-id="e6c35-156">Storing the tokens</span></span>

<span data-ttu-id="e6c35-157">Когда библиотека проверки поДлинности (Майкрософт) используется в проекте Xamarin, для кэширования маркеров по умолчанию используется встроенное безопасное хранение.</span><span class="sxs-lookup"><span data-stu-id="e6c35-157">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="e6c35-158">Дополнительные сведения см. в статье [сериализация кэша маркеров](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .</span><span class="sxs-lookup"><span data-stu-id="e6c35-158">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="e6c35-159">Проверка входа</span><span class="sxs-lookup"><span data-stu-id="e6c35-159">Test sign-in</span></span>

<span data-ttu-id="e6c35-160">На этом шаге при запуске приложения и нажатии кнопки **входа** вам будет предложено выполнить вход.</span><span class="sxs-lookup"><span data-stu-id="e6c35-160">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="e6c35-161">При успешном входе в систему отображается маркер доступа, напечатанный в выводе отладчика.</span><span class="sxs-lookup"><span data-stu-id="e6c35-161">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Снимок экрана с окном вывода в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="e6c35-163">Получение сведений о пользователе</span><span class="sxs-lookup"><span data-stu-id="e6c35-163">Get user details</span></span>

<span data-ttu-id="e6c35-164">Теперь обновите `SignIn` функцию в **app.XAML.CS** , чтобы инициализировать `GraphServiceClient`.</span><span class="sxs-lookup"><span data-stu-id="e6c35-164">Now update the `SignIn` function in **App.xaml.cs** to initialize the `GraphServiceClient`.</span></span> <span data-ttu-id="e6c35-165">Добавьте следующий код перед `await GetUserInfo();` строкой.</span><span class="sxs-lookup"><span data-stu-id="e6c35-165">Add the following code before the `await GetUserInfo();` line.</span></span>

```cs
// Initialize Graph client
GraphClient = new GraphServiceClient(new DelegateAuthenticationProvider(
    async (requestMessage) =>
    {
        var accounts = await PCA.GetAccountsAsync();

        var result = await PCA.AcquireTokenSilentAsync(scopes, accounts.FirstOrDefault());

        requestMessage.Headers.Authorization =
            new AuthenticationHeaderValue("Bearer", result.AccessToken);
    }));
```

<span data-ttu-id="e6c35-166">Теперь обновите `GetUserInfo` функцию, чтобы получить сведения о пользователе из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="e6c35-166">Now update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="e6c35-167">Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="e6c35-167">Replace the existing `GetUserInfo` function with the following.</span></span>

```cs
private async Task GetUserInfo()
{
    // Get the logged on user's profile (/me)
    var user = await GraphClient.Me.Request().GetAsync();

    UserPhoto = ImageSource.FromStream(() => GetUserPhoto());
    UserName = user.DisplayName;
    UserEmail = string.IsNullOrEmpty(user.Mail) ? user.UserPrincipalName : user.Mail;
}
```

<span data-ttu-id="e6c35-168">Если вы сохраните изменения и запустите приложение сейчас, после входа в пользовательский интерфейс отобразится отображаемое имя пользователя и адрес электронной почты.</span><span class="sxs-lookup"><span data-stu-id="e6c35-168">If you save your changes and run the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>

