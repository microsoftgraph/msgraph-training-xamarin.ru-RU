<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5bdad-101">В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="5bdad-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="5bdad-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5bdad-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="5bdad-103">На этом этапе выполняется интеграция [библиотеки проверки подлинности Microsoft для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.</span><span class="sxs-lookup"><span data-stu-id="5bdad-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

<span data-ttu-id="5bdad-104">В **обозревателе решений**разверните проект **графтуториал** и щелкните правой кнопкой мыши папку **Models** .</span><span class="sxs-lookup"><span data-stu-id="5bdad-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="5bdad-105">Выберите **Добавить класс >..**.. Назовите класс `OAuthSettings` и нажмите кнопку **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="5bdad-105">Select **Add > Class...**. Name the class `OAuthSettings` and select **Add**.</span></span> <span data-ttu-id="5bdad-106">Откройте файл **OAuthSettings.CS** и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="5bdad-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

```cs
namespace GraphTutorial.Models
{
    public static class OAuthSettings
    {
        public const string ApplicationId = "YOUR_APP_ID_HERE";
        public const string Scopes = "User.Read Calendars.Read";
    }
}
```

<span data-ttu-id="5bdad-107">Замените `YOUR_APP_ID_HERE` идентификатором приложения из регистрации приложения.</span><span class="sxs-lookup"><span data-stu-id="5bdad-107">Replace `YOUR_APP_ID_HERE` with the application ID from your app registration.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="5bdad-108">Если вы используете систему управления версиями (например, Git), то теперь мы бы не могли исключить `OAuthSettings.cs` файл из системы управления версиями, чтобы избежать случайной утечки идентификатора приложения.</span><span class="sxs-lookup"><span data-stu-id="5bdad-108">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="5bdad-109">Реализация входа</span><span class="sxs-lookup"><span data-stu-id="5bdad-109">Implement sign-in</span></span>

<span data-ttu-id="5bdad-110">Откройте файл **app.XAML.CS** в проекте **графтуториал** и добавьте приведенные ниже `using` операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="5bdad-110">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

```cs
using GraphTutorial.Models;
using Microsoft.Identity.Client;
using Microsoft.Graph;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
```

<span data-ttu-id="5bdad-111">Измените строку объявления класса **приложения** , чтобы разрешить конфликт имен для **приложения**.</span><span class="sxs-lookup"><span data-stu-id="5bdad-111">Change the **App** class declaration line to resolve the name conflict for **Application**.</span></span>

```cs
public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
```

<span data-ttu-id="5bdad-112">Добавьте в `App` класс указанные ниже свойства.</span><span class="sxs-lookup"><span data-stu-id="5bdad-112">Add the following properties to the `App` class.</span></span>

```cs
// UIParent used by Android version of the app
public static object AuthUIParent = null;

// Keychain security group used by iOS version of the app
public static string iOSKeychainSecurityGroup = null;

// Microsoft Authentication client for native/mobile apps
public static IPublicClientApplication PCA;

// Microsoft Graph client
public static GraphServiceClient GraphClient;

// Microsoft Graph permissions used by app
private readonly string[] Scopes = OAuthSettings.Scopes.Split(' ');
```

<span data-ttu-id="5bdad-113">Затем создайте новый `PublicClientApplication` элемент в конструкторе `App` класса.</span><span class="sxs-lookup"><span data-stu-id="5bdad-113">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

```cs
public App()
{
    InitializeComponent();

    var builder = PublicClientApplicationBuilder
        .Create(OAuthSettings.ApplicationId);

    if (!string.IsNullOrEmpty(iOSKeychainSecurityGroup))
    {
        builder = builder.WithIosKeychainSecurityGroup(iOSKeychainSecurityGroup);
    }

    PCA = builder.Build();

    MainPage = new MainPage();
}
```

<span data-ttu-id="5bdad-114">Теперь обновите `SignIn` функцию, чтобы она `PublicClientApplication` использовалась для получения маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="5bdad-114">Now update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="5bdad-115">Добавьте приведенный ниже код перед `await GetUserInfo();` строкой.</span><span class="sxs-lookup"><span data-stu-id="5bdad-115">Add the following code above the `await GetUserInfo();` line.</span></span>

