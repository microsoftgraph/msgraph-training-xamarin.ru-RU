<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph. На этом этапе вы интегрируете библиотеку проверки подлинности [Майкрософт для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.

1. В **обозревателе решений** разйдите проект **GraphTutorial** и щелкните правой кнопкой мыши **папку Models.** Выберите **"Добавить > класс..."** Назовем класс `OAuthSettings` и выберите **"Добавить".**

1. Откройте файл **OAuthSettings.cs** и замените его содержимое на следующий файл.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. Замените `YOUR_APP_ID_HERE` его на ИД приложения из регистрации приложения.

    > [!IMPORTANT]
    > Если вы используете управление исходным кодом, например git, пришло бы время исключить файл из системы управления исходным кодом, чтобы не допустить случайной утечки вашего `OAuthSettings.cs` ИД приложения.

## <a name="implement-sign-in"></a>Реализация входов

1. Откройте файл **App.xaml.cs** в **проекте GraphTutorial** и добавьте следующие утверждения в `using` верхнюю часть файла.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    using TimeZoneConverter;
    ```

1. Измените **строку объявления** класса App, чтобы устранить конфликт имен **для Приложения.**

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. Добавьте в класс следующие `App` свойства.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. Затем создайте новый `PublicClientApplication` в конструкторе `App` класса.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. `SignIn`Обновим функцию, чтобы использовать `PublicClientApplication` ее для получения маркера доступа. Добавьте следующий код над `await GetUserInfo();` строкой.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    Этот код сначала пытается получить маркер доступа в тихом режиме. Если сведения о пользователе уже есть в кэше приложения (например, если пользователь ранее закрыл приложение без выходных данных), это будет успешно, и нет причин для запроса пользователя. Если в кэше нет сведений о пользователе, функция `AcquireTokenSilent().ExecuteAsync()` вырлает `MsalUiRequiredException` исключение. В этом случае код вызывает интерактивную функцию для получения `AcquireTokenInteractive` маркера.

1. `SignOut`Обновите функцию, чтобы удалить данные пользователя из кэша. Добавьте следующий код в начало `SignOut` функции.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a>Обновление проекта Android, чтобы включить вход

При его использования в проекте Xamarin для Android библиотека проверки подлинности Майкрософт предъявляет несколько [требований, характерных для Android.](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics)

1. В **проекте GraphTutorial.Android** разкроте папку **"Свойства",** а затем откройте **AndroidManifest.xml.** Если вы используете Visual Studio Mac, выберите "AndroidManifest.xml" и "Открыть с помощью" и "Редактор **исходных кодов".**  Замените все содержимое на следующее.

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. Откройте **MainActivity.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. Переопределять `OnActivityResult` функцию, чтобы передать управление библиотеке MSAL. Добавьте в класс `MainActivity` следующее:

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. Добавьте `OnCreate` в функцию следующую строку после `LoadApplication(new App());` строки.

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a>Обновление проекта iOS для реализации входов

> [!IMPORTANT]
> Так как MSAL требует использования файла Entitlements.plist, необходимо настроить Visual Studio с помощью учетной записи разработчика Apple, чтобы включить ее. Если вы работаете с этим учебником в имитаторе iPhone, вам потребуется добавить **Entitlements.plist** в поле "Пользовательские права" в параметрах проекта **GraphTutorial.iOS,** подписывая пакеты **>iOS.**  Дополнительные сведения [см. в подгонке "Подготовка устройств для Xamarin.iOS".](/xamarin/ios/get-started/installation/device-provisioning)

При этом в проекте Xamarin для iOS библиотека проверки подлинности Майкрософт предъявляет несколько [требований, характерных для iOS.](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics)

1. В обозревателе решений раз откройте проект **GraphTutorial.iOS** и откройте файл **Entitlements.plist.**

1. Найдите **право на получение ключей** и выберите **"Включить keyChain".**

1. В **группах ключей** добавьте запись в формате. `com.companyname.GraphTutorial`

    ![Снимок экрана: конфигурация прав на ключ](./images/enable-keychain-access.png)

1. Обновите код в **проекте GraphTutorial.iOS,** чтобы обработать перенаправление во время проверки подлинности. Откройте **AppDelegate.cs** файла и добавьте в верхнюю часть файла следующий `using` выписку.

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. Добавьте следующую строку для `FinishedLaunching` работы прямо перед `LoadApplication(new App());` строкой.

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. Переопределять `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL. Добавьте в класс `AppDelegate` следующее:

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a>Хранение маркеров

Когда библиотека проверки подлинности Майкрософт используется в проекте Xamarin, она по умолчанию кэширование маркеров обеспечивается с помощью нативного безопасного хранилища. Дополнительные [сведения см. в](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) сведениях о сериализации кэша маркеров.

## <a name="test-sign-in"></a>Тестовый вход

На этом этапе, если запустить  приложение и нажать кнопку "Войти", вам будет предложено выполнить вход. При успешном входе маркер доступа должен отпечатываться в выходных данных отладка.

![Снимок экрана с окном вывода в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a>Получить сведения о пользователе

1. Добавьте новую функцию в **класс App** для инициализации `GraphServiceClient` .

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. `SignIn`Обновим функцию **в App.xaml.cs,** чтобы она вызывала эту функцию вместо `GetUserInfo` . Удалите из функции `SignIn` следующее:

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. Добавьте следующее в конец `SignIn` функции.

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. `GetUserInfo`Обновим функцию, чтобы получить сведения о пользователе из Microsoft Graph. Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. Сохраните изменения и запустите приложение. После того как пользовательский интерфейс будет обновлен с помощью отображаемого имени пользователя и адреса электронной почты.
