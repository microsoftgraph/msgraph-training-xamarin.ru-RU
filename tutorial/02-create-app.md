<!-- markdownlint-disable MD002 MD041 -->

1. Откройте Visual Studio и нажмите **Создать новый проект**.

1. В **диалоговом окте "Создание нового** проекта" выберите мобильное **приложение (Xamarin.Forms)** и выберите **Далее**.

    ![Visual Studio 2019 г. создайте новый диалоговое окно проекта](images/new-project-dialog.png)

1. В **диалоговом** октаге Настройка нового проекта введите имя Project и имя решения, а затем `GraphTutorial` выберите **Создать**.  

    > [!IMPORTANT]
    > Убедитесь, что вы вводите точно такое же имя для Visual Studio Project, указанного в этих лабораторных инструкциях. Имя проекта Visual Studio становится частью пространства имен в коде. Для кода в этих инструкциях важно, чтобы пространство имен совпадало с именем проекта Visual Studio, указанным в этих инструкциях. Если ввести другое имя проекта, код не будет компилироваться, пока вы не настроите все пространства имен в соответствии с именем, которое вы указали для проекта Visual Studio при его создании.

    ![Visual Studio 2019 г. настройте новый диалоговое окно проекта](images/configure-new-project-dialog.png)

1. В **диалоговом** окланте  New Cross Platform App выберите пустой шаблон и выберите платформы, которые необходимо создать в **рамках Платформы.** Выберите **ОК** для создания решения.

    ![Visual Studio 2019 новый диалоговое окно приложения для кросс-платформы](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a>Установка пакетов

Прежде чем двигаться дальше, установите дополнительные NuGet пакеты, которые вы будете использовать позже.

- [Microsoft.Identity.Client для](https://www.nuget.org/packages/Microsoft.Identity.Client/) обработки проверки подлинности Azure AD и управления маркерами.
- [Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) для звонков в microsoft Graph.
- [TimeZoneConverter для](https://www.nuget.org/packages/TimeZoneConverter/) обработки часовых поясов поперек платформы.

1. Выберите **Сервис > Диспетчер пакетов NuGet > Консоль диспетчера пакетов**.

1. В консоли диспетчера пакетов введите следующие команды.

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.34.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.34.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.34.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 4.0.0 -Project GraphTutorial
    Install-Package TimeZoneConverter -Project GraphTutorial
    ```

## <a name="design-the-app"></a>Проектирование приложения

Начните с обновления класса, чтобы добавить переменные для отслеживания состояния проверки подлинности и `App` подписанного пользователя.

1. В **Обозревателе** решений расширь **проект GraphTutorial,** а затем расширь **файл App.xaml.** Откройте **файл App.xaml.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.

    ```csharp
    using System.ComponentModel;
    using System.IO;
    using System.Reflection;
    using System.Threading.Tasks;
    ```

1. Добавьте интерфейс `INotifyPropertyChanged` в объявление класса.

    ```csharp
    public partial class App : Application, INotifyPropertyChanged
    ```

1. Добавьте в класс следующие `App` свойства.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GlobalPropertiesSnippet":::

1. Добавьте в класс следующие `App` функции. , `SignIn` `SignOut` и `GetUserInfo` функции являются только placeholders на данный момент.

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

1. Функция `GetUserPhoto` возвращает фотографию по умолчанию на данный момент. Вы можете либо предоставить собственный файл, либо скачать тот, который используется в примере [из GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png) Если вы используете собственный файл, переименуйте его в **no-profile-pic.png**.

1. Скопируйте файл в **каталог ./GraphTutorial/GraphTutorial.**

1. Щелкните правой кнопкой мыши файл в **обозревателе решений** и выберите **Свойства**. В **окне Свойства** измените значение **Действия сборки** на **встроенный ресурс.**

    ![Снимок экрана окна Свойства для PNG-файла](./images/png-file-properties.png)

### <a name="app-navigation"></a>Навигация по приложениям

В этом разделе вы измените главную страницу приложения на [FlyoutPage.](/xamarin/xamarin-forms/app-fundamentals/navigation/flyoutpage) Это обеспечит меню навигации для переключения между представлением в приложении.

1. Откройте **файл MainPage.xaml** в **проекте GraphTutorial** и замените его содержимое следующим.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml":::

#### <a name="implement-the-menu"></a>Реализация меню

1. Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем **новую папку**. Назови папку `Models` .

1. Щелкните правой кнопкой мыши **папку Модели** и выберите **Добавить**, а затем **класс ...**. Назови класс `NavMenuItem` и выберите **Добавить**.

1. Откройте **файл NavMenuItem.cs** и замените его содержимое следующим.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `MenuPage` . Нажмите **Добавить**.

1. Откройте файл **MenuPage.xaml** и замените его содержимое следующим.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml":::

1. **Расширь menuPage.xaml** в **обозревателе решений** и откройте **файл MenuPage.xaml.cs.** Замените его содержимое следующим.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > Visual Studio будет сообщать об ошибках **в MenuPage.xaml.cs**. Эти ошибки будут устранены на более позднем этапе.

#### <a name="implement-the-welcome-page"></a>Реализация страницы приветствия

1. Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `WelcomePage` . Нажмите **Добавить**. Откройте файл **WelcomePage.xaml** и замените его содержимое следующим.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml":::

1. **Расширь welcomePage.xaml** в **обозревателе** решений и откройте **файл WelcomePage.xaml.cs.** Добавьте к классу `WelcomePage` следующую функцию:

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-and-new-event-pages"></a>Добавление страниц календаря и новых событий

Теперь добавьте страницу календаря и новую страницу событий. Они будут просто местообнажителей на данный момент.

1. Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `CalendarPage` . Нажмите **Добавить**.

1. Щелкните правой кнопкой **мыши проект GraphTutorial** и выберите **Добавить**, а затем Новый **элемент ...**. Выберите **страницу контента** и назови ее `NewEventPage` . Нажмите **Добавить**.

#### <a name="update-mainpage-code-behind"></a>Обновление кода MainPage

Теперь, когда все страницы на месте, обнови код-за для **MainPage.xaml**.

1. **Разместим MainPage.xaml** в обозревателе решений и откройте **файл MainPage.xaml.cs** и замените все содержимое следующим образом. 

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. Сохраните все изменения. Щелкните правой кнопкой мыши проект, который вы хотите запустить (Android, iOS или UWP) и выберите **Set как startUp Project**. Нажмите **кнопку F5** или **выберите отладку > начать отладку** в Visual Studio.

    ![Скриншоты версий приложения для Android, iOS и UWP](./images/welcome-page.png)