```cs
// First, attempt silent sign in
// If the user's information is already in the app's cache,
// they won't have to sign in again.
try
{
    var accounts = await PCA.GetAccountsAsync();

    var silentAuthResult = await PCA
        .AcquireTokenSilent(Scopes, accounts.FirstOrDefault())
        .ExecuteAsync();

    Debug.WriteLine("User already signed in.");
    Debug.WriteLine($"Successful silent authentication for: {silentAuthResult.Account.Username}");
    Debug.WriteLine($"Access token: {silentAuthResult.AccessToken}");
}
catch (MsalUiRequiredException msalEx)
{
    // This exception is thrown when an interactive sign-in is required.
    Debug.WriteLine("Silent token request failed, user needs to sign-in: " + msalEx.Message);
    // Prompt the user to sign-in
    var interactiveRequest = PCA.AcquireTokenInteractive(Scopes);

    if (AuthUIParent != null)
    {
        interactiveRequest = interactiveRequest
            .WithParentActivityOrWindow(AuthUIParent);
    }

    var interactiveAuthResult = await interactiveRequest.ExecuteAsync();
    Debug.WriteLine($"Successful interactive authentication for: {interactiveAuthResult.Account.Username}");
    Debug.WriteLine($"Access token: {interactiveAuthResult.AccessToken}");
}
catch (Exception ex)
{
    Debug.WriteLine("Authentication failed. See exception messsage for more details: " + ex.Message);
}
```

<span data-ttu-id="5bdad-116">Этот код сначала пытается получить маркер доступа в автоматическом режиме.</span><span class="sxs-lookup"><span data-stu-id="5bdad-116">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="5bdad-117">Если сведения о пользователе уже находятся в кэше приложения (например, если пользователь закрыл приложение раньше, без выхода), это будет успешным, и нет причин запрашивать пользователя.</span><span class="sxs-lookup"><span data-stu-id="5bdad-117">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="5bdad-118">Если в кэше нет сведений о пользователе, `AcquireTokenSilent().ExecuteAsync()` функция создает исключение. `MsalUiRequiredException`</span><span class="sxs-lookup"><span data-stu-id="5bdad-118">If there is not a user's information in the cache, the `AcquireTokenSilent().ExecuteAsync()` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="5bdad-119">В этом случае код вызывает интерактивную функцию для получения маркера `AcquireTokenInteractive`.</span><span class="sxs-lookup"><span data-stu-id="5bdad-119">In this case, the code calls the interactive function to get a token, `AcquireTokenInteractive`.</span></span>

<span data-ttu-id="5bdad-120">Теперь обновите `SignOut` функцию, чтобы удалить сведения о пользователе из кэша.</span><span class="sxs-lookup"><span data-stu-id="5bdad-120">Now update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="5bdad-121">Добавьте следующий код в начало `SignOut` функции.</span><span class="sxs-lookup"><span data-stu-id="5bdad-121">Add the following code to the beginning of the `SignOut` function.</span></span>

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

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="5bdad-122">Обновление проекта Android для включения входа</span><span class="sxs-lookup"><span data-stu-id="5bdad-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="5bdad-123">При использовании в проекте Xamarin Android Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span><span class="sxs-lookup"><span data-stu-id="5bdad-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

<span data-ttu-id="5bdad-124">Сначала необходимо обновить манифест Android, чтобы зарегистрировать URI перенаправления.</span><span class="sxs-lookup"><span data-stu-id="5bdad-124">First, you need to update the Android manifest to register the redirect URI.</span></span> <span data-ttu-id="5bdad-125">В проекте **графтуториал. Android** разверните папку **свойства** , а затем откройте **файл AndroidManifest. XML**.</span><span class="sxs-lookup"><span data-stu-id="5bdad-125">In **GraphTutorial.Android** project, expand the **Properties** folder, then open **AndroidManifest.xml**.</span></span> <span data-ttu-id="5bdad-126">Если вы используете Visual Studio для Mac, переключитесь на представление **источника** с помощью вкладок в нижней части файла.</span><span class="sxs-lookup"><span data-stu-id="5bdad-126">If you're using Visual Studio for Mac, switch to the **Source** view using the tabs at the bottom of the file.</span></span> <span data-ttu-id="5bdad-127">Замените все содержимое приведенным ниже содержимым.</span><span class="sxs-lookup"><span data-stu-id="5bdad-127">Replace the entire contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:versionCode="1" android:versionName="1.0" package="com.companyname.GraphTutorial">
    <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="28" />
    <application android:label="GraphTutorial.Android">
        <activity android:name="microsoft.identity.client.BrowserTabActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="msalYOUR_APP_ID_HERE" android:host="auth" />
            </intent-filter>
        </activity>
    </application>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

