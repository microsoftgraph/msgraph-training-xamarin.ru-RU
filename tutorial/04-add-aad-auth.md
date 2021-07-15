<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы расширит приложение от предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова microsoft Graph. На этом этапе вы интегрируете библиотеку проверки подлинности [Майкрософт для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.

1. В **Обозревателе** решений разойдите **проект GraphTutorial** и щелкните правой кнопкой мыши **папку Models.** Выберите **Добавить > класс...**. Назови класс `OAuthSettings` и выберите **Добавить**.

1. Откройте **файл OAuthSettings.cs** и замените его содержимое следующим.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. Замените `YOUR_APP_ID_HERE` с помощью ID приложения из регистрации приложения.

    > [!IMPORTANT]
    > Если вы используете источник управления, например git, сейчас самое время исключить файл из источника управления, чтобы избежать случайной утечки вашего `OAuthSettings.cs` ID приложения.

## <a name="implement-sign-in"></a>Реализация входа в систему

1. Откройте **файл App.xaml.cs** в **проекте GraphTutorial** и добавьте в верхней части файла следующие `using` утверждения.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    using TimeZoneConverter;
    ```

1. Измените **строку объявления** класса App, чтобы разрешить конфликт имен для **приложения.**

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. Добавьте в класс следующие `App` свойства.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. Далее создайте новое `PublicClientApplication` в конструкторе `App` класса.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. `SignIn`Обнови функцию, чтобы использовать `PublicClientApplication` маркер доступа. Добавьте следующий код над `await GetUserInfo();` строкой.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    Этот код сначала пытается получить маркер доступа молча. Если сведения пользователя уже есть в кэше приложения (например, если пользователь закрыл приложение ранее без подписки), это будет успешно, и нет причин подсказывать пользователю. Если в кэше нет сведений пользователя, функция `AcquireTokenSilent().ExecuteAsync()` бросает `MsalUiRequiredException` . В этом случае код вызывает интерактивную функцию для получения `AcquireTokenInteractive` маркера.

1. `SignOut`Обновите функцию, чтобы удалить сведения пользователя из кэша. Добавьте следующий код в начало `SignOut` функции.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a>Обновление проекта Android, чтобы включить вход

При его реализации в android-проекте Xamarin библиотека проверки подлинности Майкрософт имеет несколько [требований, определенных для Android.](/azure/active-directory/develop/msal-net-xamarin-android-considerations)

1. В **проекте GraphTutorial.Android** разложите папку **Свойства** и откройте **AndroidManifest.xml.** Если вы используете Visual Studio для Mac, нажмите кнопку **AndroidManifest.xml** и выберите **Open With**, а затем **редактор исходных кодов**. Замените все содержимое следующим.

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. Откройте **MainActivity.cs** и добавьте следующие утверждения `using` в верхнюю часть файла.

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. Переопределять функцию, чтобы передать управление библиотеке `OnActivityResult` MSAL. Добавьте следующее в `MainActivity` класс.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. В `OnCreate` функции добавьте следующую строку после `LoadApplication(new App());` строки.

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a>Обновление проекта iOS, чтобы включить вход

> [!IMPORTANT]
> Так как MSAL требует использования файла Entitlements.plist, необходимо настроить Visual Studio учетной записью разработчика Apple, чтобы включить подготовка. Если вы работаете с этим учебником в симуляторе iPhone, вам необходимо добавить **Entitlements.plist** в поле Custom **Entitlements** в параметрах **проекта GraphTutorial.iOS,** **build->iOS Bundle Signing**. Дополнительные сведения см. в [пункте Подготовка устройств для Xamarin.iOS.](/xamarin/ios/get-started/installation/device-provisioning)

При его реализации в проекте Xamarin iOS библиотека проверки подлинности Майкрософт имеет несколько требований, [определенных для iOS.](/azure/active-directory/develop/msal-net-xamarin-ios-considerations)

1. В Обозревателе решений раз откройте **проект GraphTutorial.iOS,** а затем откройте файл **Entitlements.plist.**

1. Найдите **право Keychain** и выберите **Включить KeyChain.**

1. В **Keychain Groups** добавьте запись с форматом `com.companyname.GraphTutorial` .

    ![Снимок экрана конфигурации прав keychain](./images/enable-keychain-access.png)

1. Обновление кода в **проекте GraphTutorial.iOS** для обработки перенаправления во время проверки подлинности. Откройте **файл AppDelegate.cs** и добавьте следующее утверждение `using` в верхней части файла.

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. Добавьте следующую строку `FinishedLaunching` для работы перед `LoadApplication(new App());` строкой.

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. Переопределять `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL. Добавьте следующее в `AppDelegate` класс.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a>Хранение маркеров

Когда библиотека проверки подлинности Майкрософт используется в проекте Xamarin, она по умолчанию используется для кэширования маркеров на родном безопасном хранилище. Дополнительные сведения см. в дополнительных [сведениях.](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization)

## <a name="test-sign-in"></a>Тестовый вход

В этот момент при запуске приложения и нажатии кнопки **Вход** в кнопку, вам будет предложено войти. При успешном входе вы должны увидеть маркер доступа, напечатанный на выходе отладки.

![Снимок экрана окна Выход в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a>Получение сведений о пользователе

1. Добавьте новую функцию в **класс App,** чтобы инициализировать `GraphServiceClient` .

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. `SignIn`Обнови функцию **в App.xaml.cs,** чтобы вызвать эту функцию вместо `GetUserInfo` . Удалите следующее из `SignIn` функции.

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. Добавьте следующее в конец `SignIn` функции.

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. Обнови функцию, чтобы получить сведения пользователя `GetUserInfo` из microsoft Graph. Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. Сохраните изменения и запустите приложение. После регистрации пользовательский интерфейс обновляется с именем и адресом электронной почты пользователя.
