<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="fd550-101">В этом упражнении вы создадите новое собственное приложение Azure AD с помощью центра администрирования Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="fd550-101">In this exercise you will create a new Azure AD native application using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="fd550-102">Откройте браузер и перейдите в [Центр администрирования Azure Active Directory](https://aad.portal.azure.com) и войдите с помощью **личной учетной записи** (т.е. учетной записи Майкрософт) или **рабочей или учебной учетной записи**.</span><span class="sxs-lookup"><span data-stu-id="fd550-102">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="fd550-103">Выберите **Azure Active Directory** в левой панели навигации, а затем выберите **Регистрация приложений** в разделе **Управление**.</span><span class="sxs-lookup"><span data-stu-id="fd550-103">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="fd550-104">Снимок экрана с регистрациями приложений</span><span class="sxs-lookup"><span data-stu-id="fd550-104">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="fd550-105">Выберите **Новая регистрация**.</span><span class="sxs-lookup"><span data-stu-id="fd550-105">Select **New registration**.</span></span> <span data-ttu-id="fd550-106">На странице**Зарегистрировать приложение** задайте необходимые значения следующим образом.</span><span class="sxs-lookup"><span data-stu-id="fd550-106">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="fd550-107">Введите **имя** `Xamarin Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="fd550-107">Set **Name** to `Xamarin Graph Tutorial`.</span></span>
    - <span data-ttu-id="fd550-108">Введите **поддерживаемые типы учетных записей** для **учетных записей в любом каталоге организаций и личных учетных записей Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="fd550-108">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="fd550-109">В разделе **URI перенаправления (необязательно)** измените раскрывающийся список на общедоступный **клиент (Mobile _амп_ Desktop)** и задайте для `urn:ietf:wg:oauth:2.0:oob`него значение.</span><span class="sxs-lookup"><span data-stu-id="fd550-109">Under **Redirect URI (optional)**, change the dropdown to **Public client (mobile & desktop)**, and set the value to `urn:ietf:wg:oauth:2.0:oob`.</span></span>

    ![Снимок страницы "регистрация приложения"](./images/aad-register-an-app.png)

1. <span data-ttu-id="fd550-111">Выберите **Зарегистрировать**.</span><span class="sxs-lookup"><span data-stu-id="fd550-111">Choose **Register**.</span></span> <span data-ttu-id="fd550-112">На странице **учебника Xamarin Graph** СКОПИРУЙТЕ значение **идентификатора Application (Client)** и сохраните его, он понадобится на следующем шаге.</span><span class="sxs-lookup"><span data-stu-id="fd550-112">On the **Xamarin Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Снимок экрана с ИДЕНТИФИКАТОРом приложения для новой регистрации приложения](./images/aad-application-id.png)

1. <span data-ttu-id="fd550-114">Выберите **проверку** подлинности в разделе **Управление**.</span><span class="sxs-lookup"><span data-stu-id="fd550-114">Select the **Authentication** under **Manage**.</span></span> <span data-ttu-id="fd550-115">Выберите раздел **предлагаемые URI перенаправления для общедоступных клиентов (мобильный, Настольный)** .</span><span class="sxs-lookup"><span data-stu-id="fd550-115">Locate the **Suggested Redirect URIs for public clients (mobile, desktop)** section.</span></span> <span data-ttu-id="fd550-116">Выберите URI, который начинается с `msal`, а затем нажмите кнопку **сохранить**.</span><span class="sxs-lookup"><span data-stu-id="fd550-116">Select the URI that begins with `msal`, then choose **Save**.</span></span>

    ![Снимок экрана со страницей URI перенаправления](./images/aad-redirect-uris.png)
