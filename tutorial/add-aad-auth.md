<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph. На этом этапе выполняется интеграция [библиотеки проверки подлинности Microsoft для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.

В **обозревателе решений**разверните проект **графтуториал** и щелкните правой кнопкой мыши папку **Models** . Нажмите кнопку **Добавить класс _гт_...**. Назовите класс `OAuthSettings` и нажмите кнопку **Добавить**. Откройте файл **OAuthSettings.CS** и замените его содержимое на приведенный ниже код.

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

Замените `YOUR_APP_ID_HERE` идентификатором приложения из регистрации приложения.

> [!IMPORTANT]
> Если вы используете систему управления версиями (например, Git), то теперь мы бы не могли исключить `OAuthSettings.cs` файл из системы управления версиями, чтобы избежать случайной утечки идентификатора приложения.

## <a name="implement-sign-in"></a>Реализация входа

Откройте файл **app.XAML.CS** в проекте **графтуториал** и добавьте приведенные ниже `using` операторы в начало файла.

```cs
using GraphTutorial.Models;
using Microsoft.Identity.Client;
using Microsoft.Graph;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
```

Добавьте в `App` класс указанные ниже свойства.

```cs
// UIParent used by Android version of the app
public static object AuthUIParent = null;

// Keychain security group used by iOS version of the app
public static string iOSKeychainSecurityGroup = null;

// Microsoft Authentication client for native/mobile apps
public static IPublicClientApplication PCA;

// Microsoft Graph client
public static GraphServiceClient GraphClient;
```

Затем создайте новый `PublicClientApplication` элемент в конструкторе `App` класса.

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

Теперь обновите `SignIn` функцию, чтобы она `PublicClientApplication` использовалась для получения маркера доступа. Добавьте приведенный ниже код перед `await GetUserInfo();` строкой.

```cs
var scopes = OAuthSettings.Scopes.Split(' ');

// First, attempt silent sign in
// If the user's information is already in the app's cache,
// they won't have to sign in again.
string accessToken = string.Empty;
try
{
    var accounts = await PCA.GetAccountsAsync();
    if (accounts.Count() > 0)
    {
        var silentAuthResult = await PCA
            .AcquireTokenSilent(scopes, accounts.FirstOrDefault())
            .ExecuteAsync();

        Debug.WriteLine("User already signed in.");
        Debug.WriteLine($"Access token: {silentAuthResult.AccessToken}");
        accessToken = silentAuthResult.AccessToken;
    }
}
catch (MsalUiRequiredException)
{
    // This exception is thrown when an interactive sign-in is required.
    Debug.WriteLine("Silent token request failed, user needs to sign-in");
}

if (string.IsNullOrEmpty(accessToken))
{
    // Prompt the user to sign-in
    var interactiveRequest = PCA.AcquireTokenInteractive(scopes);

    if (AuthUIParent != null)
    {
        interactiveRequest = interactiveRequest
            .WithParentActivityOrWindow(AuthUIParent);
    }

    var authResult = await interactiveRequest.ExecuteAsync();
    Debug.WriteLine($"Access Token: {authResult.AccessToken}");
}
```

Этот код сначала пытается получить маркер доступа в автоматическом режиме. Если сведения о пользователе уже находятся в кэше приложения (например, если пользователь закрыл приложение раньше, без выхода), это будет успешным, и нет причин запрашивать пользователя. Если в кэше нет сведений о пользователе, `AcquireTokenSilent().ExecuteAsync()` функция создает исключение. `MsalUiRequiredException` В этом случае код вызывает интерактивную функцию для получения маркера `AcquireTokenInteractive`.

Теперь обновите `SignOut` функцию, чтобы удалить сведения о пользователе из кэша. Добавьте следующий код в начало `SignOut` функции.

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

### <a name="update-android-project-to-enable-sign-in"></a>Обновление проекта Android для включения входа

При использовании в проекте Xamarin Android Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).

Сначала необходимо обновить манифест Android, чтобы зарегистрировать URI перенаправления. В проекте **графтуториал. Android** разверните папку **свойства** , а затем откройте **файл AndroidManifest. XML**. Если вы используете Visual Studio для Mac, переключитесь на представление **источника** с помощью вкладок в нижней части файла. Замените все содержимое приведенным ниже содержимым.

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

Замените `YOUR_APP_ID_HERE` идентификатором приложения из регистрации приложения.

Затем откройте **MainActivity.CS** и добавьте приведенные ниже `using` операторы в начало файла.

```cs
using Microsoft.Identity.Client;
using Android.Content;
```