<span data-ttu-id="5bdad-128">Замените `YOUR_APP_ID_HERE` идентификатором приложения из регистрации приложения.</span><span class="sxs-lookup"><span data-stu-id="5bdad-128">Replace `YOUR_APP_ID_HERE` with your application ID from your app registration.</span></span>

<span data-ttu-id="5bdad-129">Затем откройте **MainActivity.CS** и добавьте приведенные ниже `using` операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="5bdad-129">Next, open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
using Android.Content;
```

<span data-ttu-id="5bdad-130">Затем переопределите `OnActivityResult` функцию, чтобы передать управление в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="5bdad-130">Then, override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="5bdad-131">Добавьте в `MainActivity` класс следующее:</span><span class="sxs-lookup"><span data-stu-id="5bdad-131">Add the following to the `MainActivity` class.</span></span>

```cs
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationContinuationHelper
        .SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

<span data-ttu-id="5bdad-132">Наконец, в `OnCreate` функции добавьте следующую строку после `LoadApplication(new App());` строки.</span><span class="sxs-lookup"><span data-stu-id="5bdad-132">Finally, in the `OnCreate` function, add the following line after the `LoadApplication(new App());` line.</span></span>

```cs
App.AuthUIParent = this;
```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="5bdad-133">Обновление проекта iOS для включения входа</span><span class="sxs-lookup"><span data-stu-id="5bdad-133">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="5bdad-134">Так как MSAL требует использования файла. plist, необходимо настроить Visual Studio с помощью учетной записи разработчика Apple, чтобы включить подготовку.</span><span class="sxs-lookup"><span data-stu-id="5bdad-134">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="5bdad-135">Если вы используете это руководство в имитаторе iPhone, вам нужно добавить **plists.** в поле **Настраиваемые** права в параметрах проекта **графтуториал. iOS** , **Сборка >Подписывание пакетов iOS**.</span><span class="sxs-lookup"><span data-stu-id="5bdad-135">If you are running this tutorial in the iPhone simulator, you need to add **Entitlements.plist** in the **Custom Entitlements** field in **GraphTutorial.iOS** project's settings, **Build->iOS Bundle Signing**.</span></span> <span data-ttu-id="5bdad-136">Для получения дополнительных сведений см [deviceing Device Подготовка для Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).</span><span class="sxs-lookup"><span data-stu-id="5bdad-136">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="5bdad-137">При использовании в проекте Xamarin для iOS Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span><span class="sxs-lookup"><span data-stu-id="5bdad-137">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

<span data-ttu-id="5bdad-138">Для начала необходимо включить доступ к цепочке ключей.</span><span class="sxs-lookup"><span data-stu-id="5bdad-138">First, you need to enable Keychain access.</span></span> <span data-ttu-id="5bdad-139">В обозревателе решений разверните проект **графтуториал. iOS** , а затем откройте файл **plist** .</span><span class="sxs-lookup"><span data-stu-id="5bdad-139">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span> <span data-ttu-id="5bdad-140">Укажите право на предоставление **цепочки ключей** и выберите **включить цепочку ключей**.</span><span class="sxs-lookup"><span data-stu-id="5bdad-140">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span> <span data-ttu-id="5bdad-141">В **группы цепочки ключей**добавьте запись в формате `com.YOUR_DOMAIN.GraphTutorial`.</span><span class="sxs-lookup"><span data-stu-id="5bdad-141">In **Keychain Groups**, add an entry with the format `com.YOUR_DOMAIN.GraphTutorial`.</span></span>

![Снимок экрана: Настройка обслуживания цепочки ключей](./images/enable-keychain-access.png)

