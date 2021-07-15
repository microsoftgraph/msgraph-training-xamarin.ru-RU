<!-- markdownlint-disable MD002 MD041 -->

В этом разделе вы добавим возможность создания событий в календаре пользователя.

1. Откройте **NewEventPage.xaml** и замените его содержимое следующим.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml":::

1. Откройте **NewEventPage.xaml.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.

    ```csharp
    using System.ComponentModel;
    using Microsoft.Graph;
    ```

1. Добавьте интерфейс **INotifyPropertyChange** в **класс NewEventPage.** Замените существующее объявление класса следующим.

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

1. Сохраните изменения и запустите приложение. Во входе выберите элемент **меню "Новое** событие", заполните форму и выберите **Создать,** чтобы добавить событие в календарь пользователя.

    ![Снимок экрана новой страницы события](images/new-event-page.png)