Затем переопределите `OnActivityResult` функцию, чтобы передать управление в библиотеку MSAL. Добавьте в `MainActivity` класс следующее:

```cs
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationContinuationHelper
        .SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

Наконец, в `OnCreate` функции добавьте следующую строку после `LoadApplication(new App());` строки.

```cs
App.AuthUIParent = this;
```

### <a name="update-ios-project-to-enable-sign-in"></a>Обновление проекта iOS для включения входа

> [!IMPORTANT]
> Так как MSAL требует использования файла. plist, необходимо настроить Visual Studio с помощью учетной записи разработчика Apple, чтобы включить подготовку. Если вы используете это руководство в имитаторе iPhone, необходимо добавить **plist** в поле " **Настраиваемые** права" в параметрах проекта **** **графтуториал. iOS** в параметрах проекта. Для получения дополнительных сведений см [deviceing Device Подготовка для Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).

При использовании в проекте Xamarin для iOS Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).

Для начала необходимо включить доступ к цепочке ключей. В обозревателе решений разверните проект **графтуториал. iOS** , а затем откройте файл **plist** . Укажите право на предоставление **цепочки ключей** и выберите **включить цепочку ключей**. В **группы цепочки ключей**добавьте запись в формате `com.YOUR_DOMAIN.GraphTutorial`.

![Снимок экрана: Настройка обслуживания цепочки ключей](./images/enable-keychain-access.png)

Далее необходимо зарегистрировать URI перенаправления MSAL по умолчанию в качестве типа URL-адреса, обрабатываемого вашим приложением. Откройте файл **info. plist** и внесите следующие изменения.

- На вкладке **приложение** убедитесь, что значение **идентификатора пакета** соответствует значению, заданному для **групп ключей** в целях **обслуживания. plist**. Если это не так, обновите его сейчас.
- На вкладке **Дополнительно** перейдите в раздел **типы URL-адресов** . Добавьте URL-адрес типа со следующими значениями:
  - **Identifier**: значение **идентификатора набора**
  - **Схемы URL-адресов**: `msal{YOUR-APP-ID}`задано значение. Например, если идентификатор приложения — `67ad5eba-0cfc-414d-8f9f-0a6d973a907c`, задайте значение. `msal67ad5eba-0cfc-414d-8f9f-0a6d973a907c`
  - **Роль**:`Editor`
  - **Значок**: оставьте пустым

![Снимок экрана с разделом "типы URL-адресов" в разделе info. plist](./images/add-url-type.png)

Наконец, обновите код в проекте **графтуториал. iOS** для обработки перенаправления во время проверки подлинности. Откройте файл **AppDelegate.CS** и добавьте приведенный ниже `using` оператор в начало файла.

```cs
using Microsoft.Identity.Client;
```

Добавьте указанную ниже строку `FinishedLaunching` , чтобы она функционировала непосредственно перед `LoadApplication(new App());` строкой.

```cs
// Specify the Keychain access group
App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
```

Наконец, переопределите `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL. Добавьте в `AppDelegate` класс следующее:

```cs
// Handling redirect URL
// See: https://github.com/azuread/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

## <a name="storing-the-tokens"></a>Сохранение маркеров

Когда библиотека проверки подлинности (Майкрософт) используется в проекте Xamarin, для кэширования маркеров по умолчанию используется встроенное безопасное хранение. Дополнительные сведения см. в статье [сериализация кэша маркеров](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .

## <a name="test-sign-in"></a>Проверка входа

На этом шаге при запуске приложения и нажатии кнопки **входа** вам будет предложено выполнить вход. При успешном входе в систему отображается маркер доступа, напечатанный в выводе отладчика.

![Снимок экрана с окном вывода в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a>Получение сведений о пользователе

Теперь обновите `SignIn` функцию в **app.XAML.CS** , чтобы инициализировать `GraphServiceClient`. Добавьте следующий код перед `await GetUserInfo();` строкой.

```cs
// Initialize Graph client
GraphClient = new GraphServiceClient(new DelegateAuthenticationProvider(
    async (requestMessage) =>
    {
        var accounts = await PCA.GetAccountsAsync();

        var result = await PCA.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
            .ExecuteAsync();

        requestMessage.Headers.Authorization =
            new AuthenticationHeaderValue("Bearer", result.AccessToken);
    }));
```

Теперь обновите `GetUserInfo` функцию, чтобы получить сведения о пользователе из Microsoft Graph. Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.

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

Если вы сохраните изменения и запустите приложение сейчас, после входа в пользовательский интерфейс отобразится отображаемое имя пользователя и адрес электронной почты.
