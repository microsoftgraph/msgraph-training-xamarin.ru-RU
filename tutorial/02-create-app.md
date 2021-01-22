<!-- markdownlint-disable MD002 MD041 -->

1. Откройте Visual Studio и выберите **"Создать новый проект".**

1. В **диалоговом окке** "Создание нового проекта" выберите мобильное **приложение (Xamarin.Forms)** и выберите **"Далее".**

    ![Visual Studio 2019 г. диалоговое окно создания проекта](images/new-project-dialog.png)

1. В **диалоговом окке** "Настройка нового проекта" введите имя проекта и имя решения, а `GraphTutorial` затем выберите **"Создать".**  

    > [!IMPORTANT]
    > Убедитесь, что введите точно такое же имя для Visual Studio Project, которое указано в этих лабораторных инструкциях. Имя Visual Studio проекта становится частью пространства имен в коде. Код в этих инструкциях зависит от пространства имен, совпадающих Visual Studio имени проекта, указанного в этих инструкциях. Если используется другое имя проекта, код не будет компилироваться, если не настроить все пространства имен в соответствие с именем Visual Studio проекта, которое вы вводите при создании проекта.

    ![Visual Studio 2019 г. диалоговое окно "Настройка нового проекта"](images/configure-new-project-dialog.png)

1. В **диалоговом** окте "Создание кроссплатформенного приложения" выберите пустой шаблон и выберите платформы, которые вы хотите создать в **рамках платформ.**  Выберите **"ОК",** чтобы создать решение.

    ![Visual Studio 2019: новое кроссплатформечное приложение](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a>Установка пакетов

Прежде чем двигаться дальше, установите некоторые дополнительные пакеты NuGet, которые вы будете использовать позже.

- [Microsoft.Identity.Client для](https://www.nuget.org/packages/Microsoft.Identity.Client/) обработки проверки подлинности Azure AD и управления маркерами.
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) для вызовов Microsoft Graph.
- [TimeZoneConverter](https://www.nuget.org/packages/TimeZoneConverter/) для обработки часовых поясов на разных платформах.

1. Select **Tools > NuGet диспетчер пакетов > диспетчер пакетов Console**.

1. В консоли диспетчер пакетов введите следующие команды.

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 3.21.0 -Project GraphTutorial
    Install-Package TimeZoneConverter -Version 3.3.0 -Project GraphTutorial
    ```

## <a name="design-the-app"></a>Проектирование приложения

Для начала обновив класс, добавьте переменные для отслеживания состояния проверки подлинности и пользователя, выписав `App` его.

1. В **обозревателе решений** разорвем **проект GraphTutorial,** а затем разорвем **файл App.xaml.** Откройте файл **App.xaml.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.

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

1. Добавьте в класс следующие `App` функции. На данный момент функции и функции `SignIn` `SignOut` являются просто `GetUserInfo` замесятелями.

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

1. На `GetUserPhoto` данный момент функция возвращает фотографию по умолчанию. Вы можете предоставить собственный файл или скачать файл, используемый в примере, с [GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png) При использовании собственного файла переименуйте его в **no-profile-pic.png.**

1. Скопируйте файл в **каталог ./GraphTutorial/GraphTutorial.**

1. Щелкните правой кнопкой мыши файл в **обозревателе решений** и выберите **"Свойства".** В **окне "Свойства"** измените значение действия сборки **на** **внедренный ресурс.**

    ![Снимок экрана: окно "Свойства" для файла PNG](./images/png-file-properties.png)

### <a name="app-navigation"></a>Навигация по приложениям

В этом разделе вы измените главную страницу приложения на главную [страницу с подробными данными.](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page) При этом будет доступно меню навигации для переключения между представлениями в приложении.

1. Откройте файл **MainPage.xaml** в проекте **GraphTutorial** и замените его содержимое на следующее.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml" id="MainPageXamlSnippet":::

#### <a name="implement-the-menu"></a>Реализация меню

1. Щелкните правой кнопкой мыши проект **GraphTutorial** и выберите **"Добавить"** и "Новая **папка".** Назовем `Models` папку.

1. Щелкните правой **кнопкой мыши** папку Models и выберите **"Добавить",** затем **"Класс"...** Назовем класс `NavMenuItem` и выберите **"Добавить".**

1. Откройте файл **NavMenuItem.cs** и замените его содержимое на следующий файл.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. Щелкните правой кнопкой мыши проект **GraphTutorial** и выберите **"Добавить"** и **"Новый элемент"...** Choose **Content Page** and name the page `MenuPage` . Нажмите кнопку **Добавить**.

1. Откройте файл **MenuPage.xaml** и замените его содержимое на следующее.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml" id="MenuPageXamlSnippet":::

1. **Разкрой файл MenuPage.xaml** **в** обозревателе решений и откройте **MenuPage.xaml.cs** файла. Замените его содержимое на следующее.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > Visual Studio будет сообщать об ошибках **в** MenuPage.xaml.cs. Эти ошибки будут устранены на более позднем этапе.

#### <a name="implement-the-welcome-page"></a>Реализация страницы приветствия

1. Щелкните правой кнопкой мыши проект **GraphTutorial** и выберите **"Добавить"** и **"Новый элемент"...** Choose **Content Page** and name the page `WelcomePage` . Нажмите кнопку **Добавить**. Откройте файл **WelcomePage.xaml** и замените его содержимое на следующее.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml" id="WelcomePageXamlSnippet":::

1. **Разкрой файл WelcomePage.xaml** в **обозревателе** решений и откройте **WelcomePage.xaml.cs** файла. Добавьте к классу `WelcomePage` следующую функцию:

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-and-new-event-pages"></a>Добавление календаря и новых страниц событий

Теперь добавьте страницу календаря и новую страницу событий. На данный момент они будут просто замесятелями.

1. Щелкните правой кнопкой мыши проект **GraphTutorial** и выберите **"Добавить"** и **"Новый элемент"...** Choose **Content Page** and name the page `CalendarPage` . Нажмите кнопку **Добавить**.

1. Щелкните правой кнопкой мыши проект **GraphTutorial** и выберите **"Добавить"** и **"Новый элемент"...** Choose **Content Page** and name the page `NewEventPage` . Нажмите кнопку **Добавить**.

#### <a name="update-mainpage-code-behind"></a>Обновление кода программной области MainPage

Теперь, когда все страницы на месте, обновите код программной части **для MainPage.xaml.**

1. Разместив **файл MainPage.xaml** в обозревателе решений, откройте файл MainPage.xaml.cs и замените его все содержимое на следующее.  

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. Сохраните все изменения. Щелкните правой кнопкой мыши проект, который вы хотите запустить (Android, iOS или UWP), и выберите "Установить в качестве **startUp Project".** Нажмите **F5** или выберите **"Отладка> начать отладку** в Visual Studio.

    ![Снимки экрана версий приложения для Android, iOS и UWP](./images/welcome-page.png)
