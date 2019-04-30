# <a name="completed-module-add-azure-ad-authentication"></a>Завершенный модуль: Добавление проверки подлинности Azure AD

Версия проекта в этом каталоге соответствует завершению работы с руководством [Добавление проверки подлиннОсти Azure AD](https://docs.microsoft.com/graph/tutorials/xamarin?tutorial-step=3). Если вы используете эту версию проекта, вам необходимо выполнить остальные руководства, начиная с [получения данных календаря](https://docs.microsoft.com/graph/tutorials/xamarin?tutorial-step=4).

> **Примечание:** Предполагается, что вы уже зарегистрировали приложение на портале Azure, как указано в разделе [Регистрация приложения на портале](https://docs.microsoft.com/graph/tutorials/xamarin?tutorial-step=2). Эту версию примера необходимо настроить следующим образом:
>
> 1. Переименуйте `./GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example` файл в `OAuthSettings.cs`.
> 1. Измените `OAuthSettings.cs` файл и внесите следующие изменения.
>     1. Замените `YOUR_APP_ID_HERE` идентификатором **приложения** , полученным с портала Azure.
>     1. Замените `YOUR_REDIRECT_URI_HERE` на URI перенаправления, начинающийся `msal` с портала Azure.
> 1. В проекте **графтуториал. iOS** откройте `Info.plist` файл.
>     1. На вкладке **Дополнительно** перейдите в раздел **типы URL-адресов** . Измените запись **схемы URL-адресов** из `YOUR_REDIRECT_URI_HERE` записи в URI перенаправления из регистрации приложения, начинающейся с `msal`.