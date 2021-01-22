<!-- markdownlint-disable MD002 MD041 -->

В этом разделе вы добавим возможность создания событий в календаре пользователя.

1. Добавьте новую страницу для нового представления событий. Щелкните правой кнопкой мыши проект **GraphTutorial** в обозревателе решений и **выберите "Добавить > Новый элемент...".** Выберите **пустую страницу,** `NewEventPage.xaml` введите в поле **"Имя"** и выберите **"Добавить".**

1. Откройте **NewEventPage.xaml** и замените его содержимое на следующее.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml" id="NewEventPageXamlSnippet":::

1. Откройте **NewEventPage.xaml.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="UsingStatementsSnippet":::

1. Добавьте интерфейс **INotifyPropertyChange** в **класс NewEventPage.** Замените существующее объявление класса на следующее.

    ```csharp
    [XamlCompilation(XamlCompilationOptions.Compile)]
    public partial class NewEventPage : ContentPage, INotifyPropertyChanged
    {
        public NewEventPage()
        {
            InitializeComponent();
            BindingContext = this;
        }
    }
    ```

1. Добавьте следующие свойства в **класс NewEventPage.**

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="PropertiesSnippet":::

1. Добавьте следующий код для создания события.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="CreateEventSnippet":::

1. Сохраните изменения и запустите приложение. Во sign in, select the **New event** menu item, fill in the form, and select **Create** to add an event to the user's calendar.

    ![Снимок экрана: новая страница события](images/new-event-page.png)
