<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph. На этом этапе выполняется интеграция [библиотеки проверки подлинности Microsoft для .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) в приложение.

1. В **обозревателе решений**разверните проект **графтуториал** и щелкните правой кнопкой мыши папку **Models** . Выберите **Добавить класс >..**.. Назовите класс `OAuthSettings` и нажмите кнопку **Добавить**.

1. Откройте файл **OAuthSettings.CS** и замените его содержимое на приведенный ниже код.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. Замените `YOUR_APP_ID_HERE` идентификатором приложения из регистрации приложения.

    > [!IMPORTANT]
    > Если вы используете систему управления версиями (например, Git), то теперь мы бы не могли исключить `OAuthSettings.cs` файл из системы управления версиями, чтобы избежать случайной утечки идентификатора приложения.

## <a name="implement-sign-in"></a>Реализация входа

1. Откройте файл **app.XAML.CS** в проекте **графтуториал** и добавьте приведенные ниже `using` операторы в начало файла.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    ```

1. Измените строку объявления класса **приложения** , чтобы разрешить конфликт имен для **приложения**.

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. Добавьте в `App` класс указанные ниже свойства.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. Затем создайте новый `PublicClientApplication` элемент в конструкторе `App` класса.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. Обновите `SignIn` функцию, чтобы она `PublicClientApplication` использовалась для получения маркера доступа. Добавьте приведенный ниже код перед `await GetUserInfo();` строкой.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    Этот код сначала пытается получить маркер доступа в автоматическом режиме. Если сведения о пользователе уже находятся в кэше приложения (например, если пользователь закрыл приложение раньше, без выхода), это будет успешным, и нет причин запрашивать пользователя. Если в кэше нет сведений о пользователе, `AcquireTokenSilent().ExecuteAsync()` функция создает исключение. `MsalUiRequiredException` В этом случае код вызывает интерактивную функцию для получения маркера `AcquireTokenInteractive`.

1. Обновите `SignOut` функцию, чтобы удалить сведения о пользователе из кэша. Добавьте следующий код в начало `SignOut` функции.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a>Обновление проекта Android для включения входа

При использовании в проекте Xamarin Android Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).

1. В проекте **графтуториал. Android** разверните папку **свойства** , а затем откройте **файл AndroidManifest. XML**. Если вы используете Visual Studio для Mac, нажмите кнопку **AndroidManifest. XML** и выберите **Открыть с помощью** **редактора исходного кода**. Замените все содержимое приведенным ниже содержимым.

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. Откройте **MainActivity.CS** и добавьте приведенные `using` ниже операторы в начало файла.

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. Переопределите `OnActivityResult` функцию, чтобы передать управление в библиотеку MSAL. Добавьте в `MainActivity` класс следующее:

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. В `OnCreate` функции добавьте следующую строку после `LoadApplication(new App());` строки.

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a>Обновление проекта iOS для включения входа

> [!IMPORTANT]
> Так как MSAL требует использования файла. plist, необходимо настроить Visual Studio с помощью учетной записи разработчика Apple, чтобы включить подготовку. Если вы используете это руководство в имитаторе iPhone, вам нужно добавить **plists.** в поле **Настраиваемые** права в параметрах проекта **графтуториал. iOS** , **Сборка >Подписывание пакетов iOS**. Для получения дополнительных сведений см [deviceing Device Подготовка для Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).

При использовании в проекте Xamarin для iOS Библиотека проверки подлинности (Майкрософт) имеет несколько [требований, характерных для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).

1. В обозревателе решений разверните проект **графтуториал. iOS** , а затем откройте файл **plist** .

1. Укажите право на предоставление **цепочки ключей** и выберите **включить цепочку ключей**.

1. В **группы цепочки ключей**добавьте запись в формате `com.companyname.GraphTutorial`.

    ![Снимок экрана: Настройка обслуживания цепочки ключей](./images/enable-keychain-access.png)

1. Обновление кода в проекте **графтуториал. iOS** для обработки перенаправления во время проверки подлинности. Откройте файл **AppDelegate.CS** и добавьте приведенный ниже `using` оператор в начало файла.

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. Добавьте указанную ниже строку `FinishedLaunching` , чтобы она функционировала непосредственно перед `LoadApplication(new App());` строкой.

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. Переопределите `OpenUrl` функцию, чтобы передать URL-адрес в библиотеку MSAL. Добавьте в `AppDelegate` класс следующее:

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a>Сохранение маркеров

Когда библиотека проверки подлинности (Майкрософт) используется в проекте Xamarin, для кэширования маркеров по умолчанию используется встроенное безопасное хранение. Дополнительные сведения см. в статье [сериализация кэша маркеров](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .

## <a name="test-sign-in"></a>Проверка входа

На этом шаге при запуске приложения и нажатии кнопки **входа** вам будет предложено выполнить вход. При успешном входе в систему отображается маркер доступа, напечатанный в выводе отладчика.

![Снимок экрана с окном вывода в Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a>Получение сведений о пользователе

1. Добавьте новую функцию в класс **app** , чтобы инициализировать `GraphServiceClient`.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. Обновите `SignIn` функцию в **app.XAML.CS** , чтобы вызвать эту функцию, `GetUserInfo`а не. Удалите из `SignIn` функции следующее:

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. Добавьте следующий элемент в конец `SignIn` функции.

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. Обновите `GetUserInfo` функцию, чтобы получить сведения о пользователе из Microsoft Graph. Замените имеющуюся функцию `GetUserInfo` указанным ниже кодом.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. Сохраните изменения и запустите приложение. После входа в пользовательский интерфейс будет выполнено обновление отображаемого имени пользователя и адреса электронной почты.