<span data-ttu-id="5bdad-143">Далее необходимо зарегистрировать URI перенаправления MSAL по умолчанию в качестве типа URL-адреса, обрабатываемого вашим приложением.</span><span class="sxs-lookup"><span data-stu-id="5bdad-143">Next, you need to register the default MSAL redirect URI as a URL type that your app handles.</span></span> <span data-ttu-id="5bdad-144">Откройте файл **info. plist** и внесите следующие изменения.</span><span class="sxs-lookup"><span data-stu-id="5bdad-144">Open the **Info.plist** file and make the following changes.</span></span>

- <span data-ttu-id="5bdad-145">На вкладке **приложение** убедитесь, что значение **идентификатора пакета** соответствует значению, заданному для **групп ключей** в целях **обслуживания. plist**.</span><span class="sxs-lookup"><span data-stu-id="5bdad-145">On the **Application** tab, check that the value of **Bundle identifier** matches the value you set for **Keychain Groups** in **Entitlements.plist**.</span></span> <span data-ttu-id="5bdad-146">Если это не так, обновите его сейчас.</span><span class="sxs-lookup"><span data-stu-id="5bdad-146">If it doesn't, update it now.</span></span>
- <span data-ttu-id="5bdad-147">На вкладке **Дополнительно** перейдите в раздел **типы URL-адресов** .</span><span class="sxs-lookup"><span data-stu-id="5bdad-147">On the **Advanced** tab, locate the **URL Types** section.</span></span> <span data-ttu-id="5bdad-148">Добавьте URL-адрес типа со следующими значениями:</span><span class="sxs-lookup"><span data-stu-id="5bdad-148">Add a URL type here with the following values:</span></span>
  - <span data-ttu-id="5bdad-149">**Identifier**: значение **идентификатора набора**</span><span class="sxs-lookup"><span data-stu-id="5bdad-149">**Identifier**: set to the value of your **Bundle identifier**</span></span>
  - <span data-ttu-id="5bdad-150">**Схемы URL-адресов**: `msal{YOUR-APP-ID}`задано значение.</span><span class="sxs-lookup"><span data-stu-id="5bdad-150">**URL Schemes**: set to `msal{YOUR-APP-ID}`.</span></span> <span data-ttu-id="5bdad-151">Например, если идентификатор приложения — `67ad5eba-0cfc-414d-8f9f-0a6d973a907c`, задайте значение. `msal67ad5eba-0cfc-414d-8f9f-0a6d973a907c`</span><span class="sxs-lookup"><span data-stu-id="5bdad-151">For example, if your app ID is `67ad5eba-0cfc-414d-8f9f-0a6d973a907c`, you would set this to `msal67ad5eba-0cfc-414d-8f9f-0a6d973a907c`.</span></span>
  - <span data-ttu-id="5bdad-152">**Роль**:`Editor`</span><span class="sxs-lookup"><span data-stu-id="5bdad-152">**Role**: `Editor`</span></span>
  - <span data-ttu-id="5bdad-153">**Значок**: оставьте пустым</span><span class="sxs-lookup"><span data-stu-id="5bdad-153">**Icon**: Leave empty</span></span>

![Снимок экрана с разделом "типы URL-адресов" в разделе info. plist](./images/add-url-type.png)

<span data-ttu-id="5bdad-155">Наконец, обновите код в проекте **графтуториал. iOS** для обработки перенаправления во время проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="5bdad-155">Finally, update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="5bdad-156">Откройте файл **AppDelegate.CS** и добавьте приведенный ниже `using` оператор в начало файла.</span><span class="sxs-lookup"><span data-stu-id="5bdad-156">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
```

<span data-ttu-id="5bdad-157">Добавьте указанную ниже строку `FinishedLaunching` , чтобы она функционировала непосредственно перед `LoadApplication(new App());` строкой.</span><span class="sxs-lookup"><span data-stu-id="5bdad-157">Add the following line to `FinishedLaunching` function just before the `LoadApplication(new App());` line.</span></span>

```cs
// Specify the Keychain access group
App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
```

<span data-ttu-id="5bdad-158">Наконец, переопределите `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL.</span><span class="sxs-lookup"><span data-stu-id="5bdad-158">Finally, override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="5bdad-159">Добавьте в `AppDelegate` класс следующее:</span><span class="sxs-lookup"><span data-stu-id="5bdad-159">Add the following to the `AppDelegate` class.</span></span>

