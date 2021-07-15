<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a48ce-101">В этом разделе вы добавим возможность создания событий в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="a48ce-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="a48ce-102">Откройте **NewEventPage.xaml** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="a48ce-102">Open **NewEventPage.xaml** and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml":::

1. <span data-ttu-id="a48ce-103">Откройте **NewEventPage.xaml.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.</span><span class="sxs-lookup"><span data-stu-id="a48ce-103">Open **NewEventPage.xaml.cs** and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using System.ComponentModel;
    using Microsoft.Graph;
    ```

1. <span data-ttu-id="a48ce-104">Добавьте интерфейс **INotifyPropertyChange** в **класс NewEventPage.**</span><span class="sxs-lookup"><span data-stu-id="a48ce-104">Add the **INotifyPropertyChange** interface to the **NewEventPage** class.</span></span> <span data-ttu-id="a48ce-105">Замените существующее объявление класса следующим.</span><span class="sxs-lookup"><span data-stu-id="a48ce-105">Replace the existing class declaration with the following.</span></span>

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

1. <span data-ttu-id="a48ce-106">Добавьте следующие свойства в **класс NewEventPage.**</span><span class="sxs-lookup"><span data-stu-id="a48ce-106">Add the following properties to the **NewEventPage** class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="PropertiesSnippet":::

1. <span data-ttu-id="a48ce-107">Добавьте следующий код для создания события.</span><span class="sxs-lookup"><span data-stu-id="a48ce-107">Add the following code to create the event.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="CreateEventSnippet":::

1. <span data-ttu-id="a48ce-108">Сохраните изменения и запустите приложение.</span><span class="sxs-lookup"><span data-stu-id="a48ce-108">Save your changes and run the app.</span></span> <span data-ttu-id="a48ce-109">Во входе выберите элемент **меню "Новое** событие", заполните форму и выберите **Создать,** чтобы добавить событие в календарь пользователя.</span><span class="sxs-lookup"><span data-stu-id="a48ce-109">Sign in, select the **New event** menu item, fill in the form, and select **Create** to add an event to the user's calendar.</span></span>

    ![Снимок экрана новой страницы события](images/new-event-page.png)