```cs
// Handling redirect URL
// See: https://github.com/azuread/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

## <a name="storing-the-tokens"></a><span data-ttu-id="5bdad-160">Сохранение маркеров</span><span class="sxs-lookup"><span data-stu-id="5bdad-160">Storing the tokens</span></span>

<span data-ttu-id="5bdad-161">Когда библиотека проверки подлинности (Майкрософт) используется в проекте Xamarin, для кэширования маркеров по умолчанию используется встроенное безопасное хранение.</span><span class="sxs-lookup"><span data-stu-id="5bdad-161">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="5bdad-162">Дополнительные сведения см. в статье [сериализация кэша маркеров](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .</span><span class="sxs-lookup"><span data-stu-id="5bdad-162">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="5bdad-163">Проверка входа</span><span class="sxs-lookup"><span data-stu-id="5bdad-163">Test sign-in</span></span>

<span data-ttu-id="5bdad-164">На этом шаге при запуске приложения и нажатии кнопки **входа** вам будет предложено выполнить вход.</span><span class="sxs-lookup"><span data-stu-id="5bdad-164">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="5bdad-165">При успешном входе в систему отображается маркер доступа, напечатанный в выводе отладчика.</span><span class="sxs-lookup"><span data-stu-id="5bdad-165">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Снимок экрана с окном вывода в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="5bdad-167">Получение сведений о пользователе</span><span class="sxs-lookup"><span data-stu-id="5bdad-167">Get user details</span></span>

<span data-ttu-id="5bdad-168">Добавьте новую функцию в класс **app** , чтобы инициализировать `GraphServiceClient`.</span><span class="sxs-lookup"><span data-stu-id="5bdad-168">Add a new function to the **App** class to initialize the `GraphServiceClient`.</span></span>

```cs
private async Task InitializeGraphClientAsync()
{
    var currentAccounts = await PCA.GetAccountsAsync();
    try
    {
        if (currentAccounts.Count() > 0)
        {
            // Initialize Graph client
            GraphClient = new GraphServiceClient(new DelegateAuthenticationProvider(
                async (requestMessage) =>
                {
                    var result = await PCA.AcquireTokenSilent(Scopes, currentAccounts.FirstOrDefault())
                        .ExecuteAsync();

                    requestMessage.Headers.Authorization =
                        new AuthenticationHeaderValue("Bearer", result.AccessToken);
                }));

            await GetUserInfo();

            IsSignedIn = true;
        }
        else
        {
            IsSignedIn = false;
        }
    }
    catch(Exception ex)
    {
        Debug.WriteLine(
            $"Failed to initialized graph client. Accounts in the msal cache: {currentAccounts.Count()}. See exception message for details: {ex.Message}");
    }
}
```

<span data-ttu-id="5bdad-169">Теперь обновите `SignIn` функцию в **app.XAML.CS** , чтобы вызвать эту функцию, `GetUserInfo`а не.</span><span class="sxs-lookup"><span data-stu-id="5bdad-169">Now update the `SignIn` function in **App.xaml.cs** to call this function instead of `GetUserInfo`.</span></span> <span data-ttu-id="5bdad-170">Удалите из `SignIn` функции следующее:</span><span class="sxs-lookup"><span data-stu-id="5bdad-170">Remove the following from the `SignIn` function.</span></span>

```cs
await GetUserInfo();

IsSignedIn = true;
```

<span data-ttu-id="5bdad-171">Добавьте следующий элемент в конец `SignIn` функции.</span><span class="sxs-lookup"><span data-stu-id="5bdad-171">Add the following to the end of the `SignIn` function.</span></span>

```cs
await InitializeGraphClientAsync();
```

<span data-ttu-id="5bdad-172">Теперь обновите `GetUserInfo` функцию, чтобы получить сведения о пользователе из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5bdad-172">Now update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="5bdad-173">Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="5bdad-173">Replace the existing `GetUserInfo` function with the following.</span></span>

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

<span data-ttu-id="5bdad-174">Если вы сохраните изменения и запустите приложение сейчас, после входа в пользовательский интерфейс отобразится отображаемое имя пользователя и адрес электронной почты.</span><span class="sxs-lookup"><span data-stu-id="5bdad-174">If you save your changes and run the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
